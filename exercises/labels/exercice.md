## Travail sur les labels

### Nettoyer l'environnement précédent

`kubectl delete pods --all`

### Création d'un nouveau tomcat

Créer un nouveau fichier **pod-tomcat-label.dev.yaml**

`kubectl create -f pod-tomcat-label.dev.yaml`

````
apiVersion: v1
kind: Pod
metadata:
  name: mytomcat
  labels:
    env: development
spec:
  containers:
  - name: rawtomcat
    image: tomcat:8.0.52-jre8
````

Afficher désormais les labels associés aux pods :

````
kubectl get pods mytomcat --show-labels
NAME       READY     STATUS    RESTARTS   AGE       LABELS
mytomcat   1/1       Running   0          35s       env=development
````

Ajouter un nouvel label au pod 

```
kubectl label pods mytomcat owner=nicolas

kubectl get pods mytomcat --show-labels
NAME       READY     STATUS    RESTARTS   AGE       LABELS
mytomcat   1/1       Running   0          2m        env=development,owner=nicolas

```

Afficher seulement les pods ayant un certain label :

```
kubectl get pods -l env=development
NAME       READY     STATUS    RESTARTS   AGE
mytomcat   1/1       Running   0          3m
```

## Utiliser les **Selector**

Lancer un second Tomcat avec nouveau propriétaire (owner=kevin) pour un environnement de production.

````
kubectl create -f https://raw.githubusercontent.com/Treeptik/training-k8s-resources/master/labels/tomcat-kevin.yaml

kubectl get pods --show-labels
NAME         READY     STATUS    RESTARTS   AGE       LABELS
mytomcat     1/1       Running   0          8m        env=development,owner=nicolas
realtomcat   1/1       Running   0          12s       env=production,owner=kevin
````

On peut désormais cibler les pods en fonction des labels avec l'option `--selector` ou `-l` en abrégé.

```
kubectl get pods --selector 'env in (production, development)'
kubectl get pods -l 'env in (production, development)'
```

Pour ne choisir que celui de production

```
kubectl get pods --selector 'env in (production)'
kubectl get pods -l 'env in (production)'
```

## Remarques

Les labels s'appliquent à tout type de ressources évidemment et pas seulement aux pods

