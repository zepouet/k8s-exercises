# Autoscaling

## Prerequisites

Create the directory **data/autoscaling** in your home folder to manage the YAML file needed in this module.

Verify the **metric-server** is running or installed...

```
kubectl get apiservices |egrep metrics

v1beta1.metrics.k8s.io                  kube-system/metrics-server   True        26m
```

Too, you could to
```
kubectl top nodes
kubectl top pods
```

If not, try to do it

```
git clone https://github.com/kubernetes-incubator/metrics-server.git
cd metrics-server
```

Edit the file *metrics-server-deployment.yaml* and add the command

```
containers:
- name: metrics-server
  image: k8s.gcr.io/metrics-server-amd64:v0.3.3
  imagePullPolicy: Always
  volumeMounts:
  - name: tmp-dir
    mountPath: /tmp
  command:
      - /metrics-server
      - --kubelet-insecure-tls
      - --kubelet-preferred-address-types=InternalIP
```

Then

```
kubectl apply -f deploy/1.8+/
```

## External documentation

Those documentations can help you to go further in this topic :
* Kubernetes official documentation on [Horizontal Pod Auto Scaling (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* Kubernetes official documentation [walkthrough HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
* Kubernetes official documentation of [autoscale API](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#autoscale)

```
mkdir ~/autoscaling
```

## Exercises guided

### Exercise

Run a sample app based on a webserver to expose it on port 80.

```
# Run a sample app in the default namespace

kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --limits=cpu=300m --expose --port=80
```

Create an Horizontal Pod Autoscaler to automatically scale the Deployment if the CPU usage is above 50%.

```
# Create an Horizontal Pod Autoscaler based on the CPU usage

kubectl autoscale deployment php-apache --cpu-percent=50 --min=3 --max=10
```

## Get Command

The get command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like tr, sort, uniq.

The default output display some useful information about each services :

* Name : the name for the newly created object
* Reference : the object managed by the autoscaler, like Pod name, a Deployment name ...
* Targets : the metrics defined to autoscale the referenced resource
* Minpods : the lower limit for the number of pods that can be set by the autoscaler
* Maxpods : the upper limit for the number of pods that can be set by the autoscaler
* Replicas : the current replicas number
* Age : the age of the object from his creation

### Exercise 1

Get the current HorizontalPodAutoscaler resources in the default namespace.

```
kubectl get hpa

php-apache   Deployment/php-apache   0%/50%   3         10        3          16m
```

### Exercise 2

```
# Connect to the Pod
kubectl run -it load-generator --image=busybox /bin/sh

# Run a loop bash command in the container to stress the CPU
while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done

# Check the Horizontal Pod Autoscaler status
```

The autoscaling can take more than 2 minutes to run.
**Please be patient**. Do not close the window or cancel the operation.

### Exercise 3

**Stop** to stress the Pod previously created and check that the autoscaler come back to normal.

```
kubectl get hpa

NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    3         10        3          29m
```

## Describe Command

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The describe command display a lot of configuration information about the Horizontal Pod Autoscaler (labels, annotations, etc.) and the scale policy (selector, type, number of pods, ...).

This command is really useful to introspect and debug an object deployed in a cluster.

### Exercise 1

```
kubectl describe horizontalpodautoscaler php-apache
```

## Explain Command

Kubernetes come with a lot of documentation about his objects and the available options in each one. Those information can be fin easily in command line or in the official Kubernetes documentation.

The explain command allows to directly ask the API resource via the command line tools to display information about each Kubernetes objects and their architecture.

### Exercise

Get the documentation of a specific field of a resource.

```
kubectl explain hpa.spec
```

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete Command

The delete command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Be careful on the deletion of an autoscaling object, this can have effects in the availability of the services associated.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource

Delete the previous autoscaling group created in command line.

```
# Delete the HorizontalPodAutoscaler
kubectl delete hpa php-apache

# Delete the Pods
kubectl delete deployment php-apache load-generator

# Delete the Services
kubectl delete service php-apache
```

## Exercise advanced

Manage the HorizontalPodAutoscaler of the worker Pods to :
* Ensure that the worker has minimum one Pods
* Ensure that the worker has maximum five Pods
* Ensure that the Pods is autoscaled when the CPU is above 80%.

Give two responses : one command line and one file.
