# FLUX CD & Helm & Selead Secrets

## FluxCD

All doc [here](https://docs.fluxcd.io/projects/helm-operator/en/stable/)

### Prerequisites

Install `helm` & `fluxcd` CLI

```
brew install helm fluxcd
```
### Install Flux

* Clone your helm chart repository: 

```
 git@github.com:gaelletalend/flux-demo.git
```
* Create the `fluxcd` namespace:

```
kubectl create ns fluxcd
```

* Add FluxCD repository to Helm repos:
```
 helm repo add fluxcd https://charts.fluxcd.io
```
* Install Flux by specifying your fork URL:

```
$ helm upgrade -i flux fluxcd/flux --wait \
--namespace fluxcd \
--set git.url=git@github.com:gaelletalend/flux-demo \
--set git.branch=master

> Release "flux" does not exist. Installing it now.
NAME: flux
LAST DEPLOYED: Tue Nov 24 23:16:16 2020
NAMESPACE: fluxcd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Get the Git deploy key by either (a) running

  kubectl -n fluxcd logs deployment/flux | grep identity.pub | cut -d '"' -f2

or by (b) installing fluxctl through
https://docs.fluxcd.io/en/latest/references/fluxctl#installing-fluxctl
and running:

  fluxctl identity --k8s-fwd-ns fluxcd
```
  
  
* Install the `HelmRelease` Kubernetes CRD
```
kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/master/deploy/crds.yaml
```
* Install Flux Helm Operator with Helm v3 support: 
```
helm upgrade -i helm-operator fluxcd/helm-operator --wait \
--namespace fluxcd \
--set git.ssh.secretName=flux-git-deploy \
--set helm.versions=v3
```  

>[color=#cb61f4]The Flux Helm operator provides an extension to Flux that automates Helm Chart releases for it. A Chart release is described through a Kubernetes custom resource named HelmRelease. The Flux daemon synchronizes these resources from git to the cluster, and the Flux Helm operator makes sure Helm charts are released as specified in the resources.
  
* In order to sync your cluster state with Git you need to copy the public key and create a deploy key with write access on your GitHub repository.

```
fluxctl identity --k8s-fwd-ns fluxcd
```

### Deploying with Flux

* helmRelease resource: 

```
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: helloworld
  namespace: default
  annotations:
    fluxcd.io/automated: "true"
    fluxcd.io/tag.chart-image: glob:master-*
spec:
  releaseName: helloworld
  helmVersion: v3
  chart:
    git: ssh://git@github.com/gaelletalend/flux-demo
    path: charts/helloworld
    ref: master
  values:
    image: "docker.io/khaly/app-node:master-20201125"
    replicaCount: 2
    extraEnv:
      message: Hello world !
```