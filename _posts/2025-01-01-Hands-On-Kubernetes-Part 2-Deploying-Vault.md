---
classes: wide
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
  teaser: /assets/images/hashicorp-vault-logo.jpg
  og_image: /assets/images/hashicorp-vault-logo.jpg
---


<br />

In Part 1, we deployed a simple application on Kubernetes. Now, let’s introduce Vault to manage secrets dynamically. Our goal is to populate an environment variable in the app with a secret stored in Vault. We will manually unseal Vault and use the Vault Agent Injector to handle secret injection.

#### Assumptions
- Part 1 is completed [link](https://matcham89.github.io/chrismatcham-dev/Hands-On-Kubernetes-Part-1-Deploying-Kind/).
- Helm is installed.
- `jq` is installed.

#### Verifying the Cluster
Ensure the cluster is operational by running:
```bash
curl app.local
```
If the app responds correctly, proceed.

### Step 1: Add Vault Helm Chart
Add the HashiCorp Helm repository:
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
```

To explore and modify Vault’s configuration, save the default values:
```bash
mkdir helm && cd helm
helm show values hashicorp/vault > default-values.yaml
```

Install Vault using Helm:
```bash
helm install vault hashicorp/vault -f default-values.yaml -n vault --create-namespace
```
> Note: Use `--create-namespace` if the namespace does not exist.

#### Verify Pods
Check the Vault pods:
```bash
kubectl get pods -n vault
```
Expected output:
```
vault-0                             0/1   Running   0       70s
vault-agent-injector-6c5fcb994-nxhdr   1/1   Running   0       71s
```

#### Check Vault Logs
View the logs of the Vault server pod:
```bash
kubectl logs pods/vault-0 -n vault
```
Look for initialization messages.

Check Vault status:
```bash
kubectl exec -i -t vault-0 -n vault -- vault status
```

Key points:
- **Seal Type:** `shamir` (default manual seal).
- **Initialized:** `false` (fresh server).
- **Sealed:** `true` (no access permitted yet).

### Step 2: Initialize and Unseal Vault
Initialize Vault:
```bash
kubectl exec -i -t vault-0 -n vault -- vault operator init -key-shares=1 -key-threshold=1 -format=json > vault-keys.json
```
Extract the unseal key:
```bash
jq -r ".unseal_keys_b64[]" vault-keys.json
```
Set the unseal key:
```bash
VAULT_UNSEAL_KEY=$(jq -r ".unseal_keys_b64[]" vault-keys.json)
```
Unseal Vault:
```bash
kubectl exec vault-0 -n vault -- vault operator unseal $VAULT_UNSEAL_KEY
```
Confirm Vault is initialized and unsealed:
```bash
kubectl exec -i -t vault-0 -n vault -- vault status
```

### Step 3: Vault Secret Configuration
Log into Vault with the root token:
```bash
jq -r ".root_token" vault-keys.json
```
```bash
kubectl exec --stdin=true --tty=true vault-0 -n vault -- /bin/sh
```
```bash
vault login
```

Enable the `kv-v2` secrets engine:
```bash
vault secrets enable kv-v2
```
Enable Kubernetes authentication:
```bash
vault auth enable kubernetes
```
Configure Kubernetes host:
```bash
vault write auth/kubernetes/config   kubernetes_host=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT
```

Store a secret in Vault:
```bash
vault kv put kv-v2/vault-local/local-k8s secret="this is a secret stored in vault and exported with vault injector"
```
Verify the secret:
```bash
vault kv get kv-v2/vault-local/local-k8s
```

#### Create a Policy
Create a policy file:
```bash
echo 'path "kv-v2/data/vault-local/local-k8s" {
  capabilities = ["read"]
}' > /tmp/vault-local-policy.hcl
```
Apply the policy:
```bash
vault policy write vault-local /tmp/vault-local-policy.hcl
```
Create a role to bind the policy to a service account:
```bash
vault write auth/kubernetes/role/vault-local   bound_service_account_names=vault-local   bound_service_account_namespaces=default   policies=vault-local   ttl=1h
```

### Step 4: Configure Kubernetes Service Account For Vault
Create a service account, role, and role binding:

```bash
mkdir vault-k8s-sa
```

```bash
cd vault-k8s-sa
```
```bash
vim service-account.yaml
```
```yaml
# service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: vault-local
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: vault-local-role
rules:
- apiGroups: [""]
  resources: ["serviceaccounts/token"]
  verbs: ["create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: vault-default-rolebinding
subjects:
- kind: ServiceAccount
  name: vault-local
roleRef:
  kind: Role
  name: vault-local-role
  apiGroup: rbac.authorization.k8s.io
```
Apply the configuration:
```bash
kubectl apply -f service-account.yaml
```

Now we will copy our exsisting application to a new folder and update the annotations to support Vault

```bash
 cp -r app app-vault
 cd app-vault
```

```bash
rm deployment.yaml
vim deployment.yaml
```


Update the application deployment:
```liquid
{% raw %}
# app/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-vault
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-vault
  template:
    metadata:
      labels:
        app: app-vault
      annotations:
        # Vault Agent Injection annotations to enable Vault integration in the pod
        vault.hashicorp.com/agent-inject: "true"  # Tells Vault to inject a Vault agent into the pod
        vault.hashicorp.com/role: "vault-local"  # Role for Vault to access secrets, providing permission for the pod
        # Specifies which Vault secret path to inject
        vault.hashicorp.com/agent-inject-secret-config: "kv-v2/vault-local/local-k8s"  # Secret path to fetch from Vault
        # Template to format the secrets and inject them into the container as environment variables
        vault.hashicorp.com/agent-inject-template-config: |
          {{- with secret "kv-v2/data/vault-local/local-k8s" -}}
          export SECRET="{{ .Data.data.secret }}"  # Extract the "secret" field from the Vault secret
          {{- end }}
    spec:
      serviceAccountName: vault-local  # Service account with Vault permissions assigned to the pod
      containers:
      - name: app-vault
        image: matcham89/app:latest
        ports:
        - containerPort: 5000
        command:
        - "sh"
        - "-c"
        args:
        - ". /vault/secrets/config && exec python app.py"  # Load the Vault secrets into environment variables before running the app
        env:
        - name: APP_MESSAGE
          value: "Test Application"  # The environment variable for the application message

{% endraw %}
```

Update the service manifest

```bash
vim service.yaml
```
```yaml
apiVersion: v1

kind: Service
metadata:
  name: app-vault
spec:
  selector:
    app: app-vault
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
```


Deploy the new application & service:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Navigate to the Ingree folder and update the configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app
            port:
              number: 80
  - host: app-vault.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-vault
            port:
              number: 80
```

```bash
kubectl apply -f default.yaml
```

#### Verify Vault Integration
Check the pod status:
```bash
kubectl get pods
```
If the pod status shows an init container, it indicates that the Vault Agent Injector is working.

Update the etc host files
(remember to checkout the external loadbalncer IP and ensure the kind api is running)

```bash
vim /etc/hosts
```

```bash
# local cluster
172.18.0.5 app.local
172.18.0.5 app-vault.local
```

Test the application:
```bash
curl app-vault.local
```
Expected output:
```
    <html>
    <head><title>Application Title</title></head>
    <body>
        <h1>Test Application</h1>
        <p><strong>Vault Secret:</strong> this is a secret stored in vault and exported with vault injector</p>
    </body>
    </html>
```

You can also verify the injected secret directly:
```bash
kubectl exec -i -t app-XXXXXX -c app-vault -- cat /vault/secrets/config
```
Expected output:
```bash
export SECRET="this is a secret stored in vault and exported with vault injector"  # Extract the "secret" field from the Vault secret
```

### Conclusion
We have successfully integrated Vault into our Kubernetes application to dynamically inject secrets. 

This setup forms the foundation for managing secrets securely in production environments.
