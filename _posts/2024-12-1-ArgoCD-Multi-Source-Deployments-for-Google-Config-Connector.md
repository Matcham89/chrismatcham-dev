---
classes: wide
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
---


<br />

ArgoCD has become a cornerstone for managing Kubernetes deployments, offering a declarative GitOps approach to continuous delivery. As deployments grow in complexity, the need for multi-source deployments becomes apparent, allowing for greater flexibility and integration with various tools. Recently, I aimed to integrate Google Config Connector as an additional source to support ArgoCD and Kubernetes management. This blog explores the journey, challenges, and solutions I discovered along the way.

#### Existing Components

- Deployed Kubernetes Cluster  
- ArgoCD deployment  
- Google Config Connector deployment  
- GitHub repository  
- CircleCI pipeline  

#### Understanding ArgoCD and Multi-Source Deployments

ArgoCD is a powerful tool designed for Kubernetes continuous delivery, ensuring that the desired state of applications as defined in Git repositories is achieved and maintained. Multi-source deployments in ArgoCD refer to the ability to pull configurations from multiple repositories or sources, enhancing modularity and flexibility in managing complex deployments.

##### Benefits of Multi-Source Deployments

- Manage different components of an application from separate repositories.  
- Easily scale deployments by adding new sources.  
- Seamlessly integrate various tools and services.  

#### The Need for Google Config Connector

Google Config Connector (GCC) allows you to manage Google Cloud resources through Kubernetes, providing a unified management approach. Integrating GCC with ArgoCD enhances the capability to manage both Kubernetes and Google Cloud resources efficiently.

##### Advantages of Integration

- Unified management of Kubernetes and cloud resources.  
- Enhanced automation for provisioning and managing resources.  
- Consistent configurations across environments.  

#### The 'App of Apps' Deployment Method

The 'app of apps' pattern in ArgoCD is a hierarchical approach where a single parent application manages multiple child apps. This method organizes complex deployments into manageable components.

##### Challenges

- YAML configuration issues due to its strict syntax.  
- Long pipeline durations when syncing numerous apps sequentially.  
- Difficulty managing deployment commands efficiently.  

#### Solutions

1. **YAML Linters**: Catch syntax errors early.  
2. **Modular Templates**: Maintain consistency and reduce redundancy.  

##### ArgoCD Application Manifest

The ArgoCD application manifest YAML I came up with is as follows:

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  labels:
    argo-name: $APPLICATION-NAME
  name: $APPLICATION-NAME
  namespace: argocd
spec:
  project: $PROJECT
  sources:
    - repoURL: 'git@github.com:$ORG/$REPO.git'
      path: gitops/$PROJECT/charts/$APPLICATION-NAME
      targetRevision: HEAD
      helm:
        releaseName: $APPLICATION-NAME
    - repoURL: 'git@github.com:$ORG/$REPO.git'
      path: gitops/$PROJECT/charts/$APPLICATION-NAME/config-connector
      targetRevision: HEAD
  destination:
    name: $CLUSTER
    namespace: default
```

I provide this manifest to the parent application in the `app of apps`. With this approach, I am able to pool together the raw Google Config Connector resource manifest with the application Helm chart and values. This gives me a single, complete Helm template for ArgoCD.

#### Config Connector Integration

To ensure a smooth deployment and developer process, I decided to use raw Config Connector manifests provided by Google. The benefit is that engineers wanting to create a resource will have all the official documentation to support them. To achieve this, I added an additional directory to the application repo to support the ArgoCD application manifest:

```plaintext
config-connector/
  runbook.md
  pubsubsubscription.yaml
```

Now I needed to get the pipeline to work with the new files and directory. To achieve this, I considered two options:

1. Using an `if` statement in the pipeline (`if config-connector dir then x`).  
2. Providing a Config Connector directory for all applications by default.

I opted for the latter, as it is cleaner and more efficient, with the only requirement being that the directory is not empty. Thus, I included a `runbook.md` in the directory. This setup ensures that the pipeline does not fail if there is no Config Connector configuration, as not all applications require additional resources from Config Connector.

This method allows application engineers to use raw Google Config Connector manifests to create resources, avoiding the need to create a dynamic Helm template for each resource supported by the Config Connector.

The only required Helm template was an instruction to read the Config Connector directory. Additionally, I added lines to ignore service account creation to maintain control over resource creation.

```liquid
{% raw %}
{{- $files := .Files.Glob "config-connector/*.yaml" }}
{{- if $files }}
  {{- range $filename, $file := $files }}
    {{- $content := $.Files.Get $filename }}
    {{- $docs := splitList "---" $content }}
    {{- range $index, $doc := $docs }}
      {{- $trimmed := $doc | trim }}
      {{- if $trimmed }}
        {{- $manifest := fromYaml $trimmed }}
        {{- if $manifest }}
          {{- if not (eq $manifest.kind "ServiceAccount") }}
---
{{ $trimmed | nindent 0 }}
          {{- end }}
        {{- end }}
      {{- end }}
    {{- end }}
  {{- end }}
{{- else }}
# No config connector files found
{{- end }}      
{% endraw %}
```

The final step was to copy the contents of the Config Connector directory from the application repo to the central GitOps repo, allowing the application manifest to recognize and build the necessary resources. This was done using a `cp` step in the CircleCI pipeline.

#### CircleCI Pipeline

Improving CircleCI pipelines involves using the `argocd cli` commands in a more fine-grained fashion. To resolve the long wait times involved with the `app of apps` sync times, I used the resource flag to sync only the one child app inside the parent.

```sh
argocd app sync $PARENT-APP \
--resource argoproj.io:Application:$NAMESPACE/$APPLICATION-NAME \
--server $SERVER \
--auth-token $TOKEN \
--grpc-web
```

Once the sync is complete and successful, I sync the child application as well.

```sh
argocd app sync $APPLICATION-NAME --server $SERVER \ 
  --auth-token $TOKEN --grpc-web
```

Again, once a successful sync occurs, the pipeline will continue and complete.

#### Conclusion

Integrating Google Config Connector with ArgoCD for multi-source deployments can significantly enhance your Kubernetes management capabilities. While challenges such as YAML configuration issues and pipeline delays exist, employing best practices and optimized strategies can mitigate these problems effectively. As you explore these techniques, remember that the goal is to streamline and simplify your deployment processes.