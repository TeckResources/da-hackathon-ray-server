# Sandbox Infrastructure

This repo is used for sandbox infrastructure for Devs to play around in the platform

## Structure 
This repo is connected via gitops to our [Argo CD](https://argo-cd.platform-dev.da.teck.com/). 
It is using a simple directory structure that allows users to find the resources deployed to a given instance easily. 

The argo-apps directory contains all of the argo applications that link deployment folders to argo gitops. 
An example is shown below with inline comments - 
```yaml
  project: core-example-dev # name of the argo project that holds this app
  source:
    path: dev/core-example-dev-1 # path in this repo to find deployment manifests 
    repoURL: https://github.com/TeckResources/core-example-infra # git repo connection
    targetRevision: HEAD # branch or revision - can be used to point to a feature branch
    directory:
      recurse: true
  destination:
    server: "https://kubernetes.default.svc"
    namespace: core-example-dev-1 # the kubernetes namespace that will contain the deployed infra
  syncPolicy:
    managedNamespaceMetadata:
      labels:
        da-project: core-example-dev # label applied to namespace for policy automation
    automated: # enable auto sync - changes are auto rolled out on the cluster
      selfHeal: true # overwrite any changes done via clickops or kubectl
```

## Usage
In order to change the infrastructure managed by an argo gitops application 
you can navigate to the given path in the application to see and edit the manifests.

An example of this may be to edit the image tag used in a deployment.yaml

### Creating a new argo app
1. Copy the example-app.yaml and rename
2. Edit the new application to update the required fields as shown below - 
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: core-example-dev-1 # Update to unique ref - ex core-example-dev-2
  namespace: argo-cd
  labels:
    env: dev
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: sandbox
  source:
    path: dev/core-example-dev-1 # Update to point to the path you will use in step 3
    repoURL: https://github.com/TeckResources/core-example-infra
    targetRevision: HEAD
    directory:
      recurse: true
  destination:
    server: "https://kubernetes.default.svc"
    namespace: sandbox-... # Update to a unique namespace using convention core-example-dev-[unique]
  syncPolicy:
    managedNamespaceMetadata:
      labels:
        da-project: sandbox
    automated:
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - ServerSideApply=true
      - Prune=confirm
    retry:
      limit: -1 # Unlimited
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

```
3. Copy the example-infra directory and rename to [argoPathfromStep2]
4. Navigate to the [Argo UI](https://argo-cd.platform-dev.da.teck.com/) to see your application running

### Clean-up
Please ensure to follow good development practices and clean up your deployments when you are done.
If argo app has prune enabled then just prune them from the argo UI and delete the infra from the repo here. 

## Policy
Note this is a sandbox and will be unstable and wiped at any time. 
