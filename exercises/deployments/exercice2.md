## Utiliser les Deployments

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

## Pourquoi utiliser les Deployments ?

Un ReplicaSet ne peut fournir de service de rolling-update que les ReplicationController savent faire.
Pour faire une action de rolling-update ou de rollout sur un ReplicaSet, il est nécessaire d'utiliser un Deployment de façon déclarative.

## Créer son premier Deployment

Copier le contenu dans le fichier **tomcat-dep.yml**

```
kubectl apply -f tomcat-dep.yml --record

apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: tomcat-deploy
spec:
  replicas: 2
  template:
    metadata:
      labels:
        env: dev
        owner: nicolas
    spec:
      containers:
      - name: sise
        image: tomcat:8.0.52-jre8
        ports:
        - containerPort: 8080
        env:
        - name: URL_WS
          value: "https://foo.01.ws.com"
```

On peut voir toutes les ressources créées :

```
kubectl get all
NAME                                 READY     STATUS    RESTARTS   AGE
pod/tomcat-deploy-5777588498-lxm4q   1/1       Running   0          6s
pod/tomcat-deploy-5777588498-qrbvs   1/1       Running   0          6s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   14s

NAME                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tomcat-deploy   2         2         2            2           6s

NAME                                       DESIRED   CURRENT   READY     AGE
replicaset.apps/tomcat-deploy-5777588498   2         2         2         6s
```

Afficher la valeur de la variable d'environnement :

```
kubectl exec -it tomcat-deploy-5777588498-qrbvs env | grep URL_WS
URL_WS=https://foo.01.ws.com
```

Editer le fichier "tomcat-dep.yml" pour y changer de la variable d'environnement URL_WS

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: tomcat-deploy
spec:
  replicas: 2
  template:
    metadata:
      labels:
        env: dev
        owner: nicolas
    spec:
      containers:
      - name: sise
        image: tomcat:8.0.52-jre8
        ports:
        - containerPort: 8080
        env:
        - name: URL_WS
          value: "https://foo.02.ws.com"
```

Tentez de déployer la modification avec :

```
kubectl create -f tomcat-dep.yml
Error from server (AlreadyExists): error when creating "tomcat-dep.yml": deployments.apps "tomcat-deploy" already exists
```

Pour déployer la modification, vous ne devez pas utiliser la commande **create** mais bien **apply**.

```
kubectl apply -f tomcat-dep.yml --record
deployment.apps "tomcat-deploy" configured

kubectl get pods
NAME                             READY     STATUS              RESTARTS   AGE
tomcat-deploy-7d54877fdd-blrg2   1/1       Running             0          2s
tomcat-deploy-7d54877fdd-snnws   0/1       ContainerCreating   0          1s
tomcat-deploy-849cd86d55-8zmmb   1/1       Terminating         0          2m
tomcat-deploy-849cd86d55-jdm8h   1/1       Running             0          2m

kubectl get pods
NAME                             READY     STATUS        RESTARTS   AGE
tomcat-deploy-7d54877fdd-blrg2   1/1       Running       0          3s
tomcat-deploy-7d54877fdd-snnws   1/1       Running       0          2s
tomcat-deploy-849cd86d55-8zmmb   0/1       Terminating   0          2m
tomcat-deploy-849cd86d55-jdm8h   1/1       Terminating   0          2m

kubectl get pods
NAME                             READY     STATUS    RESTARTS   AGE
tomcat-deploy-7d54877fdd-blrg2   1/1       Running   0          1m
tomcat-deploy-7d54877fdd-snnws   1/1       Running   0          1m
```

Afficher la valeur de la variable d'environnement :

```
kubectl exec -it tomcat-deploy-7d54877fdd-blrg2 env | grep URL_WS
URL_WS=https://foo.02.ws.com
```

Pour lister les versions disponibles :

```
kubectl rollout history deployment

REVISION  CHANGE-CAUSE
1         kubectl apply --filename=tomcat-dep.yml --record=true
```

Editer le fichier "tomcat-dep.yml" pour y changer de la variable d'environnement URL_WS
en la mettant à **URL_WS=https://foo.03.ws.com**

Cette fois-ci, vous ne devez pas mettre l'option **--record**

```
kubectl apply -f tomcat-dep.yml
deployment.apps "tomcat-deploy" configured

REVISION  CHANGE-CAUSE
1         kubectl apply --filename=tomcat-dep.yml --record=true
2         <none>
```

Il est possible de rajouter l'information *a posteriori*

```
kubectl annotate deployment.extensions/tomcat-deploy kubernetes.io/change-cause=xxxxxxxxx

kubectl rollout history deployment
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=tomcat-dep.yml --record=true
2         xxxxxxxxx
```

### Faire un RollBack


Il est possible de revenir en arrière avec :

```
kubectl rollout undo deploy/tomcat-deploy --to-revision=1
deployment "tomcat-deploy" rolled back

kubectl get pods
NAME                             READY     STATUS    RESTARTS   AGE
tomcat-deploy-85b96964c8-ts64h   1/1       Running   0          1m
tomcat-deploy-85b96964c8-wftnn   1/1       Running   0          1m

kubectl exec -it tomcat-deploy-85b96964c8-ts64h env | grep WS
URL_WS=https://foo.01.ws.com
```

### Suppression des ressources

Comme attendu, la suppression de la ressource **Deployment** implique la suppression de toutes ses dépendances.

```
kubectl delete deploy tomcat-deploy
deployment.extensions "tomcat-deploy" deleted

kubectl get pods
No resources found.
```
