# Utilisation des variables d'environment

## Créer un pod Redis

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

## Créer un Service exposant le pod sur un ClusterIP

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

## Afficher les Variables

```
kubectl get pods
...
..
.

kubectl exec -it redis-aaaaaaa-xxxx -- env | grep REDIS
```

Vous remarquez que les variables ne sont pas renseignées.

## Supprimer et recréer le pod

```
kubectl delete -f dep.yml
kubectl create -f dep.yml
```

## Afficher les Variables

```
kubectl get pods
...
..
.

kubectl exec -it redis-aaaaaaa-xxxx -- env | grep REDIS
```

Les variables sont bien renseignées cette fois-ci.
