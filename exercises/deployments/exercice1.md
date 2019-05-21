# Utiliser les Deployments

## Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

## Pourquoi utiliser les Deployments ?

Un ReplicaSet ne peut fournir de service de rolling-update que les ReplicationController savent faire.

Pour faire une action de rolling-update ou de rollout sur un ReplicaSet, il est nécessaire d'utiliser un Deployment de façon déclarative.

## Créer son premier Deployment

Copier le contenu dans le fichier **nginx0.yaml**

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  template:
    metadata:
      labels:
        service: http-server
    spec:
      containers:
      - name: nginx
        image: nginx:1.10.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

Créer le déploiement:

```
kubectl create -f nginx0.yaml
deployment "nginx" created
```

Afficher l'état du déploiement et du ReplicatSet:

```
kubectl get deployment
NAME                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx               3         3         3            3           8m

kubectl get rs
NAME                DESIRED   CURRENT   AGE
nginx-3322722759    3         3         8m

kubectl get pod -l "service in (http-server)"
NAME                     READY     STATUS    RESTARTS   AGE
nginx-3322722759-7vp34   1/1       Running   0          14m
nginx-3322722759-ip5w2   1/1       Running   0          14m
nginx-3322722759-q97b7   1/1       Running   0          14m
```

Afficher le détail du déploiement

```
kubectl describe deployment/nginx

Name:                   nginx
Namespace:              default
CreationTimestamp:      Mon, 18 Apr 2019 15:00:55 +0000
Labels:                 service=http-server
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               service=http-server
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  service=http-server
  Containers:
   nginx:
    Image:        nginx:1.10.2
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-859d69c8cd (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  6s    deployment-controller  Scaled up replica set nginx-859d69c8cd to 3
```

## Impératif par fichier

Editer le fichier **nginx0.yaml** et passer le nombre de réplicas à 5

```
kubectl replace -f nginx0.yaml
deployment.extensions/nginx replaced
```

Afficher les pods :

```
kubectl get pods
```

Que constatez vous ?

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-67b67f7678-nm42b   1/1     Running   0          21s
nginx-67b67f7678-tcsfz   1/1     Running   0          21s
nginx-67b67f7678-tsqjq   1/1     Running   0          3s
nginx-67b67f7678-w785k   1/1     Running   0          3s
nginx-67b67f7678-wsqw5   1/1     Running   0          21s
```

Réponse : De nouveaux pods ont été rajoutés.

Editer à nouveau le fichier **nginx0.yaml** pour changer la version de l'image en **1.10.2** en **1.11.5**

```
kubectl replace -f nginx0.yaml
kubectl get pod
```

Que constatez vous ?

```
NAME                     READY   STATUS              RESTARTS   AGE
nginx-67b67f7678-nm42b   1/1     Running             0          2m56s
nginx-67b67f7678-tcsfz   1/1     Running             0          2m56s
nginx-67b67f7678-tsqjq   1/1     Terminating         0          2m38s
nginx-67b67f7678-w785k   0/1     Terminating         0          2m38s
nginx-67b67f7678-wsqw5   1/1     Running             0          2m56s
nginx-79fc79777b-t6rjp   0/1     ContainerCreating   0          2s
nginx-79fc79777b-w452p   1/1     Running             0          2s
nginx-79fc79777b-xv8c4   0/1     ContainerCreating   0          0s
```

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-79fc79777b-6bnwq   1/1     Running   0          19s
nginx-79fc79777b-frp95   1/1     Running   0          21s
nginx-79fc79777b-t6rjp   1/1     Running   0          23s
nginx-79fc79777b-w452p   1/1     Running   0          23s
nginx-79fc79777b-xv8c4   1/1     Running   0          21s
```

Réponse : Tous les pods ont été remplacés dans le quasi laps de temps.

Créer le fichier **nginx1.yaml** avec le bloc spécifique **strategy**

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      service: http-server
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 10
  template:
    metadata:
      labels:
        service: http-server
    spec:
      containers:
      - name: nginx
        image: nginx:1.12
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

Explications:
* minReadySeconds: attente minimale entre la création de deux pods
* maxSurge: nombre de pods qui vont s'ajouter au nombre désiré (absolu ou pourcentage)
* maxUnavailable: nombre de pods qui peuvent être indisponible durant la mise à jour

Déployons la nouvelle version :
```
kubectl replace -f nginx.yaml

kubectl get pods -w
```

## Impératif en ligne de commande

### Choix 1 - Redéfinition du tag Image

Le format de la mise à jour est le suivant:
`kubectl set image deployment <deployment> <container>=<image> --record`

Dans notre cas, ce serait:
`kubectl set image deployment nginx nginx=nginx:1.11.5 --record`

### Choix 2 - Edition directe

```
kubectl edit deployment nginx --record
```

Modifier la valeur de l'image nginx par tomcat

```

### Rollout status

```
kubectl rollout status deployment nginx
```

### Pause Rolling Update

```
kubectl rollout pause deployment nginx
```

### Resume Rolling Update

```
kubectl rollout resume deployment nginx
```

## Rollback

Vous souhaitez revenir en arrière sur une des versions.

```
kubectl rollout history deployment nginx
deployments "nginx":
REVISION  CHANGE-CAUSE
1   kubectl apply -f nginx.yaml --record
2   kubectl set image deployment nginx nginx=nginx:1.11.5 --record
```

Allons restaurer la version 1:

```
# to previous revision
kubectl rollout undo deployment nginx

# example
kubectl rollout undo deployment nginx --to-revision=1
```

Pour spécifier la profondeur de l'historique, ajouter **revisionHistoryLimit: 10**

```
kubectl edit deployment nginx --record

...
spec:
  replicas: 10
  selector:
    matchLabels:
      service: http-server
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 5
  revisionHistoryLimit: 10
...
```

Vous obtiendrez alors:

```
kubectl rollout history deployment/nginx
deployments "nginx":
REVISION  CHANGE-CAUSE
2   kubectl set image deployment nginx nginx=nginx:1.11 --record
3   kubectl set image deployment nginx nginx=nginx:1.11.5 --record
4   kubectl set image deployment nginx nginx=nginx:1.10 --record
5   kubectl set image deployment nginx nginx=nginx:1.10.2 --record
```
