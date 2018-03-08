# Chapter 8: ReplicaSets
We need multiple replicas of containers running at a time because:
- Redundancy: fault-tolerant system
- Scale: more requests can be served
- Sharding: computation can be handled in a parallel manner

Multiple copies of pods can be created manually, but it’s a tedious process. We need a way in which a replicated set of pods can be managed and defined as a single entity. This what the **ReplicaSet** does: ensures the right types and number of pods are running correctly.
Pods managed by a ReplicaSet are automatically rescheduled during a node failure or network partition.

When defining ReplicaSet, we need:
- specification of the pods we want to create
- desired number of replicas
- a way of finding the pods controlled by the ReplicaSet

## Reconciliation Loops
This is the main concept behind how ReplicaSets work and it is fundamental to the design and implementation of Kubernetes.
Here we deal with two states:
- desired state is the state you want - the desired number of replicas
- the current state is the observed staten the present moment - the number of pods presently running

- The reconciliation loop runs constantly to check if there is a mismatch between the current and the desired state of the system.
- If it finds a mismatch, then it takes the required actions to match the current state with what’s desired.
- For example, in the case of replicating pods, it’ll decide whether to scale up or down the number of pods based on what’s specified in the pod’s YAML. If there are 2 pods and we require 3, it’ll create a new pod.

### Benefits:
- goal-driven
- self-healing
- can be expressed in few lines of code
-
## Relating Pods and ReplicaSets
- pods and ReplicaSets are loosely coupled
- ReplicaSets doesn’t own the pods they create
- use label queries to identify which set of pods they’re managing

This decoupling supports:

### Adopting Existing Containers:
If we want to replicate an existing pod and if the ReplicaSet owned the pod, then we’d have to delete the pod and re-create it through a ReplicaSet.  This would lead to downtime. But since they’re decoupled, the ReplicaSet can simply “adopt” the existing pod.

### Quarantining Containers
If a pod is misbehaving and we want to investigate what’s wrong, we can isolate it by changing its labels instead of killing it. This will dissociate it from the ReplicaSet and/or service and consequently the controller will notice that a pod is missing and create a new one. The bad pod is available to developers for debugging.

## Designing with ReplicaSets
- represent a single, scalable microservice in your app
- every pod created is homogeneous and are inter-changable
- designed for stateless services

## ReplicaSet spec
- must’ve a unique name (within a namespace or cluster-wide?)
- a `spec` section that contains:
	- number of replicas
	- pod template
	
### Pod Templates
- created using the pod template in the `spec` section
- the ReplicaSet controller creates & submits the pod manifest to the API server directly

### Labels
- ReplicaSets use labels to filter pods it’s tracking and responsible for
- When a ReplicaSet is created, it queries the API server for the list of pods, filters it by labels. It adds/deletes pods based on what’s returned.

## Scaling ReplicaSets
### Imperative Scaling
`kubectl scale replica-set-name —replicas=4`

Don’t forget to update any text-file configs you’ve to match the value set imperatively.

### Declarative Scaling
Change the `replicas` field in the config file via version control and then apply it to the cluster.

### Autoscaling a ReplicaSet
K8s has a mechanism called *horizontal pod autoscaling* (HPA). It is called that because k8s differentiates between:
- horizontal scaling: create additional replicas of a pod
- vertical scaling: adding resources (CPU, memory) to a particular pod

HPA uses a pod known as *heapster* in your cluster to work correctly. This pod keeps track of metrics and provides an API for consuming those metrics when it makes scaling decisions.

> Note: There is no relationship between HPA and ReplicaSet. But it’s a bad idea to use both imperative/declarative management and autoscaling together. It can lead to unexpected behaviour.  

## Deleting ReplicaSets
Deleting a ReplicaSet deletes all the pods it created & managed as well. TO delete only the ReplicaSet object & not the pods, use `—cascasde=false`.