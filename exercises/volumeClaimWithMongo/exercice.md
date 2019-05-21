## Travail en équipe même avec les HDD/SSD

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Creer un Persistent Volume

Fichier pv.yml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: "pv"
spec:
  storageClassName: manual
  capacity:
    storage: "1Gi"
  accessModes:
    - "ReadWriteOnce"
  hostPath:
    path: /data/pv
```

Lancer les commandes: 
```
kubectl create -f pv.yml
kubectl get pv
```

### Créer un Persistent Volume Claim

Fichier pvc.yml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: requetevolume
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```


```
kubectl create -f pvc.yml
kubectl get pvc
```

### Créer le container Mongo

Créer le fichier mongo.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: mongo
spec:
  containers:
    - name: mongo
      image: mongo:3.6
      volumeMounts:
      - mountPath: /data/db
        name: data-db
  volumes:
    - name: data-db
      persistentVolumeClaim:
        claimName: requetevolume
```

Lancer et lister les fichiers locaux
```
kubectl create -f mongo.yml
```

En se connectant sur le slave où se trouve le container mongo
```
slave# ls /data/pv 
```

