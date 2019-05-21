# Service dns

## Créer un déploiement et le service associé

Créer le fichier **nginx-dns.yml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30001
    protocol: TCP
  selector:
    app: nginx
```

Exécuter la commande :

```
kubectl apply -f nginx-dns.yml
```

Puis lister les services
```
kubectl get svc                      

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.43.240.1    <none>        443/TCP        12d
nginxsvc     NodePort    10.43.245.59   <none>        80:30001/TCP   9s
```

## Inspecter le container

Créer un fichier **shell-demo.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: shell-demo
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
```

Puis déployer le :

```
kubectl create -f shell-demo.yaml

kubectl exec -it shell-demo -- /bin/bash
```

Dans le nouveau prompt terminal
```
apt-get update
apt-get install busybox -y
```

```
busybox nslookup nginxsvc
```

Qui doit retourner l'équivalent de :
```
Server:    10.43.240.10
Address 1: 10.43.240.10 kube-dns.kube-system.svc.cluster.local

Name:      nginxsvc
Address 1: 10.43.245.59 nginxsvc.default.svc.cluster.local
```

### Depuis un autre namespace

Créer un namespace **foo**

```
kubectl create namespace foo
```

Puis relancer un interpréteur dans le namespace **foo**

```
kubectl create -f shell-demo.yaml -n foo
kubectl exec -it shell-demo -n foo -- /bin/bash
```

Dans le nouveau prompt terminal
```
apt-get update
apt-get install busybox -y
```

Désormais vous ne pouvez plus faire

```
busybox nslookup nginxsv
Server:    10.43.240.10
Address 1: 10.43.240.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'nginxsv'
root@shell-demo:/# busybox nslookup nginxsvc
Server:    10.43.240.10
Address 1: 10.43.240.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'nginxsvc'
```

Vous devez préciser le namespace **default**

```
nslookup nginxsvc.default
Server:    10.43.240.10
Address 1: 10.43.240.10 kube-dns.kube-system.svc.cluster.local

Name:      nginxsvc.default
Address 1: 10.43.245.59 nginxsvc.default.svc.cluster.local
```
