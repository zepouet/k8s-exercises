# Stratégies de placement

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

## Node Selector : nodeSelector

Obtenez la liste des Nodes 
`kubectl get nodes`
```
NAME                           STATUS     ROLES          AGE       VERSION
demo1   					   	Ready      master         8d        v1.10.2
slave1     						Ready   ingress,node   	 8d        v1.10.2
slave2   						Ready   ingress,node      8d        v1.10.2
```

Comme toutes ressources il est possible de la labeliser. Ce ou ces labels serviront au kube-scheduler pour lancer un Pod porteur du label. 
Ajoutons le Label **"schedulePodName"="hello-pod"** au node slave1. 

### Labeliser le Node : Slave1 

```
kubectl label nodes slave1 schedulePodName=hello-pod`
node "slave1" labeled
```

Vérifier les labels : (regarder à la fin de la liste)
```
kubectl get nodes --show-labels
slave1     NotReady   ingress,node   8d        v1.10.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=slave1,node-role.kubernetes.io/ingress=true,node-role.kubernetes.io/node=true,schedulePodName=hello-pod
```

Il s'agit maintenant de configurer le Pod pour avoir le label correpondant dans le champ qui specifiera le **NodeSelector** :

### Creer le fichier de configuration 

Créer un fichier de configuration Pod "hello-nodeselector.yaml"

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-nodeselector
  labels:
    app: hello
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  containers:
    - name: hello
      image: "kelseyhightower/hello:1.0.0"
      ports:
      - name: http
        containerPort: 80
      - name: health
        containerPort: 81
      resources:
        limits:
          cpu: 25m
          memory: 50Mi
  nodeSelector:
  	schedulePodName: hello-pod
```

### Lancer le Pod 

```
kubectl create -f hello-nodeselector.yaml
pod "hello-nodeselector" created
```

Observer sur quel Node le Pod a été orchestré : 
```
kubectl describe pod hello-nodeselector
```

Ce qui amènera la sortie suivante : On vérifie bien que le Pod a été crée sur le Node **slave1**

```
Name:         hello-nodeselector
Namespace:    default
Node:         slave1/209.97.130.4
Start Time:   Wed, 20 Jun 2018 04:03:17 +0000
Labels:       app=hello
              environement=dev
              partition=training-k8s
              realease=stable
              tier=webserver
Annotations:  cni.projectcalico.org/podIP=10.42.1.4/32
Status:       Running
IP:           10.42.1.4
Containers:
  hello:
    Container ID:   docker://c5f3c21011c91f0cdfa38d3ffdbd1613e9f1b33cba15fa3fac3dd2a2ba40993c
    Image:          kelseyhightower/hello:1.0.0
    Image ID:       docker-pullable://kelseyhightower/hello@sha256:6d60ae5cf957ee1d87ac7e93bbe29e991eab18d18c29385bf9a5bf791a8d82e2
    Ports:          80/TCP, 81/TCP
    Host Ports:     0/TCP, 0/TCP
    State:          Running
      Started:      Wed, 20 Jun 2018 04:03:19 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     25m
      memory:  50Mi
    Requests:
      cpu:        25m
      memory:     50Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-bc2hd (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-bc2hd:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-bc2hd
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  schedulePodName=hello-pod
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              54s   default-scheduler  Successfully assigned hello-nodeselector to slave1
  Normal  SuccessfulMountVolume  53s   kubelet, slave1    MountVolume.SetUp succeeded for volume "default-token-bc2hd"
  Normal  Pulled                 53s   kubelet, slave1    Container image "kelseyhightower/hello:1.0.0" already present on machine
  Normal  Created                53s   kubelet, slave1    Created container
  Normal  Started                52s   kubelet, slave1    Started container
```


## Node Afinity : nodeAfinity 

Obtenez la liste des Nodes 
`kubectl get nodes`
```
NAME                           STATUS     ROLES          AGE       VERSION
demo1   	Ready     		 master         8d        v1.10.2
slave1     Ready   			ingress,node   	 8d        v1.10.2
slave2     Ready  		 ingress,node      8d        v1.10.2
```

### Labeliser les Nodes : slave1 et slave2

Ajoutons les labels suivants aux 2 Nodes : 
- slave1 : "AvailZone=az-North"
- slave2 : "AvailZone=az-South"

**Node 1**

```
kubectl label nodes slave1 AvailZone=az-North`
node "slave1" labeled
```

**Node 2**

```
kubectl label nodes slave2 AvailZone=az-South`
node "kevindp-form-k8s-user1-node-2" labeled
```


### Vérifier les labels : (regarder au début de chaque liste des labels)
```
kubectl get nodes --show-labels
NAME      STATUS    ROLES               AGE       VERSION   LABELS
demo1     Ready     controlplane,etcd   6h        v1.10.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=demo1,node-role.kubernetes.io/controlplane=true,node-role.kubernetes.io/etcd=true
slave1    Ready     worker              6h        v1.10.1   AvailZone=az-North,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=slave1,node-role.kubernetes.io/worker=true,schedulePodName=hello-pod
slave2    Ready     worker              6h        v1.10.1   AvailZone=az-South,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=slave2,node-role.kubernetes.io/worker=true
```

### Configuration des Pods à scheduler

Il s'agit maintenant de configurer 3 Pods
- Pod-North qui devra se lancer sur un Node avec le Label az-North - Affinité = Required 
- Pod-South qui devra se lancer sur un Node avec le Label az-South - Affinité = Required 
- Pod-Middle qui se lancera sur un Node Présentant soit le Label az-North soit le Label az-South - Affinité = Prefered ( le kube-scheduler choisit le Node en se basant sur un score calculé en fonction de plusieurs paramètres )

