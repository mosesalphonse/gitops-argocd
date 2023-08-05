# ArgoCD - GitOps

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

It can be integrated with Git repositories, and used jointly with CI tools (Jenkins, Github-actions) to define end-to-end CI/CD pipeline to automatically build and deploy applications in Kubernetes.

## POC:
* Architecture overview of ArgoCD which is a declarative Continues Delivery tool for K8
* Install ArgoCD in K8 cluster with ArgoCD CLI
* Setup infra (deployment) repo (This repo is different than source code repo in general)
* Configurate application in ArgoCD and Sync between the infra repo and K8 cluster
* Verify

## Prerequisite

* Kubernetes Cluster
* Pre-built docker image in imgae Repo

## Architecture Overview of ArgoCD:

![GitOps-ArgoCD](https://github.com/mosesalphonse/gitops-argocd/assets/16347988/7075143c-f1c6-4795-a158-26bc206816f7)


**Note:**

In this demo (repo), we have not covered the below;
<li>
Source code repo
</li>
<li>
CI Pipeline
</li>

We have just covered the below components:
<li>
Image Repo
</li>
<li>
Infra (deployment) git repo
</li>
<li>
ArgoCD
</li>
<li>
K8 Cluster
</li>

# Steps:

## Install ArgoCD

```
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
## Install ArgoCD CLI

```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

rm argocd-linux-amd64

```

## Configurae and access ArgoCD API server

Change the argocd-server service type to LoadBalancer:
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

```
Grab the external IP of the ArgoCD Server:
```
kubectl -n argocd get svc

```
Note: Use this IP to access the UI console of ArgoCD-server in the default http port(80)

Grab the initial password of admin using ArgoCD CLI:
```
argocd admin initial-password -n argocd

```
Note: Login into the ArgoCD admin console using the username as 'admin' and the password you grabed from the above command
