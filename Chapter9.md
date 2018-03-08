# Chapter 8: DaemonSets

- replicates a single pod on all or a subset of nodes in the cluster
- land an agent or daemon on every node for, say, logging, monitoring, etc.

ReplicaSet and DaemonSet both create pods which are expected to be long-running
services & try to match the observed and desired state in the cluster.

Use a ReplicaSet when:
- the app is completely decoupled from the node
- multiple copies of the pod can be run on one node
- no scheduling restrictions need to be in place to ensure replicas don't run on the same node

Use a DaemonSet when:
- one pod per node or subset of nodes in the cluster



## DaemonSet Scheduler
- By default the a copy is created on every pod
- The nodes can be limited using node selectors which matches to a set of labels
- DaemonSets determine which node a pod will run on when creating it using the `nodeName` field
- Hence, the k8s scheduler ignores the pods created by DaemonSets

The DaemonSet controller:
- creates a pod on each node that doesn't have one
- if a new node is added to the cluster, the DaemonSet controller adds a pod to it too
- it tries to reconcile the observed state and the desired state

## Creating DaemonSets
- the name should be unique across a namespace
- includes a pod spec   
- creates the pod on **every** node if a node selector isn't specified

## Limiting DaemonSets to Specific Nodes
1. Add labels to nodes
``` 
$ kubectl label nodes k0-default-pool-35609c18-z7tb ssd=true
node "k0-default-pool-35609c18-z7tb" labeled
```
2. Add the `nodeSelector` key in the pod spec to limit the number of nodes the pod will run on:

```
apiVersion: extensions/v1beta1
kind: "DaemonSet"
metadata:
  labels:
    app: nginx
    ssd: "true"
  name: nginx-fast-storage
spec:
  template:
    metadata:
      labels:
        app: nginx
        ssd: "true"
    spec:
      nodeSelector:
        ssd: "true"
      containers:
        - name: nginx
          image: nginx:1.10.0
```
- If the labels specified as  a `nodeSelector` is added to a new to existing node,
the pod will be created on that node. Similarly, if the label is removed from an
existing node, the pod will also be removed.

## Updating a DaemonSet
- For k8s < 1.6, the DaemonSet was updated and the pods were deleted manually
- For k8s >= 1.6, the DaemonSet has an equivalent to the `Deployment` object
which manages the rollout

### Updating a DaemonSet by Deleting Individual Pods
- manually delete the pods associated with a DaemonSet
- delete the entire DaemonSet and create a new one with the updated config. This 
approach causes downtime as all the pods associated with the DaemonSet are also deleted.

### Rolling Update of a DaemonSet
- for backwards compatibility, the default update strategy is the one described above
- to use a rolling update set the following in your yaml:
` spec.updateStrategy.type: RollingUpdate`
- any change to `spec.template` or sub-field will initiate a rolling update
- it's controlled by the following two params:
    - `spec.minReadySeconds`, which determines how long a Pod must be “ready” before the
    rolling update proceeds to upgrade subsequent Pods
    - `spec.updateStrategy.rollingUpdate.maxUnavailable`, which indicates how many Pods
    may be simultaneously updated by the rolling update.

A higher value for `spec.updateStrategy.rollingUpdate.maxUnavailable` increases the blast 
radius in case a failure happens but increase the time it takes for the rollout.

> Note: In a rolling update, the pods associated with the DaemonSet are upgraded gradually
> while some pods are still running the old configuration until all pods have the new configuration.

To check the status of the rollout, run:
`kubectl rollout`

### Deleting a DaemonSet
`kubectl delete -f daemonset.yml`

- Deleting a DaemonSet deletes all the pods associated with it
- in order to retain the pods and delete only the DaemonSet, use the flag: `--cascade = false`  








