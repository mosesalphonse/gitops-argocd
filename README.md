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


## Setup Infra (deployment) repo

Please refer the sample infra repo (sash-quarkus) in this repo.

Your application folder must contain kustomization.yaml file in which you have to refer the workload manifest. Please refer the below sample code;

**kustomization.yaml**

```
namePrefix: kustomize-

resources:
- sash-quarkus-deployment.yaml
- sash-quarkus-svc.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
```
As per the above kustomization.yaml file, here are the sample deployment and service manifest

**sash-quarkus-deployment.yaml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sash-quarkus
spec:
  replicas: 1
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: sash-quarkus
  template:
    metadata:
      labels:
        app: sash-quarkus
    spec:
      containers:
      - image: gcr.io/iconic-nation-393113/sash-quarkus-async:v1
        name: sash-quarkus
        ports:
        - containerPort: 8080
```

**sash-quarkus-svc.yaml**

```
apiVersion: v1	
kind: Service	
metadata:	
  name: sash-quarkus-svc	
spec:	
  ports:	
  - port: 80
    targetPort: 8080
  selector:	
    app: sash-quarkus
```

## Configurate application in ArgoCD API server (using admin UI console)

**Setup Repo:**

Settings --> Repositories --> give the github URL and click 'Connect'

Screenshot after succuessful repo settings

![Screenshot from 2023-08-05 17-09-09](https://github.com/mosesalphonse/gitops-argocd/assets/16347988/9fdfb6c9-7a4c-4015-b6f1-b15fd76841a2)


**Setup Application:**

Applications --> New App --> And Enter only the below(mandatory) values

  * Application Name (free text)
  * Select Project Name (drop down, as of now select default)
  * Select Source Repo (since we have completed our repo settings earlier, our repo must present in the drop down)
  * Automatically eligible path will show (note, the folder should have kustomization.yaml and the respective valid yaml manifest), select the 
    correct path where your deployments manifest are stored
  * Select Destination Cluster URL, by default our own K8 cluster is configured
  * Enter the K8 namespace, you may enter 'default' (free text)
  * And enter 'CREATE' button

**Note** : if all the values fed correctly, your application will be created as shown in the below screenshot

![image](https://github.com/mosesalphonse/gitops-argocd/assets/16347988/b1ec95a3-d6f4-4f99-9c90-dc1de891b47d)

**Note** : By default Sync mode is MANUAL, so you hit the 'SYNC' button to synch between infra repo to K8 cluster

## Verify :

Make changes in the git infra repo's yaml file and hit the 'SYNC' button, you can see the changes in the K8 cluster and the ArgoCD API server admin console as well.
