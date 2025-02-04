---
classes: wide
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
  teaser: /assets/images/KinD_logo.png
  og_image: /assets/images/KinD_logo.png
---


<br />

When diving into Kubernetes, having a reliable local development environment is crucial. In this post, I'll walk through setting up a local Kubernetes cluster using KIND (Kubernetes IN Docker) and deploying a simple application to it.

#### Assumptions

- **Quick Read:** [link](https://chrismatcham.dev/Local-Kubernetes-Development-A-Journey-to-KIND/)
- **Laptop / Machine to run KIND:** (Linux/Mac/Windows)
- **Docker Engine installed and logged in:** [guide](https://docs.docker.com/engine/install/)
- **kubectl installed:** [guide](https://kubernetes.io/docs/tasks/tools/)
- **IDE installed:** (vim/nvim/Visual Studio Code)
- **Go installed:** [guide](https://go.dev/doc/install)



#### KIND Installation

The installation steps are straightforward and documented well [here](https://kind.sigs.k8s.io/docs/user/quick-start/).



#### Setting Up Our Cluster

Let's start by creating a KIND cluster with one control plane node and two worker nodes running Kubernetes v1.32.0.

We can create this using a simple configuration file and the KIND CLI:

##### Create the cluster by running:
```sh
vim kind-config.yaml
```

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

`Exit vim by typing ":wq"`

> w=write (save)

> q=quite

Create the cluster by running:

```sh
kind create cluster --config kind-config.yaml
```
View the nodes with:

```sh
kubectl config get-contexts
```

_Output_
```sh
CURRENT   NAME        CLUSTER     AUTHINFO    NAMESPACE
*         kind-kind   kind-kind   kind-kind 
```

Select Context:

```sh
kubectl config use-context kind-kind
```

_Output_
```sh
Switched to context "kind-kind"
```

View Nodes:
```sh
kubectl get nodes
```

_Output_
```sh
kind-control-plane   Ready    control-plane   43h   v1.32.0
kind-worker          Ready              43h   v1.32.0
kind-worker2         Ready              43h   v1.32.0
```

#### Test Application

We'll use a simple Flask web application that demonstrates environment variable configuration and secret management. 

Here's the current structure and the application:

```sh
touch app.py
touch Dockerfile
```

```sh
.
├── app.py
├── Dockerfile
└── kind-config.yaml
```

Update the `app.py` with the below:

```py
# app.py
from flask import Flask, render_template_string
import os

app = Flask(__name__)

@app.route("/")
def home():
    app_message = os.getenv("APP_MESSAGE", "Default Message")
    secret = os.getenv("SECRET", "Not Connected")

    # HTML Template to render in the browser
    html_template = """
    <html>
    <head><title>{{ title }}</title></head>
    <body>
        <h1>{{ app_message }}</h1>
        <p><strong>Vault Secret:</strong> {{ secret }}</p>
    </body>
    </html>
    """
    return render_template_string(html_template,
                                  title="Application Title",
                                  app_message=app_message,
                                  secret=secret)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

We containerize this application using a straightforward Dockerfile:

```sh
# DOCKERFILE
FROM python:3.9-slim
WORKDIR /app
COPY app.py /app/
RUN pip install flask
CMD ["python", "app.py"]
```

Log into Docker 
(change the username)

```sh
docker login -u USERNAME
```

Nothing fancy here, build the application and tag it with "app:latest".

```sh
docker build . -t USERNAME/app:latest
docker push USERNAME/app:latest
```

Push it to your local registry.

#### Deploying to Kubernetes

With our application containerized, we can deploy it to our cluster.


Create the following files:
```sh
mkdir app && touch app/deployment.yaml && touch app/service.yaml

mkdir ingress && touch ingress/default.yaml
```

Navigate to each file and populate it accordingly with the below manifests

Deployment manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: matcham89/app:latest
        ports:
        - containerPort: 5000
        env:
        - name: APP_MESSAGE
          value: "Test Application"

```
Service manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  selector:
    app: app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
```

From within the application directory, deploy the manifests:

```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

#### Setting Up Ingress Controller

KIND provides its own ingress controller, which we can deploy with. 

Detailed [here](https://kind.sigs.k8s.io/docs/user/ingress/).

```sh
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

Define The Application Ingress

Add this configuration to the default.yaml file created earlier in the ingress folder.

The Ingress configuration maps requests to our previously created service.

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
```

Apply the maniest:

```sh
kubectl apply -f default.yaml
```


To handle external IPs locally, we'll need KIND's cloud provider. Detailed [here](https://kind.sigs.k8s.io/docs/user/loadbalancer/).


```sh
go install sigs.k8s.io/cloud-provider-kind@latest
sudo install ~/go/bin/cloud-provider-kind /usr/local/bin
```

Once installed, fire up the tool with the below command:

```sh
cloud-provider-kind
```

We must leave the tool running in a terminal whilst we are working on our local cluster with externalIP's.

Once the tool is "running" you will see some output similar to this:

```sh
43 Protocol:TCP} Cluster:[{Address:172.18.0.3 Port:31870 Protocol:TCP} {Address:172.18.0.4 Port:31870 Protocol:TCP}]} 
IPv4_80_TCP:{Listener:{Address:0.0.0.0 Port:80 Protocol:TCP} Cluster:[{Address:172.18.0.3 Port:32131 Protocol:TCP} 
{Address:172.18.0.4 Port:32131 Protocol:TCP}]}] SessionAffinity:None SourceRanges:[]}        
```

Do not be discouraged if you see some output "server misbehaving" - - this is expected as it boots up.


Running `cloud-provider-kind` will attach an external IP to our ingress controller. In my case, it looks like this:

```sh
ingress-nginx-controller   LoadBalancer   10.96.210.134   172.18.0.6    80:30748/TCP,443:31112/TCP   24h    
```


#### Final Configuration

The last step is to update our `/etc/hosts` file to route traffic to our applications:  

```sh
sudo vim /etc/hosts

# local k8s dev 
172.18.0.6 app.local
```

*(Be sure to update the IP address with the output from your controller)*



#### What We've Accomplished

At this point, we have:

1. A functioning Kubernetes cluster with one control plane and two worker nodes.
2. Two deployed applications with their own services.
3. An ingress controller handling external traffic.
4. Local DNS resolution for our applications.

You can now access the applications by curling it:

```sh
curl app.local 
```

```sh
    <html>
    <head><title>Application Title</title></head>
    <body>
        <h1>Test Application</h1>
        <p><strong>Vault Secret:</strong> Not Connected</p>
    </body>
    </html>
```

#### Key Takeaways

- **KIND** provides a robust local Kubernetes environment.
- The built-in ingress controller simplifies local access to applications.
- Environment variables offer a flexible way to configure applications.
- Local DNS configuration enables seamless access to your services.



This setup provides a solid foundation for local Kubernetes development. In the next part, we'll explore further and introduce tools like **Vault** for secret management.
