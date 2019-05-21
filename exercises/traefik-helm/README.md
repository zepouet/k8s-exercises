# jenkins-mastering-k8s

## Jenkins Service

You can use it through
* Docker container
* K8S deployment

https://github.com/eldada/jenkins-in-kubernetes

## How to deploy Traefik

We will deploy it as Ingress not as DaemonSet.

`rbac.enabled=true` to the parameters when running the helm install command creates the necessary account that Traefik needs to run in the kube-system namespace.


```
export KUBECONFIG=/Users/nicolas/software/k8s-1-13-1-do-kubeconfig.yaml
kubectl get nodes

helm init

kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

helm install stable/traefik --set dashboard.enabled=true,dashboard.domain=dashboard.domain,rbac.enabled=true --name demo

```

Then follow these instructions

```
NOTES:

1. Get Traefik's load balancer IP/hostname:

     NOTE: It may take a few minutes for this to become available.

     You can watch the status by running:

         $ kubectl get svc demo-traefik --namespace default -w

     Once 'EXTERNAL-IP' is no longer '<pending>':

         $ kubectl describe svc demo-traefik --namespace default | grep Ingress | awk '{print $3}'

2. Configure DNS records corresponding to Kubernetes ingress resources to point to the load balancer IP/hostname found in step 1
```

## Traefik Dashboard

Update your configuration `/etc/hosts`
And try to access http://dashboard.domain:NODEPORT/dashboard/

**Todo**
* Tiller TLS configuration
