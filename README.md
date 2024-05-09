# Workshop On Deploy Application in Kubernetes Using GitOps & ArgoCD

Hello everyone, welcome to the workshop on "Deploy Application in Kubernetes Using GitOps & [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)". 

In this workshop we will be working on the following tasks.

Task 1: Deploy an application in your k8 cluster using GitOps and [ArgoCD](https://argo-cd.readthedocs.io/en/stable/).

Task 2: Deploy a helm chart in your k8 cluster using GitOps and [ArgoCD](https://argo-cd.readthedocs.io/en/stable/).

Along with these tasks, we will be seeing different functionalities, scopes and best practices of [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) and GitOps.

## Pre-Requisite

To follow along with this workshop, you will require a basic understanding of the followings:

1. Basic knowledge of [Kubernetes](https://kubernetes.io/) and its different resources and components (such as: Deployments, Pods, Services etc).
2. Basic understanding of [Helm](https://helm.sh/).
3. Basic idea of [yaml](https://yaml.org/).
4. [Kubernetes manifests file](https://spacelift.io/blog/kubernetes-manifest-file#what-is-a-kubernetes-manifest-file).
5. Basic usage of [Kubectl](https://kubernetes.io/docs/tasks/tools/).
6. Basic understanding of [Docker](https://docker.io/). 

If you full-fill all the requirements, then you are good to go.

## Setup Environment

Let's start with setting our environment. To start working with [ArgoCD](https://argo-cd.readthedocs.io/en/stable/). 

For this workshop we will be working on our local machine. We'll setup [Kubernetes](https://kubernetes.io/) cluster using [Kind](https://kind.sigs.k8s.io/) deploy [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) on our machine. 

For production environment, you will need to setup [Kubernetes](https://kubernetes.io/) production cluster. [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) setup is same for any environment.

So, before proceeding further, let's setup the environment. To setup environment for the workshop, your local machine should have the following tools pre installed.

* [Docker](https://docker.io/)
* [Kind](https://kind.sigs.k8s.io/)
* [Kubectl](https://kubernetes.io/docs/tasks/tools/)

## Create a new Kubernetes cluster

We will use [kind](https://kind.sigs.k8s.io/) to create a new cluster on your local machine. Run the following command,

```
kind create cluster --name kcd-cluster-one
```

You can check the cluster context and use it.

```
kubectl config get-contexts

kubectl config use-context kind-kcd-cluster-one
```

## Install ArgoCD

For more info [click here](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd)

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait until all the required pods are running.

### Get ArgoCD Admin Password

```
kubectl get secrets/argocd-initial-admin-secret -n argocd --template={{.data.password}} | base64 -d
```

### ArgoCD Dashboard

Since we are working on local machine, we will be exposing the Argocd dashboard through port forwarding. In 
production, you can use kubernetes [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#what-is-ingress) or 
[LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/) to expose ArgoCD Web UI.

Run the following command to port-forward the argocd server. You can expose it to any port.

```
kubectl port-forward -n argocd svc/argocd-server 8080:80
```

Login with the username "admin" and password admin password from secret. 

## Install ArgoCD CLI

Install ArgoCD Cli tool to connect argocd from terminal. The installation guidelines for each operating system are 
documented [here](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

You can found all the command reference of ArgoCD Cli from [here](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/).

## Login to ArgoCD from CLI

Since we are running our argocd in local machine using port forwarding, to connect with the argocd we need to
provide the localmachine IP, port and admin credentials. In production, you will need to provide the exposed ArgoCD URL for login along with
the admin credentials of ArgoCD.

Run the following command in your machine. **Make sure the port-forward is running, before executing the command**. To connect with the production ArgoCD, replace the server host and port with your
ArgoCD URL along with the admin username and password.

```
argocd login localhost:8080 --insecure \
--username admin \
--password $(kubectl get secrets/argocd-initial-admin-secret -n argocd --template={{.data.password}} | base64 -d)
```

Check if its successfully connected or not.

```
argocd cluster list
```

## Let's Get Started

### Task 1 - Deploy Application

Create a new application from ArgoCD UI and deploy the application. [Check here](https://github.com/shaekhhasanshoron/kcd-workshop-gitops-argocd/tree/main/manifests/demo-app) for the application descriptors.

We are going to deploy [kcd-demo-app](https://github.com/shaekhhasanshoron/kcd-demo-app) in ArgoCD.

```
REPO URL: https://github.com/shaekhhasanshoron/kcd-demo-app
PATH: kubernetes/manifests
REVISION: HEAD
```

### Task 2 - Automated vs Manual Sync

In manual sync we need to manually sync the updates or commits. ArgoCD supports both of the sync system. However, If you consider GitOps principles, 
application it needs to automated. 


### Task 3 - Update manifests

We will update the manifests to check the output. We will update it from Git also from Cluster it-self. 
We will update the configurations inside the server directly and we will see that ArgoCD will alter that change and move it to
the desired state.


### Task 4 - Redeploy and Rollback to Previous Deployment

Let us see how the re-deploy and rollback feature.

### Task 5 - Create a new Project

We will create a new project and add another application to that newly created project.

### Task 6 - Adding Roles to Project

We will set different roles to project.

### Task 7 - Add a Git repositories 

We will add a git repo in ArgoCD git repo list. We will add [kcd-notify-app](https://github.com/shaekhhasanshoron/kcd-notify-app) repo. Run the following command. Here, you will attach token instead of password.
```
argocd repo get https://github.com/shaekhhasanshoron/kcd-notify-app
```

Now check the repo list.
```
argocd repo list
```

### Task 8 - Create a new user

You need to update the `argocd-cm` configmap and add user data.

```
kubectl edit cm -n argocd argocd-cm
```

Add the following code under `data` and save it.

```
  accounts.shoron: apiKey, login
  accounts.shoron.enabled: "true"
```

Check User list

```
argocd account list
```

### Task 9 - Update the user password

Since initially newly created your will not have any password, we have to add the password for the new user.

```
argocd account update-password \
  --account shoron \
  --current-password $(kubectl get secrets/argocd-initial-admin-secret -n argocd --template={{.data.password}} | base64 -d) \
  --new-password Hello@123
```


### Task 10 - Provide Permission: RBAC

By default, new user does not have any permission. You need to provide the permissions to user.
You need to add user permission in a `argocd-rbac-cm` configmap.

```
kubectl edit cm -n argocd argocd-rbac-cm
```

Here is the following format to update permission for user.

`p, <role/user/group>, <resource>, <action>, <appproject>/<object>, <allow/deny>`

Now edit `argocd-rbac-cm` configmap and  add the following code under `data` and save it.

```
  policy.csv: |
    p, shoron, applications, *, default/*, allow
  policy.default: read:readonly
```

You can add several permissions to it.

```
  policy.csv: |
    p, shoron, applications, *, default/*, allow
    p, shoron, clusters, get, *, allow
    p, shoron, projects, get, default, allow
    p, shoron, repositories, *, *, allow
```


### Task 11 - Disable user

You need to update the `argocd-cm` configmap and add user data.

```
kubectl edit cm -n argocd argocd-cm
```

### Task 12 - Deploy a new Application in Project

Create a new application from ArgoCD UI and deploy the application. [Check here](https://github.com/shaekhhasanshoron/kcd-workshop-gitops-argocd/tree/main/manifests/demo-app) for the application descriptors.

We are going to deploy [kcd-notify-app](https://github.com/shaekhhasanshoron/kcd-notify-app) in ArgoCD.

```
REPO URL: https://github.com/shaekhhasanshoron/kcd-notify-app
PATH: kubernetes/manifests
REVISION: HEAD
```

Here we will use ArgoCD CLI to deploy.

```
argocd app create argocd/applications/kcd-notify.yaml
```

### Task 13 - Hooks

Hooks can be used while synchronizing your application with ArgoCD. For details [click here](https://argo-cd.readthedocs.io/en/stable/user-guide/resource_hooks/)

Here just to show how does it work, we will create a new file under the manifest folder and add the following code. We are just
notify a message to notify service api.

```
apiVersion: batch/v1
kind: Job
metadata:
  generateName: kcd-notify-config-
  annotations:
    argocd.argoproj.io/hook: PostSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
    argocd.argoproj.io/sync-wave: "1"
spec:
  template:
    spec:
      containers:
        - name: kcd-notify
          image: curlimages/curl
          command:
            - "curl"
            - "-X"
            - "POST"
            - "http://kcd-notify.demo:8080/api/notify?message=Synched"
      restartPolicy: Never
  backoffLimit: 2
```

### Task 14 - Removing Application

We will remove the application.

```
argocd app delete <app name>
```

### Task 15 - Create a new Helm Application from Public Helm Repo

We will create a new application from a public helm repository. We will deploy [WildFly](https://www.wildfly.org/) app server from [https://charts.bitnami.com/bitnami](https://charts.bitnami.com/bitnami) helm repo.
We will update parameters and deploy the applications. 

```
REPO URL : https://charts.bitnami.com/bitnami
CHART NAME: wildfly
VERSION: 19.1.1
```

### Task 16 - Create a Helm Application from Git Repo with custom Values

We have already extracted the [WildFly](https://www.wildfly.org/) app server chart from [https://charts.bitnami.com/bitnami](https://charts.bitnami.com/bitnami) public repo.
View the app repository, [kcd-wildfly](https://github.com/shaekhhasanshoron/kcd-wildfly).

```
REPO URL: https://github.com/shaekhhasanshoron/kcd-wildfly
PATH: helm/wildfly
REVISION: HEAD
```

### Task 17 - Take Backup and Restore

We will take a backup of the current state. Run the following command.

```
argocd -n argocd admin export > backup.yaml
```

We will update some changes and then import the backup file and check what happens.

```
argocd -n argocd admin import - < backup.yaml
```

## Thank you
Please visit [Klovercloud.com](https://klovercloud.com/).  