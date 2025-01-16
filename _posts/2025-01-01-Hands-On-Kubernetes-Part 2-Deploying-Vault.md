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
curl app-a.local
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
kubectl exec --stdin=true --tty=true vault-0 -n vault -- /bin/sh
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

### Step 4: Configure Kubernetes Deployment
Create a service account, role, and role binding:
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

Update the application deployment:
```liquid
{% raw %}
# app-b/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-b
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-b
  template:
    metadata:
      labels:
        app: app-b
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "vault-local"
        vault.hashicorp.com/agent-inject-secret-config: "kv-v2/vault-local/local-k8s"
        vault.hashicorp.com/agent-inject-template-config: |
          {{- with secret "kv-v2/data/vault-local/local-k8s" -}}
          export SECRET="{{ .Data.data.secret }}"
          {{- end }}
    spec:
      serviceAccountName: vault-local
      containers:
      - name: app-b
        image: matcham89/app:latest
        ports:
        - containerPort: 5000
        command:
        - "sh"
        - "-c"
        args:
        - ". /vault/secrets/config && exec python app.py"
        env:
        - name: APP_MESSAGE
          value: "Application B"
{% endraw %}
```
Deploy the application:
```bash
kubectl apply -f app-b/deployment.yaml
```

#### Verify Vault Integration
Check the pod status:
```bash
kubectl get pods
```
If the pod status shows an init container, it indicates that the Vault Agent Injector is working.

Test the application:
```bash
curl app-b.local
```
Expected output:
```
Environment-Application
  Application B
  Vault Secret: this is a secret stored in vault and exported with vault injector
  Other Secret: Not Set
```

You can also verify the injected secret directly:
```bash
kubectl exec -i -t app-b-XXXXXX -c app-b -- cat /vault/secrets/config
```
Expected output:
```
export SECRET="this is a secret stored in vault and exported with vault injector"
```

### Conclusion
We have successfully integrated Vault into our Kubernetes application to dynamically inject secrets. This setup forms the foundation for managing secrets securely in production environments.
