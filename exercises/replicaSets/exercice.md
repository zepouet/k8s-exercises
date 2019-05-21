## Utiliser les ReplicaSets

### Nettoyer l'environnement précédent

`kubectl delete pods --all`

## Comment multiplier les pods

Un ReplicaSet est un superviseur pour les pods qui doivent absolument fonctionner. Il lancera autant de pods que spécifié.

Si un pod survient au niveau d'un pod, il sera automatiquement relancé.

Copier dans un fichier **rs.yaml**


```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        env:
        - name: GET_HOSTS_FROM
          value: dns
        ports:
        - containerPort: 80
```

Lancer le RS via `kubectl create -f rs.yaml`

Désormais vous pouvez afficher tous les objets :

```
kubectl get all

NAME                 READY     STATUS    RESTARTS   AGE
pod/frontend-ggqjn   1/1       Running   0          18s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   16s

NAME                       DESIRED   CURRENT   READY     AGE
replicaset.apps/frontend   1         1         1         18s
```

### Supprimer le pod

Pour vous prouver que quelqu'un gère bien votre infrastructure, veuillez supprimer le pod directement

```
kubectl delete pods --all
```

Vous pouvez voir que le pod précédent est en train d'être supprimé et qu'un nouveau apparaît

```
kubectl get all

NAME                 READY     STATUS              RESTARTS   AGE
pod/frontend-s9lrl   0/1       ContainerCreating   0          1s
pod/frontend-vf9cs   1/1       Terminating         0          12s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   5m

NAME                       DESIRED   CURRENT   READY     AGE
replicaset.apps/frontend   1         1         0         5m
```

Puis finalement :

```
kubectl get all

NAME                 READY     STATUS    RESTARTS   AGE
pod/frontend-s9lrl   1/1       Running   0          46s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   6m

NAME                       DESIRED   CURRENT   READY     AGE
replicaset.apps/frontend   1         1         1         6m
```

### Multiplier les pods

Désormais on va demander au superviseur d'en lancer en plusieurs instances :

```
kubectl scale --replicas=3 replicaset.apps/frontend

kubectl get pod,rs
NAME                 READY     STATUS    RESTARTS   AGE
pod/frontend-6xdbh   1/1       Running   0          1m
pod/frontend-qcd65   1/1       Running   0          1m
pod/frontend-s9lrl   1/1       Running   0          6m

NAME                             DESIRED   CURRENT   READY     AGE
replicaset.extensions/frontend   3         3         3         11m
```

Comme précédemment, la suppression d'un pod provoque sa regénération :

```
kubectl delete pod/frontend-qcd65
pod "frontend-qcd65" deleted

kubectl get pod,rs
NAME                 READY     STATUS    RESTARTS   AGE
pod/frontend-6xdbh   1/1       Running   0          2m
pod/frontend-dwmwd   1/1       Running   0          6s
pod/frontend-s9lrl   1/1       Running   0          7m

NAME                             DESIRED   CURRENT   READY     AGE
replicaset.extensions/frontend   3         3         3         12m
```

### Supprimer le ReplicaSet

Comme attendu, la suppression du ReplicaSet implique la suppression des pods.

```
kubectl delete rs/frontend
replicaset.extensions "frontend" deleted

kubectl get pod,rs
NAME                 READY     STATUS        RESTARTS   AGE
pod/frontend-6xdbh   0/1       Terminating   0          4m
pod/frontend-dwmwd   0/1       Terminating   0          1m
pod/frontend-s9lrl   0/1       Terminating   0          8m

kubectl get pod,rs
No resources found.
```











