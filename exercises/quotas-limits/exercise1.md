# Quotas and limits overview

### Creation

The most basic resource metrics for a pod are CPU and memory. Kubernetes provides requests and limits to pre-allocate resources and limit resource usage, respectively.
Limits restrict the resource usage of a pod as follows:
* If its memory usage exceeds the memory limit, this pod is out of memory (OOM) killed.
* If its CPU usage exceeds the CPU limit, this pod is not killed, but its CPU usage is restricted to the limit.

### Resource quota

Kubernetes provides the ResourceQuota object to set constraints on the number of Kubernetes objects by type and the amount of resources (CPU and memory) in a namespace.
* One or more ResourceQuota objects can be created in a namespace.
* If the ResourceQuota object is configured in a namespace, requests and limits must be set during deployment; otherwise, pod creation is rejected.
* To avoid this problem, the LimitRange object can be used to set the default requests and limits for each pod.

## Prerequisites

Create the directory data/quotas in your home folder to manage the YAML file needed in this module.

```
mkdir ~/data/quotas
```

## Resource quota

Create the file **resourcequota.yaml**

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: myfirstresourcequota
  namespace: default
spec:
  hard:
    requests.cpu: "3"
    requests.memory: 1Gi
    limits.cpu: "5"
    limits.memory: 2Gi
    pods: "5"
```    

Create the object based on the previous yaml file definition.

```
kubectl create -f ~/data/quotas/resourcequota.yaml
```

## Limit range

The LimitRange object is used to set the default resource requests and limits as well as minimum and maximum constraints for each pod in a namespace.

```
apiVersion: v1
kind: LimitRange
metadata:
  name: myfirstlimitrange
  namespace: default
spec:
  limits:
  - default:  # default limit
      memory: 512Mi
      cpu: 2
    defaultRequest:  # default request
      memory: 256Mi
      cpu: 0.5
    max:  # max limit
      memory: 800Mi
      cpu: 3
    min:  # min request
      memory: 100Mi
      cpu: 0.3
    maxLimitRequestRatio:  # max value for limit / request
      memory: 2
      cpu: 2
    type: Container # limit type, support: Container / Pod / PersistentVolumeClaim
```

Create the object based on the previous yaml file definition.

```
kubectl create -f ~/data/limits/limitrange.yaml
```

## GET Command

The **get command** list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like tr, sort, uniq.

### Resource quota

The default output display some useful information about each services :
* Name : the name of the newly created object
* Age : the age since his creation

```
kubectl get resourcequota
```

### Limit range

The default output display some useful information about each services :
* Name : the name of the newly created object
* Age : the age of the object since his creation

```
kubectl get limitrange
```

## Describe Command

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The describe command display a lot of configuration information about the Resource Quotas and Limits (labels, annotations, etc.) and the amount of resource (default, memory, cpu, ...).

This command is really useful to introspect and debug an object deployed in a cluster.

Describe one of the existing resource quota in the default namespace.

### Resource  quota

Describe one of the existing resource quota in the default namespace.

```
kubectl describe resourcequota myfirstresourcequota
```

### Limit range

Describe one of the existing limit range in the default namespace.

```
kubectl describe limitrange myfirstlimitrange
```

## Explain

Kubernetes come with a lot of documentation about his objects and the available options in each one. Those information can be fin easily in command line or in the official Kubernetes documentation.

The *explain command* allows to directly ask the API resource via the command line tools to display information about each Kubernetes objects and their architecture.

### Resource quota

Get the documentation of a specific field of a resource quota.

```
kubectl explain resourcequota
```

### Limit range

Get the documentation of a specific field of a limit range.

```
kubectl explain limitrange
```

## Delete Command

The **delete** command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Be careful on the deletion of a quota or a limit object, this can have effects in the availability of the services associated by increasing the resource consumption.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

```
# Delete the resource quota
kubectl delete resourcequota myfirstresourcequota

# Delete the limit range
kubectl delete limitrange myfirstlimitrange
```

# External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
* Kubernetes official documentation on [how to manage resource quotas in a namespace](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)
* Kubernetes official documentation on the [configuration of CPU limits](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)
* Kubernetes official documentation on the [configuation of memory limits](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)


# Exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to dynamically and automatically manage the limits of CPU and memory of Pods and manage resource quotas of the namespace.

**Solution** is given but try to do it without reading it.

## Resource quota

Create a new file **quotas.yml** limiting the amount of available resources in the **foo** namespace :
* 5 CPU unit can be requested
* 4Gi of memory can be requested
* The limit of CPU unit available is 7
* The limit of memory available is 6Gi

## Limit range

Create a new file **limitrange.yml**
Set the default limits of container resources has below :
* By default, a container can request 256Mi of memory and 0.5 of CPU unit
* The default limits is 512Mi of memory and 1 CPU unit
