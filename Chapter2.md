#Kubernetes/Books/K8s Up And Running/chapter 2#

# Chapter 2: Creating and Running Containers
The distributed systems that can be deployed by k8s are made up primarily of *application container images*.

Applications = language run time + libraries + source code
- Problems occur when you deploy an application to a machine that doesn’t available on the production OS.  Such a program, naturally, has trouble executing.
- Deployment often involves running scripts which have a set of instructions resulting in a lot of failure cases being missed.
- The situation becomes messier if multiple apps deployed to a single machine use different versions of the same shared library. 

Containers help solve the problems described above.

## Container images
> A *container image* is a binary package that encapsulates all the files   
> necessary to run an application inside of an OS container.  
It bundles the application along with its dependencies (runtime, libraries, config, env variables) into a single artefact under a root filesystem.
A container in a runtime instance of the image — what the image becomes in memory when executed.
We can obtain container images in two ways:
- a pre-existing image for a *container registry* (a repository of container images to which people can push images which can be pulled by others)
- build your own locally

Once an image is present on a computer, it can be run to get an application running inside an OS container.

### The Docker Image Format
- De facto standard for images 
- made up of a series of layers of filesystem where each layers adds/removes/modifies files from the previous layer. It’s called an **overlay** filesystem.
- - - -
**Note on Container Layering:**
Each layer inherits and modifies previous layer in an image
Example:
![](Chapter%202/C4B4C456-A459-4940-B018-95B429893C0E.png)

Containers B and C are *forked* from A and share nothing beside the base container’s files.
Each container image builds on the previous one. Each reference to the parent image is a pointer.
- - - -
Container images also have a configuration files which specify how to setup the env (networking, namespaces), the entry point for running the container, resource limits and privileges, etc. 

The Docker image format = container root file system + config file

Containers can be of two types:
1. System containers: like VMs, run a fool boot process, have system services like `ssh`, `cron`, etc.
2. Application containers: commonly run only a single app

## Building Application Images with Docker
### Dockerfiles
It a file that automates the building of container images.  Read more [here](https://docs.docker.com/engine/reference/builder/) .
The book uses a demo app, the source code is available on [GitHub](https://github.com/kubernetes-up-and-running/kuard).
To run the `kuar` (Kubernetes Up and Running) image, follow these steps:
1. Ensure you’ve Docker [installed and running](https://docs.docker.com/get-started/)
2. Download and clone the `kuar` repo
3. Run `make build`  to generate a binary
4. Create a file named Dockerfile (no extension) containing the following:
```
FROM alpine
COPY bin/1/amd64/kuard /kuard
ENTRYPOINT ["/kuard"]
```
5. Next, we build the image using the following command:
`$ docker build -t kuard-amd64:1 .`
The `-t` specifies the name of the [tag](https://docs.docker.com/engine/reference/commandline/build/#tag-an-image--t). [Tags](https://docs.docker.com/engine/reference/commandline/tag/) are a way to version Docker images.  the `.` tells docker to use the Dockerfile present in the current directory to build the image. 

`alpine` as mentioned in the Dockerfile is a minimal Linux distribution that is used as a base image. More info can be found [here](https://hub.docker.com/_/alpine/).

### Image Security
Don’t ever have passwords/secrets in any layer of your container image. Deleting it from one layer will not help if the preceding layers contain sensitive info.

### Optimising Image Sizes
- If a file is present in a preceding layer, it’ll be present in the image that uses that layer even though it’ll be inaccessible. 
Consider this example:
![](Chapter%202/60896C28-D934-4A80-95C6-D1CD85EAABAA.png)

It’s not that *BigFile* isn’t present in the final image. It’s just not accessible.  As it is still present in layer A, whenever the image is pushed or pulled the file goes over the network.

- Every time a layer is changed, every layer that comes after is also changed.  Changing any layer the image uses means that layer and all layers dependent on it need to be rebuilt, re-pushed and re-pulled for the image to work. 
As a rule of thumb, the layers should be ordered from least likely to change to most likely to change to avoid a lot of pushing and pulling.

## Storing Images in a Remote Registry
To  easily share container images to promote reuse and make them available on more machines, we use what’s called a registry. It’s a remote location where we can push our images and other people can download them from there. They’re of two types:
1) Public: anyone can download images
2) Private: authorisation is needed to download images 

The book uses the Google Container Registry whereas I used the Docker Hub. 
After creating an account on Docker Hub, run the following commands:
1. `$ docker login`
2. `$ docker tag kuard-amd64:1 $DockerHubUsername/kuard-amd64:1`
3. `$ docker push $DockerHubUsername/kuard-amd64:1`
Replace $DockerHubUsername with your username.

# The Docker Container Runtime
The default container run time used by Kubernetes is Docker.

## Running Containers with Docker
Run the following command to run the container that you pushed to Docker Hub in the previous step:
`$ docker run -d --name kuard -p 8080:8080 $DockerHubUsername/kuard-amd64:1`
Let’s try to unpack everything that command does one thing at a time:
- `-d` tells Docker to run the contain in [detached mode](https://docs.docker.com/engine/reference/run/#detached-vs-foreground). In this mode, the container doesn’t attach its output to your terminal and runs in the background. 
- `—name` gives a name to your *container*. Keep in mind it doesn’t alter the name of your *image* in anyway.
- `-p` enables port-forwarding. It maps port 8080 on your local machine to your container’s port 8080. Each container gets its own IP address and doesn’t have access to the host network (your machine in this case) by default. Hence, we’ve to explicitly expose the port.

To stop & remove the container, run:
`$ docker stop kuard`
`$ docker rm kuard`

Docker allows controlling how many resources your container can use (memory, swap space, CPU) etc., using various flags that can be passed to the `run` command.

# Cleanup
Images can be removed using the following command:
`$ docker rmi <tag-name/image-id>`

Docker IDs can be shortened as long as they remain unique.

**Note**:  Unless an image is explicitly deleted, it’ll not be removed from your system. Even if you build a new image with the same name, only the tag will be moved to the new image. The old image will not be replaced or deleted.

 