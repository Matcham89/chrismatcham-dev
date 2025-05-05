---
classes: wide
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
  teaser: /assets/images/rancher-k8s.png
  og_image: /assets/images/rancher-k8s.png
---

<br />


# 🥷 Deploying Kagent MCP on Rancher Desktop with Helm & ArgoCD

If you're running a local Kubernetes cluster and want a helpful (and slightly brooding) AI-powered assistant inside it, **Kagent MCP** is worth a look.

In this guide, I’ll walk you through how I deployed Kagent on my local Rancher Desktop Kubernetes cluster, using **Helm** and **ArgoCD**—with a Naruto-inspired twist. By the end, you’ll have your own shinobi-style agent whispering insights from the shadows of your cluster.

---

## What You’ll Need

- Rancher Desktop (with Kubernetes enabled)
- Helm installed
- ArgoCD running in your cluster
- Git repo set up for ArgoCD
- An OpenAI API key (you’ll need to create an account and add some credit—$10 is enough to get started)
- kubectl configured for your local cluster

---

## Step 1: Install Kagent with Helm

First, add the Kagent Helm repo and install it into your cluster. Even though this is an OCI managed Helm Chart, we can still use delcartive templates, they just require a different format.

Sincle I am using ArgoCD, this my Application manifest looks like this:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kagent 
  namespace: argocd
spec:
  project: default
  destination:
    server: "https://kubernetes.default.svc"
    namespace: kagent
  sources:
    - repoURL: ghcr.io/kagent-dev/kagent/helm
      chart: kagent
      targetRevision: 0.3.0
      helm:
        values: |
          providers:
            default: openAI
            openAI:
              provider: OpenAI
              model: "gpt-4.1-mini"
              apiKeySecretRef: kagent-openai
              apiKeySecretKey: OPENAI_API_KEY
    - repoURL: ghcr.io/kagent-dev/kagent/helm
      chart: kagent-crds
      targetRevision: 0.3.0
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
```

Couple of things to note:

  1)  I am using multiple sources in this manifest, that is because current `Kagent` is a new service and the CRD's are installed seprately. 

  2) I am providing my API Key via a Kubernetes Secret.

  3) The ArgoCD application will create the namespace if needed.

## Step 2: Interacting with Kagent

Once deployed confirm your pod are healthy.

```bash
kubectl get pods -n kagent
```

Hopefully you will something like

_output_
```bash
NAME                      READY   STATUS    RESTARTS      AGE
kagent-86d876bf65-fw52c   3/3     Running   7 (35m ago)   93m
```

Now we can interact with your agent using kubectl port-forwarding.

```bash
kubectl get svc -n kagent
```
_output_
```bash
NAME     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)           AGE
kagent   ClusterIP   10.43.243.91   <none>        80/TCP,8081/TCP   98m
```


```bash
kubectl port-forward svc/kagent 8080:80 -n kagent
```

Then hit http://localhost:8080 in your browser.

![Welcome-Screen](../assets/images/welcome-screen.png)

## Success