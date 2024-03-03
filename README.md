# argocd-segmentation

This project aims to use only one ArgoCD and secure a project tenant.

## Prerequisite (Option)

> You need:
> - one or more Kubernetes Cluster ArgoCD up and running
> - a github or gitlab repo to store your folder structure

### Test with a kind Cluster

> (my local dev by Sphinxgaia)[https://github.com/Sphinxgaia/my-local-dev]


## Install RAW ArgoCD


```sh
kubectl create ns argocd
```

Use external isntaller (to be up to date - `patch needed`)

```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

or already patched (sigle argocd mode)

```sh
kubectl apply -n argocd -f argocd-install/install.yaml
```


> Connect locally to argocd server 

get admin password

```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

ingress

```sh
kubectl apply -f argocd-install/ingress.yaml
```

port-forward

```sh
kubectl port-forward -n argocd service/argocd-server 8443:443
```


### Restricted applications ArgoCD / Project (single argocd)

#### Patch for applications / Namespace

patch deployment `argocd-server and application-controller`

```yaml
...
  - args:
    - ...
    - --application-namespaces=*

```

patch deployment `applicationset-controller`

```yaml
...
  - args:
    - ...
    - --applicationset-namespaces=*
```

patch configmap `argocd-cmd-params-cm`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cmd-params-cm
  ...
data:
  applicationsetcontroller.allowed.scm.providers: https://github.com/

```

## Namespace argocd (multi argocd instance)

```sh
kubectl create ns zonea-argocd

kubectl create ns zoneb-argocd
```

Install argocd zone a

```sh
kubectl apply -n zonea-argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/namespace-install.yaml
```

Install argocd zone b

```sh
kubectl apply -n zoneb-argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/namespace-install.yaml
```

Patch clsuterrolebinding if zone manage themselves 

```sh
kubectl apply -f argocd-install/clusterrolebinding.yaml
```


## Create kind clusters

Create zone a

```sh
export KUBECONFIG=$(pwd)/wescale-zonea-app1.yaml

kind create cluster --name wescale-zonea-app1
```

Create zone a

```sh
export KUBECONFIG=$(pwd)/wescale-zoneb-app2.yaml

kind create cluster --name wescale-zoneb-app2
```

## Config (TBU)


ArgoCD Config

1. Create app of apps (Self Management)

Env config:

1. Create project tenant + project dev
2. Create ns/quota (destination)
3. Create app of apps env
4. Create app

Update self management cluster zonea

```sh
kubectl apply -f root_conf/zonea/kind-wescale-zonea/project-manager-zonea.yaml
kubectl apply -f root_conf/zonea/kind-wescale-zonea/app-of-apps-infra.yaml
```


Update self management cluster zoneb

```sh
kubectl apply -f root_conf/zoneb/kind-wescale-zoneb/project-manager-zoneb.yaml
kubectl apply -f root_conf/zoneb/kind-wescale-zoneb/app-of-apps-infra.yaml
```


## RBAC Isolation (TBD)

## Manage secrets with Vault (TBD)