Créer les fichiers de configuration des Pods suivants 

### Pod "hello-pod-north"

Editer un nouveau fichier de configuration :  **pod-north.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-north
  labels:
    app: hello-pod-north
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: AvailZone
            operator: In
            values:
            - az-North
            - az-North-East 
  containers:
    - name: hello
      image: "kelseyhightower/hello:1.0.0"
      ports:
      - name: http
        containerPort: 80
      - name: health
        containerPort: 81
      resources:
        limits:
          cpu: 25m
          memory: 50Mi
```

### Pod "hello-pod-south"

Editer un nouveau fichier de configuration : **pod-south.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-south
  labels:
    app: hello-pod-south
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: AvailZone
            operator: In
            values:
            - az-South
            - az-South-East 
  containers:
    - name: hello
      image: "kelseyhightower/hello:1.0.0"
      ports:
      - name: http
        containerPort: 80
      - name: health
        containerPort: 81
      resources:
        limits:
          cpu: 25m
          memory: 50Mi
```

### Pod "hello-pod-middle"

Editer un nouveau fichier de configuration : **pod-middle.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-middle
  labels:
    app: hello
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    nodeAffinity: 
      preferredDuringSchedulingIgnoredDuringExecution: 
      - weight: 1 
        preference:
          matchExpressions:
          - key: AvailZone
            operator: In 
            values:
            - az-North 
            - az-South
  containers:
    - name: hello
      image: "kelseyhightower/hello:1.0.0"
      ports:
      - name: http
        containerPort: 80
      - name: health
        containerPort: 81
      resources:
        limits:
          cpu: 25m
          memory: 50Mi
```

### Lancer les Pods 

```
kubectl create -f pod-north.yaml 
pod "hello-pod-north" created

kubectl create -f pod-south.yaml
pod "hello-pod-south" created

kubectl create -f pod-middle.yaml
pod "hello-pod-middle" created
```

### Obtenir la description des Pods : 

Vérifier le placement des Pods 

```
kubectl describe pod  hello-pod-north
...
Node:         slave1/207.154.237.15
...

kubectl describe pod  hello-pod-south
...
Node:         slave2/207.154.237.16
...

kubectl describe pod  hello-pod-middle
...
Node:         slave2/207.154.237.16 // Ici le scheduler a choisit "Node2"
...
```

## Pod Afinity : podAfinity & podAntiAffinity

Pour commencer, vérifier que les 2 Pods de l'exercice précédent sont bien lancés : 
- Pod : "hello-pod-north" 
- Pod : "hello-pod-south" 

```
kubectl get pod hello-pod-north
NAME              READY     STATUS    RESTARTS   AGE
hello-pod-north   1/1       Running   0          5h
```

```
kubectl get pod hello-pod-south
NAME              READY     STATUS    RESTARTS   AGE
hello-pod-south   1/1       Running   0          5h
```

### Configuration des nouveaux Pods

Dans cet exerice nous allons configurer 2 nouveaux Pod qui se lanceront en fonction de leur affinité (ou non) avec les Pods précedents.
- Le Pod hello-pod-north-east aura une affinité avec le pod "hello-pod-north" et se lancera donc sur le même Node
- Le Pod hello-pod-south-east aura une "anti"-affinité avec les pods du nord "hello-pod-north" "hello-pod-north-east" et se lancera donc un autre Node - ici avec le mecanisme preferred et la pondération "weight"

Créer le fichier de configuration de ces 2 pods tels que : 


### Pod "hello-pod-north-east"

Editer un nouveau fichier de configuration : **pod-affinity-north.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-north-east
  labels:
    app: hello-pod-north-east
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    podAffinity: 
      requiredDuringSchedulingIgnoredDuringExecution: 
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In 
            values:
            - hello-pod-north
            - hello-pod-north-east
        topologyKey: kubernetes.io/hostname
  containers:
    - name: hello
      image: "kelseyhightower/hello:1.0.0"
      ports:
      - name: http
        containerPort: 80
      - name: health
        containerPort: 81
      resources:
        limits:
          cpu: 25m
          memory: 50Mi
```

### Pod "hello-pod-south-east"

Editer un nouveau fichier de configuration : **pod-anti-affinity-north.yaml** 

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-south-east
  labels:
    app: hello-pod-south-east
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    podAntiAffinity: 
      preferredDuringSchedulingIgnoredDuringExecution: 
      - weight: 100 
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app 
              operator: In 
              values:
              - hello-pod-north
              - hello-pod-north-east
          topologyKey: kubernetes.io/hostname
  containers:
    - name: hello
      image: "kelseyhightower/hello:1.0.0"
      ports:
      - name: http
        containerPort: 80
      - name: health
        containerPort: 81
      resources:
        limits:
          cpu: 25m
          memory: 50Mi
```

### Lancer les Pods 

```
kubectl create -f pod-affinity-north.yaml 
pod "hello-pod-north-east" created

kubectl create -f pod-anti-affinity-north.yaml
pod "hello-pod-south-east" created
```

### Obtenir la description des Pods : vérifier le placement des Pods 

```
kubectl describe pod hello-pod-north-east 
...
Node:         slave1/207.154.237.15
...

kubectl describe pod  hello-pod-south-east
...
Node:         slave2/207.154.237.16
...
```

**Nota** : il se peut que le scheduler est lancé le Pod sur demo1 (master) - la règle d'anti affinité étant respectée : lancer le Pod "hello-pod-south-east" là ou il n'y a pas les Pods "hello-pod-north" et "hello-pod-north-east" - Il sera donc nécéssaire d'affiner le placement 

**Perpective : question bonus**

A votre avis comment empêcher l'orchestrateur de lancer les pods sur le master demo1 ? .  





