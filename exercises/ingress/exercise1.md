# ingress

## Overview

At the end of this module, you will :
* Learn to manage the external access of internal resources
* Learn to manage ingress controller
* Learn to secure the cluster access

## Prerequisites

Create the directory **data/ingress** in your home folder to manage the YAML file needed in this module.

In order for the ingress resource to work, the cluster must have an ingress controller running :

* Contour
* F5 Networks
* HAproxy
* Istio
* Kong
* Nginx
* Traefik

The create command can create a Ingress object based on a yaml file definition.

Check that nginx-ingress-controller and default-http-backend is running

```
kubectl get pods -n ingress-nginx

NAME                                        READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-76c86d76c4-knc2t   1/1     Running   0          2m1s
```

Else deploy a Nginx Ingress Controller following :

https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md#verify-installation

Quickly :
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml

# NodePort Usage
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
```

To check if the ingress controller pods have started, run the following command:

```
kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx --watch
```

## Exercise 1

First, deploy two static website in two different deployments.
Then, expose each one on the port 80.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webserver1
  template:
    metadata:
      labels:
        app: webserver1
    spec:
      containers:
      - name: static
        image: dockersamples/static-site
        env:
        - name: AUTHOR
          value: Nicolas
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webserver2
  template:
    metadata:
      labels:
        app: webserver2
    spec:
      containers:
      - name: static
        image: dockersamples/static-site
        env:
        - name: AUTHOR
          value: Nausicaa
        ports:
        - containerPort: 80
```

Expose each on of the Deployment on port 80.

```
apiVersion: v1
kind: Service
metadata:
  name: webserver1
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: webserver1
---
apiVersion: v1
kind: Service
metadata:
  name: webserver2
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: webserver2
```

Create an Ingress resource to expose an Nginx pod Service's on port 80.


```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: myfirstingress
spec:
  rules:
  - host: training.treeptik.io
    http:
      paths:
      - backend:
          serviceName: webserver1
          servicePort: 80
        path: /path1
      - backend:
          serviceName: webserver2
          servicePort: 80
        path: /path2
```

Once the Pods are up and running, you should be able to connect to this two urls :
* http://training.treeptik.io : NODEPORT /path1
* http://training.treeptik.io : NODEPORT /path2

## Get command

### Resume

The get command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like tr, sort, uniq.

The default output display some useful information about each services :
* Name : the name of the newly created resource
* Hosts : the host to apply the newly created resource
* Address : the address exposed by the newly created resource
* Ports : the ports exposed by the resource
* Age : the age since his creation

### Exercise

List the current Ingress resources created.

```
kubectl get ingress
```

## Describe command

### Resume

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The describe command display a lot of configuration information about the Ingress (labels, annotations, events, backend associated, IP address associated, etc.) and the rules available for each ingress (hosts, path, backend associated).

This command is really useful to introspect and debug an object deployed in a cluster.

### Exercise

Describe one of the existing Ingress in the default namespace.

```
kubectl describe ingress myfirstingress
```

## Explain command

### Resume

Kubernetes come with a lot of documentation about his objects and the available options in each one. Those information can be fin easily in command line or in the official Kubernetes documentation.

The explain command allows to directly ask the API resource via the command line tools to display information about each Kubernetes objects and their architecture

### Exercise

```
kubectl explain ingresses.spec
```

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete command

### Resume

The delete command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Be careful on the deletion of a Ingress object. This will interupt existing communication based on this resource.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along
with the rest of the resource.

### Exercise

```
# Delete Ingress
kubectl delete ingress myfirstingress

# Delete Deployments
kubectl delete deployment webserver1 webserver2

# Delete Services
kubectl delete service webserver1 webserver2
```

## Module Exercise

Switch to Digital Ocean and clone the repository

https://github.com/dockersamples/example-voting-app

Once the Pods are up and running, you should be able to connect to this two urls :
* http://vote.treeptik.io
* http://result.treeptik.io
