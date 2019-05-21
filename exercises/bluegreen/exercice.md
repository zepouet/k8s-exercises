## Blue Green Deployment

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Créer le déploiment Bleu

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: teapot-blue
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: demobg
        deployment: blue
    spec:
      containers:
      - name: teapot
        image: treeptik/nginx:blue
        ports:
        - containerPort: 80
        readinessProbe:
           tcpSocket:
              port: 80
           initialDelaySeconds: 2
           periodSeconds: 1 
```

### Créer le déploiment Vert

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: teapot-green
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: demobg
        deployment: green
    spec:
      containers:
      - name: teapot
        image: treeptik/nginx:green
        ports:
        - containerPort: 80
        readinessProbe:
           tcpSocket:
              port: 80
           initialDelaySeconds: 2
           periodSeconds: 1
```

### Créer le service

```
apiVersion: v1
kind: Service
metadata:
  name: demobg
spec:
  type: NodePort
  selector:
    app: demobg
    deployment: ${COLOR}
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30010
      protocol: TCP
```

### Changer de couleur

Plutôt que hardcoder la couleur:

```
COLOR=green envsubst < service.yml | kubectl apply -f -
```

### Pour aller plus loin

- Factoriser également les deux fichiers déploiements
- Monter le nombre de réplicas à 10

