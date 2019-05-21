## S'assurer que les applications travaillent bien

### Nettoyer l'environnement précédent

`kubectl delete pods --all`

### Créer un pod avec une sonde liveness

```
apiVersion: v1
kind: Pod
metadata:
  name: rightpod
spec:
  containers:
  - name: sonde
    image: treeptik/probe
    ports:
    - containerPort: 9876
    livenessProbe:
      initialDelaySeconds: 2
      periodSeconds: 5
      httpGet:
        path: /health
        port: 9876
```

Lancer la commande:
```
kubectl create -f probe.yml
pod "rightpod" created

kubectl describe pod rightpod
```

### Lancer un port qui bogue

Créer le fichier **nitro.yml**

```
apiVersion: v1
kind: Pod
metadata:
  name: nitropod
spec:
  containers:
  - name: nitro
    image: treeptik/probe
    ports:
    - containerPort: 9876
    env:
    - name: HEALTH_MIN
      value: "1000"
    - name: HEALTH_MAX
      value: "4000"
    livenessProbe:
      initialDelaySeconds: 2
      periodSeconds: 5
      httpGet:
        path: /health
        port: 9876
```

Lancer le pod : 

```
kubectl create -f nitro.yml

kubectl describe pod nitropod

kubectl get pods

...
...
Events:
  Type     Reason                 Age                From               Message
  ----     ------                 ----               ----               -------
  Normal   Scheduled              34s                default-scheduler  Successfully assigned nitropod to user1
  Normal   SuccessfulMountVolume  34s                kubelet, user1     MountVolume.SetUp succeeded for volume "default-token-zkwpx"
  Normal   Pulling                33s                kubelet, user1     pulling image "treeptik/probe"
  Normal   Pulled                 32s                kubelet, user1     Successfully pulled image "treeptik/probe"
  Normal   Created                32s                kubelet, user1     Created container
  Normal   Started                32s                kubelet, user1     Started container
  Warning  Unhealthy              19s (x3 over 29s)  kubelet, user1     Liveness probe failed: Get http://10.42.2.28:9876/health: net/http: request canceled (Client.Timeout exceeded while awaiting headers)

kubectl get pods
NAME       READY     STATUS    RESTARTS   AGE
nitropod   1/1       Running   1          57s
rightpod   1/1       Running   0          1m

```

### Are you ready ?

Créer le fichier ready.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: readypod
spec:
  containers:
  - name: ready
    image: treeptik/probe
    ports:
    - containerPort: 9876
    readinessProbe:
      initialDelaySeconds: 10
      httpGet:
        path: /health
        port: 9876
```

Observer son état via :

```
kubectl describe pod readypod
```

Et en particulier le bloc **Condition**

```
...
...
Conditions:
  Type           Status
  Initialized    True
  Ready          False
  PodScheduled   True
...
...
```

