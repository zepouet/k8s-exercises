## Lister les Namespaces disponibles

```
kubectl get ns

NAME          STATUS    AGE
default       Active    6d
kube-public   Active    6d
kube-system   Active    6d
```

Comme toutes ressources il est possible de la labeliser mais bien plus encore.

```
kubectl describe ns default
Name:   default
Labels: <none>
Status: Active

No resource quota.

No resource limits.
```

## Créer son propre namespace

```
apiVersion: v1
kind: Namespace
metadata:
  name: test
```

On peut désormais l'ajouter 

```
kubectl create -f mynamespace.yml
```

Puis créer un Pod utilisant ce namespace

```
apiVersion: v1
kind: Pod
metadata:
  name: podinnamespace
  namespace: test
spec:
  containers:
  - name: chaton
    image: tomcat
```

On peut créer ce pod

```
kubectl create -f pod-in-namespace.yml
kubectl get pods --namespace=test
kubectl get pods --all-namespaces
```

