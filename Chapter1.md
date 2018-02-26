# Chapter 1: Introduction
Kubernetes is an orchestrator for deploying containerised applications.
(Explain what’s an orchestrator)
It provides the software required to build and deploy “scalable, reliable” distributed systems.

Services today are delivered over the network via APIs. The distributed system comprising different machines implementing various parts of the API is responsible for serving these APIs. These machines communicate and coordinate via the network. These systems must be:
1. Reliable: the system as a whole must not fail if a part of it fails
2. Available: should be working even during maintenance, rollouts
3. Scalable: keep up with the increasing load without requiring radical redesign

## Benefits:
1. **Velocity**: 
	- refers to the speed with which software is developed, deployed and shipped.
	- isn’t measured in terms of raw speed or number of features shipped
	- more to do with how your system improves while being highly available
Concepts that enable this in k8s:
	1. **Immutability**:  there has been a shift from mutable to immutable infrastructure with k8s.
	
	In mutable infra,  changes are applied as an incremental updates to an		existing system. Ex: upgrading via apt-get. The state of the system is a 		not is composition of all the upgrades that happen over time.
	
	In immutable infra, a new image is built which replaces the older			image completely in one operation. This make it easier to keep track of	the history of the system. We’ve a record of the differences between		various versions. The old image is still around and can be used for a 		rollback.

	2. **Declarative Configuration**: everything in k8s is a declarative configuration object depicting the desired state of the system. 
	By contrast, in a system with imperative configuration, there is no 			notion of a desired state. The state is defined by execution of a series 		of instructions. It defines action, not state.

	Example: if you want three replicas for a system, the declarative			approach would be: replicas = 3 where the imperative approach 			would be run A, run B, run C.

	Declarative state makes rollback really easy as we can just revert to the 	previous declarative state of the system. In imperative systems, there is 	rarely away to trace our steps back to the previous state.

	3. **Self-healing systems**: K8s not only initialises your system, but it continuously takes actions to match the current state to the desired state. It’s not a one-time thing, it’s an ongoing process.

	This reduces the burden on operators and spending on maintenance   	and improves reliability of the system.

2. **Scaling Your Service and Your Teams**
K8s favours declarative architectures to help scale your services and teams.
	1. **Decoupling**:
	In a decoupled architectures, each component is isolated from others
	by defined APIs and service load balancers.
	Load balancers ensure that when your traffic increases, you can 			increase capacity without affecting 	other layers  of your service.
	Having clear, defined and decoupled APIs allows using a micro 			services architecture helping teams scale.

	2. **Easy Scaling for Applications and Clusters**:
	