## Utiliser les Services pour exposer les pods

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

## Application Voting APP

Application exemple fournie par Docker dont le FrontEnd nécessite une base de données REDIS.
Commençons par la déployer en premier:

## Service ClusterIP

Créer le fichier **redis-deployment.yaml**

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis:alpine
        name: redis
        volumeMounts:
        - mountPath: /data
          name: redis-data
      volumes:
      - name: redis-data
        emptyDir: {}
```

Créer le fichier **redis-service.yaml**

```
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  type: ClusterIP
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
```

Déployer Redis et rendez le accessible

```
kubectl create -f redis-deployment.yaml

kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/redis-659469b86b-fczsr   1/1       Running   0          5s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   22m

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1         1         1            1           5s

NAME                               DESIRED   CURRENT   READY     AGE
replicaset.apps/redis-659469b86b   1         1         1         5s

```

Puis déployer le service **redis**
```
kubectl create -f redis-service.yaml

kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/redis-659469b86b-fczsr   1/1       Running   0          40s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.233.0.1      <none>        443/TCP    22m
service/redis        ClusterIP   10.233.41.239   <none>        6379/TCP   1s

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1         1         1            1           40s

NAME                               DESIRED   CURRENT   READY     AGE
replicaset.apps/redis-659469b86b   1         1         1         40s
```

Désormais **redis** est accessible sur le ClusterIp pour la future application **vote**.

## Service NodePort

Nous allons commencer par l'exposition la plus simple.

Commencer par créer le fichier **vote-deployment.yaml**

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: vote
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
      - image: dockersamples/examplevotingapp_vote:before
        name: vote
```

Puis déployer le avec la commande :

```
kubectl create -f vote-deployment.yaml
```

Désormais nous allons essayer de l'exposer sur l'extérieur. La solution la plus simple est d'utiliser un NodePort pour exposer sur chaque noeud slave un port d'écoute.

Commencer par créer le fichier **vote-service.yaml**

```
apiVersion: v1
kind: Service
metadata:
  name: vote
spec:
  type: NodePort
  ports:
  - name: "vote-service"
    port: 5000
    targetPort: 80
    nodePort: 31000
  selector:
    app: vote
```

Votre application doit être accessible sur chaque IP de vos noeuds WORKER:31000

### Scaler le service Vote

Vous pouvez soit éditer le fichier **vote-deployment.yaml** et le redéployer ou bien l'éditer directement en ligne:

Passer le nombre de **replicas** à 10 par exemple.

```
kubectl edit deployment vote
deployment.extensions "vote" edited

kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
redis-659469b86b-fczsr   1/1       Running   0          8m
vote-54f5f76b95-55rnn    1/1       Running   0          6m
vote-54f5f76b95-5flbg    1/1       Running   0          2m
vote-54f5f76b95-96ql6    1/1       Running   0          2m
vote-54f5f76b95-cqwnl    1/1       Running   0          2m
vote-54f5f76b95-d6mq4    1/1       Running   0          2m
vote-54f5f76b95-d84bw    1/1       Running   0          2m
vote-54f5f76b95-nwdjx    1/1       Running   0          2m
vote-54f5f76b95-q6r4f    1/1       Running   0          2m
vote-54f5f76b95-r4sp4    1/1       Running   0          2m
vote-54f5f76b95-zpnhm    1/1       Running   0          2m
```

Vous pouvez voir sur la page web que les ID des containers changent désormais.










