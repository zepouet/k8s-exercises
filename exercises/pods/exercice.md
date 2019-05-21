## Démarrer une image Tomcat simplement via la ligne de commande

Lancer une instance Tomcat sur le port standard.

```
kubectl run mytomcat --restart=Never --image=tomcat:8.0.52-jre8
```

or 

```
kubectl run mytomcat --generator=run-pod/v1 --image=tomcat:8.0.52-jre8
```

Vérifier le status du pod :

`kubectl get pods`

Obtenir l'IP du pod :

`kubectl describe pod mytomcat | grep IP:`

Vous pouvez remarquer qu'aucuun **Deployment** vient également d'être créé car les options `--restart=Never` ou `--generator=run-pod/v1` ont évité la création de ce dernier.

`kubectl get deployment`

Dans le cas où vous vous trouvez directement en ssh sur un noeud du cluster,  vous pouvez accéder en local à l'application via 
`curl 10.233.69.80:8080`

Vous pouvez également rentrer dans le container et y accéder en local :

```
kubectl exec mytomcat  -i -t -- bash
root@mytomcat:/usr/local/tomcat# curl localhost:8080
```

Vous pouvez lister les variables d'environnement :

`kubectl exec mytomcat printenv`

## Déployer un pod sur base d'un fichier 

Créer un fichier pour un nouveau pod.
Remarquez bien l'option **dry-run**.

```
kubectl run rawtomcat --generator=run-pod/v1 --image=tomcat:8.0.52-jre8 --dry-run -o yaml > pod-rawtomcat.yaml
```

Modifier le fichier pour avoir un second container

```
apiVersion: v1
kind: Pod
metadata:
  name: rawtomcat
spec:
  containers:
  - name: rawtomcat
    image: tomcat:8.0.52-jre8
  - name: shell
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
```

Vous remarquerez que l'on crée désormais deux containers au sein du même pod.

```
kubectl create -f pod-rawtomcat.yaml
kubectl exec rawtomcat -c shell -i -t -- bash
[root@mytomcat /]# ps -ef
[root@mytomcat /]# curl localhost:8080
```

On remarque bien que l'on est dans le container shell mais que l'on peut accéder en localhost au tomcat.






