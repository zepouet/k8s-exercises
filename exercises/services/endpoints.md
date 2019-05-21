# Endpoints

## Objectifs

* Apprendre à lister les EndPoints courants
* Voir la relation avec les noeuds et les pods
* En créer de nouveaux

## Lister les courants

Exécuter la commande:

```
kubectl get endpoints

NAME            ENDPOINTS                           AGE
ingress-nginx   10.233.77.130:80,10.233.87.198:80   16m
kubernetes      209.97.180.71:6443                  3h35m
```

## Créer de nouveaux endpoints

Créer le fichier **nginx-ingress.yml**

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
  name: ingress-nginx
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

Créer les ressources:
```
kubectl apply -f nginx-ingress.yml
```

Exécuter la commande:

```
kubectl get endpoints

NAME            ENDPOINTS                           AGE
ingress-nginx   10.233.77.130:80,10.233.87.198:80   16m
kubernetes      209.97.180.71:6443                  3h35m
```

Vous devez avoir deux endpoints correspondant aux deux pods.

```
kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP              NODE                            NOMINATED NODE
nginx-deployment-68775fc68c-f9htj   1/1     Running   0          19m   10.233.77.130   form-k8s-user1-node-2   <none>
nginx-deployment-68775fc68c-xj7bx   1/1     Running   0          19m   10.233.87.198   form-k8s-user1-node-1   <none>
```

Pour accèder aux endpoints, vous pouvez faire directement un **curl** depuis votre machine *master**.

```
curl 10.233.77.130

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Créer un nouvel objet EndPoint

Créer le fichier suivant en reportant bien l'IP d'un des

```
apiVersion: "v1"
kind: "Service"
metadata:
  name: "external-web"
spec:
  ports:
    -
      name: "fake-apache"
      protocol: "TCP"
      port: 80
      targetPort: 80
---
apiVersion: "v1"
kind: "Endpoints"
metadata:
  name: "external-web"
subsets:
  -
    addresses:
      -
        ip: "10.233.77.130" #The IP Address of the external web server
    ports:
      -
        port: 80
        name: "fake-apache"
```        

Créer les ressources:

```
kubectl apply -f endpoint.yml
```

Puis observer le nouvel endpoint créé:

```
kubectl get endpoints

NAME            ENDPOINTS                           AGE
external-web    10.233.77.130:80                    60s
ingress-nginx   10.233.77.130:80,10.233.87.198:80   24m
kubernetes      209.97.180.71:6443                  3h43m
```

Créer désormais un container éphémère pour accéder à cet endpoint via son nom DNS:

```
kubectl create -f https://k8s.io/examples/application/shell-demo.yaml

kubectl exec -it shell-demo -- /bin/bash

apt-get update
apt-get install -y curl
curl external-web
```
