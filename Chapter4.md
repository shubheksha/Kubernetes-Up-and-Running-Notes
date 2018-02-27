# Chapter 4. Common kubectl Commands
Basic commands that apply to all k8s objects:

# Namespaces
- used to group objects in the cluster
- each namespace acts like a folder holding a set of objects
- `kubectl` works with the `default` namespace by default
- `—namespace` can be passed to specify a different one

# Context
- can be used to change the default namespace, manage different clusters or different users for authenticating to them

- To change the namespace to `abc`, run:
`$ kubectl config set-context my-context --namespace=abc`
- This command just creates a context. To run it, use:
`$ kubectl config use-context my-context`
- This records the change in the kubectl config file while is usually located at `$HOME/.kube/config`. This file also stores information related to finding and authenticating to the cluster.

# Viewing Kubernetes API Objects
- Everything is represented by a RESTful resource, called objects.
- Each object has a unique HTTP endpoint scoped using namespaces. 
- kubectl uses these endpoints to fetch the objects

- Viewing resources in the current namespace:
`kubectl get <resource-name>` 

- Viewing a specific resource:
`kubectl get <resource-name> <object-name>`
Use `describe` instead of  `get`` for more detailed information about the object

- To get more info than normally displayed, use `-o wide` flag
- To get raw JSON or YAML objects from the API server, use `-o json`, `-o yaml`.


# Creating, Updating, and Destroying Kubernetes Objects
- k8s objects are represented using YAML or JSON files (each object has its own JSON/YAML file?)
- used to manipulate objects on the server
- To create an object described using `obj.yaml`:
`kubectl apply -f obj.yaml` 
K8s automatically infers the type, it doesn’t need to be specified.

# Labeling and Annotating Objects
- used to tag objects
- `kubectl label/annotate pods bar color=red`  adds the `color=red` label to a pod called bar.
- To remove a label:
 `kubectl label pods bar -color`

# Debugging Commands
To see logs for a container:
`kubectl logs <pod-name>`
- Use `-c` to choose a particular container 
- Use `-f` to stream logs to terminal
- `kubectl exec -it <pod-name> — bash` provides an interactive shell within the container.