Deploying Gitlab to IBM Cloud
=============================

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

The scripts here are used to deploy GitLab on IBM Cloud using a Kubernetes Service.

IBM Cloud already has a Git repository for free in some data certers. But in my case customer cannot put the source code outside of the country, so i need to deploy GitLab.


# Available examples

* Deploy GitLab without persistent storage (This page).
* Draft [Deploy GitLab with NFS](https://github.com/ebasso/ibm-cloud-deploy-gitlab/tree/master/doc/deploy-with-nfs-storage.md)
* Draft [Deploy GitLab with IBM Cloud Block Storage](https://github.com/ebasso/ibm-cloud-deploy-gitlab/tree/master/doc/deploy-with-cloud-storage.md)


# 1. Prerequisites


## 1.1 Cloning Repository

Clone the repository

```bash
git clone https://github.com/ebasso/ibm-cloud-deploy-gitlab.git

cd ibm-cloud-deploy-gitlab
```

## 1.2 Install ibmcloud and kubectl

Have the IBM Cloud client (ibmcloud) and Kubernetes (kubectl) CLIs installed and working.

## 1.3 Login to IBM Cloud and Configure environment

Login to IBM Cloud

```
ibmcloud login
```

Target the right organization and space

```
ibmcloud target --cf
```

## 1.4 (Only first time) Install the IBM Cloud Container Service plug-in

```
ibmcloud plugin install container-service
```

# 2 Deploying

## 2.1 Check for existing clusters or create one

Test if you have any clusters set-up already, or create a new cluster from the web-based UI, because of multiple options.

```
$ ibmcloud ks clusters
  
  OK
  Name            ID                     State    Created       Workers   Datacenter   Version                   Resource Group Name   Provider
  eb-k8s-cluster  aaaaaaaaaaaaaaaaaa01   normal   2 days ago    1         ams03        1.16.9_1529               rg-dev                classic
  eb-oc31-cluster bbbbbbbbbbbbbbbbbb02   normal   1 month ago   3         sao01        3.11.200_1548_openshift   rg-dev                classic
```

## 2.2 Download and set the configuration for your cluster

```
ibmcloud ks cluster-config --cluster ek8s-cluster
```

you can confirm

```
kubectl config current-context
```

## 2.3 Run the scripts in the repo

 I suggest you to open Kubernetes console and follow when each component is ready!

Execute the scripts in the following order:
```
kubectl create -f kubernetes/postgres/postgres.yaml
kubectl create -f kubernetes/redis/redis.yaml
kubectl create -f kubernetes/gitlab/gitlab.yaml
kubectl create -f kubernetes/ingress.yaml
```

After this, the GitLab container should be ready after 2 to 5 minutes.



# Access GitLab

For Standard cluster: the Gitlab will be now accessible at ek8s-cluster.us-south.containers.mybluemix.net. Happy coding!


## (For Free Cluster) Get the Public IP and Port for your GitLab

Run the following commands to get your public IP and NodePort number.

```
ibmcloud ks workers --cluster ek8s-cluster

  OK
  ID                                                 Public IP        Private IP      Flavor   State    Status   Zone    Version
  kube-hou02-pa817264f1244245d38c4de72fffd527ca-w1   159.122.179.48   10.144.215.48   free     normal   Ready    mil01   1.16.9_1531

$ kubectl get svc -o wide

  NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                     AGE     SELECTOR
  gitlab       NodePort    172.21.207.31    <none>        80:30080/TCP,22:30022/TCP   60s     app=gitlab,tier=frontend
  kubernetes   ClusterIP   172.21.0.1       <none>        443/TCP                     2d19h   <none>
  postgresql   ClusterIP   172.21.137.242   <none>        5432/TCP                    46m     app=gitlab,tier=postgreSQL
  redis        ClusterIP   172.21.73.3      <none>        6379/TCP                    46m     app=gitlab,tier=backend
```

With the "Public IP" and "PORT", you can access your newly deployed gitlab. In this example http://159.122.179.48:30080

Note: The administrator username is 'root'.

# Delete the environment

If you need to customization or try again, you can delete the environment with commands

```bash
kubectl delete -n default deployment postgresql
kubectl delete -n default service postgresql
kubectl delete -n default persistentvolumeclaim gitlab-data-claim
kubectl delete -n default persistentvolume gitlab-postgres-pv

kubectl delete -n default deployment redis
kubectl delete -n default service redis
kubectl delete -n default persistentvolumeclaim gitlab-redis-claim

kubectl delete -n default deployment gitlab
kubectl delete -n default service gitlab

kubectl delete -n default ingress gitlab
```

ignore error is you don't have storage
