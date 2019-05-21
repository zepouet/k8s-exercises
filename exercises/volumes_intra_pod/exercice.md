## Faire communiquer deux container d'un même pod

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Créer le volume

```
apiVersion: v1
kind: Pod
metadata:
  name: sharedvolume
spec:
  containers:
  - name: container1 
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: mysharedvolume
        mountPath: "/tmp/data1"
  - name: container2
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: mysharedvolume
        mountPath: "/tmp/data2"
  volumes:
  - name: mysharedvolume
    emptyDir: {}
```
 
### Vérifier que les deux volumes soient bien montées

```
kubectl exec sharedvolume -c container1 -i -t -- bash
[root@sharedvolume /]# mount | grep data
/dev/vda1 on /tmp/data1 type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
[root@sharedvolume /]# touch /tmp/data1/coucou
```

Puis 

```
kubectl exec sharedvolume -c container2 -i -t -- bash
[root@sharedvolume /]# mount | grep data
/dev/vda1 on /tmp/data2 type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
[root@sharedvolume /]# ls /tmp/data2
```


