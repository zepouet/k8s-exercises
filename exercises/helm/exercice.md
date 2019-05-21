## Cycle de vie d'un CHART

### Installer Helm

La procédure se trouve ici : https://helm.sh/docs/using_helm/#from-script

```
helm init
kubectl get po --all-namespaces
```

Créer le ServiceAccount pour tiller

```
kubectl create serviceaccount tiller --namespace kube-system
```

Puis le RoleBinding
```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: tiller
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
```

Le déployer
```
kubectl create -f tiller-clusterrolebinding.yaml
````

Le mettre à jour et attendre quelques secondes.
```
helm init --service-account tiller --upgrade
```

### Génération de fichiers de ressources K8S

Avant de créer son premier chart, il est nécessaire d'avoir des fichiers ressources pour K8S.
Une solution élégante est de les générer avec l'outil kubectl.

```
mkdir ~/my-first-chart
cd ~/my-first-chart

mkdir resources
kubectl create deployment example --image=nginx:1.13.5-alpine --dry-run -o yaml > resources/deployment.yaml
kubectl apply -f resources/deployment.yaml
kubectl expose deployment example \
              --port=80 \
              --type=NodePort \
              -o yaml > resources/service.yaml
kubectl apply -f resources/service.yaml              
```

Il est possible d'accéder au déploiement via le NodePort exposé.
```
kubectl get svc,po
```

Il est nécessaire de nettoyer l'environnement avant de continuer.

```
$ kubectl delete -f resources
deployment "example" deleted
service "example" deleted
```

## Création de son premier Chart

```
$ helm create mychart
Creating mychart
$ sudo yum install tree
tree mychart
mychart
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml
2 directories, 7 files
```

Le fichier **Chart.yaml** est le manifest.
```
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
name: mychart
version: 0.1.0
```

Il est nécessaire de supprimer les fichiers non utilisés
* ingress.yaml
* NOTES.txt

```
$ cp -f resources/* mychart/templates/
$ rm mychart/templates/ingress.yaml
$ rm mychart/templates/NOTES.txt
```

```
helm install -n my-first-chart mychart
```

De même que précédemment, vous devez pouvoir accéder à la page NGINX.

```
kubectl get svc,po
```

Vous pouvez ensuite supprimer le déploiement
```
helm del --purge my-first-chart
release "my-first-chart" deleted
```

### Configuration des charts

Editez le fichier *mychart/values.yaml* pour qu'il soit
```
replicaCount: 1
image:
  repository: nginx
  tag: 1.13.5-alpine
  pullPolicy: IfNotPresent
  pullSecret:
service:
  type: NodePort  
```

Désormais les valeurs sont accessibles dans les fichiers de ressources à travers le moteur de templating de Golang.

**replicaCount** serait accessible selon **{{ .Values.replicaCount }}**

Mettez à jour le fichier **mychart/templates/deployment.yaml** avec
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  creationTimestamp: 2017-10-03T15:03:17Z
  generation: 1
  labels:
    run: "{{ .Release.Name }}"
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"     
  name: "{{ .Release.Name }}"
  namespace: default
  resourceVersion: "3030"
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/example
  uid: fd03ac95-a84b-11e7-a417-0800277e13b3
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      run: "{{ .Release.Name }}"
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: "{{ .Release.Name }}"
    spec:
      {{- if .Values.image.pullSecret }}    
            imagePullSecrets:
              - name: "{{ .Values.image.pullSecret }}"
      {{- end }}          
      containers:
      - image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        name: example
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}
```

De même éditez le fichier **mychart/templates/service.yaml**
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2017-10-03T15:03:30Z
  labels:
    run: "{{ .Release.Name }}"
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"  
  name: "{{ .Release.Name }}"
  namespace: default
  resourceVersion: "3066"
  selfLink: /api/v1/namespaces/default/services/example
  uid: 044d2b7e-a84c-11e7-a417-0800277e13b3
spec:
  clusterIP:
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 30254
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: "{{ .Release.Name }}"
  sessionAffinity: None
  type: "{{ .Values.service.type }}"
status:
  loadBalancer: {}
```

Désormais vous pouvez installer

```
helm install -n my-second-chart mychart
```    

De même que précédemment, vous devez pouvoir accéder à la page NGINX.

```
kubectl get svc,po
```

### Mettre à jour le déploiement

On va désormais remplacer NGINX par ApacheHTTPd à travers helm

```
helm upgrade --set image.repository=httpd --set image.tag=2.2.34-alpine my-second-chart mychart

Release "my-second-chart" has been upgraded. Happy Helming!
LAST DEPLOYED: Tue Nov  3 12:09:30 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME    CLUSTER-IP  EXTERNAL-IP  PORT(S)       AGE
second  10.0.0.160  <nodes>      80:30254/TCP  9m

==> v1beta1/Deployment
NAME    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
second  1        1        1           0          9m
```

### Packaging de l'application

Editez le fichier **mychart/templates/service.yaml** pour supprimer le NodePort hardcodé
```
  ...
  ports:  
  - port: 80
    protocol: TCP
    targetPort: 80
  ...
```

Puis générez un export tar.gz

```
helm package ./mychart
Successfully packaged chart and saved it to: /root/my-first-chart/mychart-0.1.0.tgz
```

Vous pouvez désormais réinstaller plusieurs fois l'application
```
helm install --name example3 mychart-0.1.0.tgz --set service.type=NodePort
helm install --name example4 mychart-0.1.0.tgz
```

Vous pouvez accéder à la dernière application par exemple

```
export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services example4)

export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

echo http://$NODE_IP:$NODE_PORT/
```
