## Créer des volumes et les monter dans des pods

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc,pv,pvc --all`

### Creer un Persistent Volume

Assurez-vous qu'il existe bien un dossier: "tmp" dans votre dossier personnel sinon créez le:

`mkdir ~/tmp`

Fichier pv.yml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
  labels:
    type: local
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  hostPath:
    path: "/home/$USER/tmp"
```

Lancer les commandes:
```
kubectl create -f pv.yml
kubectl get pv
```

### Créer un Persistent Volume Claim

Fichier pvc.yml
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: slow
```


```
kubectl create -f pvc.yml
kubectl get pvc
```

### Créer un Déploiment

Créer le fichier deployment.yml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
        - name: container
          image: alpine
          command: ["watch"]
          args: ["echo", "hello"]
          volumeMounts:
          - name: vol
            mountPath: "/data"
      volumes:
      - name: vol
        persistentVolumeClaim: 
          claimName: pvc
```

`kubectl create -f deployment.yml`

Listez les pods:

`kubectl get pods`

```
NAME                          READY     STATUS    RESTARTS   AGE
deployment-59ccfff984-4h4pd   1/1       Running   0          10m
deployment-59ccfff984-5zjg7   1/1       Running   0          10m
deployment-59ccfff984-nqqqh   1/1       Running   0          10m
deployment-59ccfff984-rb279   1/1       Running   0          10m
deployment-59ccfff984-vcnch   1/1       Running   0          10m
```

Lancez un shell dans un des pods précédemment créé:

`kubectl exec -it deployment-59ccfff984-4h4pd -- sh`

Créez un fichier dans le volume monter:

`touch /data/test`

Sortez du shell (exit ou Ctrl D)

Lancez un shell dans un autre pod:
`kubectl exec -it deployment-59ccfff984-5zjg7 -- sh`

Affichez le contenu du volume partagé entre les pods:

`ls /data/`

Que voyez vous ? Pourquoi ?

### Créer un deuxième Déploiement

Créer le fichier deployment2.yml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment2
spec:
  replicas: 5
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
        - name: container
          image: alpine
          command: ["watch"]
          args: ["echo", "hello"]
          volumeMounts:
          - name: vol
            mountPath: "/data"
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: pvc
```

`kubectl create -f deployment2.yml`

Listez les pods:

`kubectl get pods`

```
NAME                           READY     STATUS    RESTARTS   AGE
deployment-59ccfff984-4h4pd    1/1       Running   0          23m
deployment-59ccfff984-5zjg7    1/1       Running   0          23m
deployment-59ccfff984-nqqqh    1/1       Running   0          23m
deployment-59ccfff984-rb279    1/1       Running   0          23m
deployment-59ccfff984-vcnch    1/1       Running   0          23m
deployment2-59ccfff984-9st24   1/1       Running   0          1m
deployment2-59ccfff984-c7xtv   1/1       Running   0          1m
deployment2-59ccfff984-gw4l9   1/1       Running   0          1m
deployment2-59ccfff984-lxqpz   1/1       Running   0          1m
deployment2-59ccfff984-x6vvv   1/1       Running   0          1m

```
Lancez un shell dans un des pods du deuxième déploiement:

`kubectl exec -it deployment2-59ccfff984-9st24 -- sh`

Affichez le contenu du volume partagé entre les pods:

`ls /data/`

Que voyez vous ? Pourquoi ?

Créez un fichier dans le volume monter:

`touch /data/test2`

Sortez du shell (exit ou Ctrl D)

Lancez un shell dans un autre pod du deuxième déploiement:

`kubectl exec -it deployment2-59ccfff984-x6vvv -- sh`

Affichez le contenu du volume partagé entre les pods:

`ls /data/`

Que voyez vous ? Pourquoi ?

## Solution

L'attribut accessMode spécifié dans le Persistent Volume et le Persistent Volume Claim assure au pods du premier déploiement l'accés en lecture écriture au volume. Cependant "ReadWriteOnce" implique que le deuxième déploiement ne peut obtenir ces droits. Il donc impossible de lire ou écrire sur le volume depuis un pod du deuxième déploiement.
