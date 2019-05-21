# VELERO

## Overview

Velero gives you tools to back up and restore your Kubernetes cluster resources and persistent volumes. Velero lets you:
* Take backups of your cluster and restore in case of loss.
* Copy cluster resources to other clusters.
* Replicate your production environment for development and testing environments.

Velero consists of:
* A server that runs on your cluster
* A command-line client that runs locally

You can run Velero in clusters on a cloud provider or on-premises.

## Install it !

As CRD, it is installed into the cluster itself.

```
mkdir velero-demo
cd velero-demo
wget https://github.com/heptio/velero/releases/download/v0.11.0/velero-v0.11.0-linux-amd64.tar.gz
tar xvf velero-v0.11.0-linux-amd64.tar.gz
```

## Use Minio as Storage platform

Minio is an option if you want to keep your backup data on-premises and you are not using another storage platform that offers an S3-compatible object storage API.

Edit the file to change **ClusterIP** to **NodePort**
```
vi config/minio/00-minio-deployment.yaml
```

Then start the server and the local storage service. In the Velero directory, run:

```
kubectl apply -f config/common/00-prereqs.yaml
kubectl apply -f config/minio/
```

Check to see that both the Velero deployment is successfully created:

```
kubectl get deployments,po  --namespace=velero
```

Display the Service Port for **minio**

```
kubectl get svc -n velero minio
```

And one the Node IP
```
kubectl get nodes -o wide
```

## Deploy Nginx

```
kubectl apply -f config/nginx-app/base.yaml
kubectl get deployments --namespace=nginx-example
```

## Back up

Create a backup for any object that matches the app=nginx label selector:

```
velero backup create nginx-backup --selector app=nginx --include-namespaces nginx-example
velero backup get
```

## Simulate a disaster

```
kubectl delete namespace nginx-example

kubectl get deployments --namespace=nginx-example
kubectl get services --namespace=nginx-example
kubectl get namespace/nginx-example
```

## Restore

```
velero restore create --from-backup nginx-backup
```

The restore can take a few moments to finish. During this time, the STATUS column reads InProgress.

```
velero restore get
```

You can duplicate the namespaced environment. Please wait :)
```
velero restore create --from-backup nginx-backup --namespace-mappings nginx-example:production
velero restore create --from-backup nginx-backup --namespace-mappings nginx-example:production2
velero restore create --from-backup nginx-backup --namespace-mappings nginx-example:production3
...
```

Wait for the end of duplication and you will find new namespaces.

```
kubectl get ns

NAME             STATUS   AGE
cattle-system    Active   8d
default          Active   8d
ingress-nginx    Active   8d
istio-system     Active   2d3h
kube-public      Active   8d
kube-system      Active   8d
production       Active   16h
production2      Active   16h
production3      Active   16h
velero           Active   16h
```
