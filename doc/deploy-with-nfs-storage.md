# Deploy Gitlab to IBM Cloud with a NFS Storage

Have the IBM Cloud client (ibmcloud) and Kubernetes (kubectl) CLIs installed and working. See main article.

Have the IBM Cloud Object Storage already created and configured.


## . Edit file kubernetes/postgres/postgres-nfs.yaml

Change to your IBM Cloud Object Store

```YAML
nfs:
   server: "fsf-<CHANGE_ME>-fz.service.softlayer.com"
   path: "/<CHANGE_ME>/data01"
```


## . Edit file kubernetes/postgres/gitlab-nfs.yaml

Change to your NFS Server

```YAML
nfs:
   server: "fsf-<CHANGE_ME>-fz.service.softlayer.com"
   path: "/<CHANGE_ME>/data01"
```

## Run the scripts in the repo

Execute the scripts in the following order:

```
kubectl create -f kubernetes/postgres/postgres-nfs.yaml

kubectl create -f kubernetes/redis/redis-storage.yaml

kubectl create -f kubernetes/gitlab/gitlab-nfs.yaml
kubectl create -f kubernetes/gitlab/gitlab-storage.yaml

kubectl create -f kubernetes/ingress.yaml
```

After this, the GitLab container should be ready after 2 to 5 minutes.
