# Chapter 5: Pods
* represent a unit of work in k8s
* single atomic unit grouping multiple containers and volumes
* unit of scheduling on k8s on a single machine, smallest deployable artifact
* each container has its own cgroup, but share namespaces
	* share the network namespace, each pod has an IP and same network namespace
	* same hostname
	* communicate using native IPC
* Pods on the same machine are isolated from each other as if they were of separate servers

# Pod Manifest
* text-file representation of the k8s API object
* API server accepts and processes manifests and then stores them in persistent storage (etd)

* scheduler uses the API to find unscheduled pods, places them on nodes based on resources in the manifest
* pods from the same app distributed on different machines for reliability
* **do not move** from one node to another, must be destroyed and resheduled

## Creating a Pod
* Create: `kubectl run --image=path-to-image`

* Status: `kubectl get pods`
Pending -> Running
Running implies that the pod and its containers have been created.

## Creating a Pod Manifest
* YAML/JSON
* key-value pairs
	* `metadata` - describes the pods and its labels
	* `spec` - describes volumes & list of containers

# Running Pods
- use `kubectl apply -f pod.yaml`
- submitted to the API server, the scheduler puts it on a healthy node
- monitored by the kubelet daemon process

## Listing Pods
- `kubectl get pods` : brief one-line description about each pod
- `kubectl describe pods pod_name` : detailed description about the pod
These commands work for any k8s object.

## Deleting Pods
- By name: `kubectl delete pods/kuard`
- By file used to create it: `kubectl delete -f pod.yaml`

Pods terminate gracefully to complete active requests. Their graceful period is 30 seconds. It transitions into the `Terminating` state and stop accepting requests. Data within the pod is gone once it’s deleted.

# Accessing Your Pod
## Using Port Forwarding







