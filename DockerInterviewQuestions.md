# Docker Interview Questions 

#### Containers Vs Virtual Machines — Horizontal Comparison?
![image](https://user-images.githubusercontent.com/33947539/147723164-e1cd2013-ed74-403d-9b91-6f23053df3d9.png)   ![image](https://user-images.githubusercontent.com/33947539/147723176-b5656380-9b53-482e-b8c2-7bad1fa265e5.png)

**Hence a Virtual Machine will have:**

1: Higher utilization of resources as it runs multiple operating systems. Where as Docker actually utilizes the underlying kernel and not the operating system.
   So even if the machine on which docker installed is of ubuntu, docker can still install containers based on fedora, etc as the underlying kernel is same ie -linux.
   
2: Higher disk space, in GBs as compared to MBs for docker

3: Boot up slower, in minutes as compared to seconds for docker


#### Which instruction sets the base image for the subsequent builds in the Dokcerfile?
FROM

#### What does the RUN instruction do in the Dockerfile?

The RUN instruction will execute any commands in a new layer on top of the current image and commit the results.

#### How many forms that CMD instruction has?

```yaml
CMD ["executable","param1","param2"] (exec form, this is the preferred form)

CMD ["param1","param2"] (as default parameters to ENTRYPOINT)

CMD command param1 param2 (shell form)
```
#### What are Volumes in Docker?

Volumes can be more safely shared among multiple containers. We can prepopulate the volume with the run command and we can even backup, restore and remove volumes.

#### Why do we need Docker Volumes?

Data is no longer persisted and difficult to access if container stops.
As we can see writable layer is tightly coupled with host filesystem and difficult to move the data.
We have an extra layer of abstraction with a storage driver which reduces the performance.

#### What is the purpose of the CMD instruction in the Dockerfile?

The main purpose of a CMD is to provide defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.

#### What is the purpose of the LABEL instruction in the Dockerfile?

It adds metadata to the Image.

#### What is the difference between ADD and COPY instructions?

ADD [--chown=<user>:<group>] <src>... <dest>The ADD instruction copies new files, directories or remote file URLs from <src> and adds them to the filesystem of the image at the path <dest>.

COPY [--chown=<user>:<group>] <src>... <dest>The COPY instruction copies new files or directories from <src> and adds them to the filesystem of the container at the path <dest>.

#### What is the WORKDIR instruction in the Dockerfile?

The WORKDIR instruction sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile.
  
#### What is the ARG instruction in the Dockerfile?
  
ARG <name>[=<default value>]The ARG instruction defines a variable that users can pass at build-time to the builder with the docker build command using the --build-arg <varname>=<value> flag.
  
#### What is the command to remove unused images?
```DockerFile  
docker image prune
```
  
#### Where do you configure any credential helpers or credentials for the registry to prevent passing every time you log in?
```Dockerfile  
/etc/docker/daemon.json
```
  
How to pull an image from the repository?
```Dockerfile  
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
//pulling from docker hub by default
docker pull debian
// pulling from other repositories
docker pull myregistry.local:5000/testing/test-image
```
#### How do you create a Docker Image for Python FLASK API?

```DockerFile
FROM python:3.7-alpine
RUN mkdir /app
WORKDIR /app
ADD requirements.txt /app
ADD main.py /app
RUN pip3 install -r requirements.txt
CMD ["gunicorn", "-w 4", "-b", "0.0.0.0:8000", "main:app"]
```
  
##### In Detail:
  
✔ First, you want to build the image base on the other Docker image with name python:3.7-alpine. This image is the lightweight one than python:3.7 which may have 1GB size of   
  image in your local machine or server.
  
✔ Then you create a directory under the root dir within the container. You will have /app directory once your container run.
  
✔ Using WORKDIR will let you change your current directory inside container when the image is still being built. The default directory is root directory then you move to /app  
  to proceed your building further.
  
 ✔ copy requirements.txt and main.py to /app directory within your container.
    
 ✔ run Python dependencies which is required by our web application using RUN pip3 install -r requirements.txt
 
#### What is difference between CMD and RUN?
  
✔ RUN is an image build step, the state of the container after a RUN command will be committed to the container image. A Dockerfile can have many RUN steps that layer on top of   one another to build the image.

✔ CMD is the command the container executes by default when you launch the built image. A Dockerfile will only use the final CMD defined. The CMD can be overridden when 
  starting a container with docker run $image $other_command.
  
  >RUN - command triggers while we build the docker image.
  >CMD - command triggers while we launch the created docker image.
 
#### What are Multi-stage Builds?
👉  With multi-stage builds, we can use multiple FROM statements to build each phase. Every FROM statement starts with the new base and leave behind everything which you don’t need from the previous FROM statement.
  
👉 Use a single Dockerfile file with distinct sections. An image can be named simply by adding AS at the end of the FROM instruction. Consider the following simplified Dockerfile file:

```Dockerfile
FROM python:3.7-slim AS compile-image
RUN apt-get update
RUN apt-get install -y --no-install-recommends build-essential gcc

COPY requirements.txt .
RUN pip install --user -r requirements.txt

COPY setup.py .
COPY myapp/ .
RUN pip install --user .

FROM python:3.7-slim AS build-image
COPY --from=compile-image /root/.local /root/.local

# Make sure scripts in .local are usable:
ENV PATH=/root/.local/bin:$PATH
CMD ['myapp']
```
👉 References: https://pythonspeed.com/articles/multi-stage-docker-python/
  
#### Explain the container architecture?
*Docker uses a client-server architecture. The docker client talks to the Docker daemon, which used to building, running, and distributing the Docker containers. The Docker client and daemon communicate using a REST API, over UNIX sockets, or a network interface.*

  ![image](https://user-images.githubusercontent.com/33947539/147566492-efb635a9-b70c-4f55-9c4e-7c3d3ef32318.png)

**There are five major components in the Docker architecture:**
  
✔ Docker Daemon listens to Docker API requests and manages Docker objects such as images, containers, networks and volumes.

✔ Docker Clients: With the help of Docker Clients, users can interact with Docker. Docker client provides a command-line interface (CLI) that allows users to run, and stop  
  application commands to a Docker daemon.

✔ Docker Host provides a complete environment to execute and run applications. It comprises of the Docker daemon, Images, Containers, Networks, and Storage.

✔ Docker Registry stores Docker images. Docker Hub is a public registry that anyone can use, and Docker is configured to use images on Docker Hub by default. You can run your   
  own registry on it.
  
✔ Docker Images are read-only templates that you build from a set of instructions written in Dockerfile. Images define both what you want your packaged application and its  
   dependencies to look like what processes to run when it’s launched.
 
#### What is Open Container Initiatives ?
👉 The Open Container Initiative (OCI) is a Linux Foundation project to design open standards for containers.
  
👉 OCI currently contains two specifications: the Runtime Specification (runtime-spec) and the Image Specification (image-spec).
  
👉 **OCI runtime spec defines how to run the OCI image bundle as a container.**
  
👉 OCI image spec defines how to create an OCI Image, which includes an image manifest, a filesystem (layer) serialization, and an image configuration.
  
#### What are the Container Runtimes?

So in a nutshell the container runtimes does some or all of the below tasks:
  
    ✔Container image management
  
    ✔Container lifecycle management
  
    ✔Container creation
    
    ✔Container resource management
  
###### Various Container Runtimes:
1. containerd, 

2. lxc

3. runc: runc is a CLI tool for spawning and running containers according to the OCI specification. It was developed by Docker Inc and donated to OCI as the first OCI runtime-spec compliant reference implementation.

4. cri-o: CRI-O is an implementation of the Kubernetes CRI (Container Runtime Interface) to enable using OCI (Open Container Initiative) compatible runtimes. It is a lightweight alternative to using Docker as the runtime for Kubernetes.

5. rkt

  ![image](https://user-images.githubusercontent.com/33947539/147568123-59b3d27c-be67-4397-b74a-4e8ee9aa4271.png)

>Docker is no longer a monolith and under the hood its using the containerd and runc runtimes to manage containers.
containerd enables us to have container ecosystem without docker, for example cri-o in Kubernetes world can have the container runtime without docker.  

#### Finally Explain the Docker Engine?

When Docker was first released, it has one monolithic binary which includes Docker client, Docker API, container runtime, an image builds, etc. But, it was later modularized into separate components according to OCI(open container Initiative). OCI created two container-related specifications
       ✔ Image Spec
       ✔ Container runtime spec

All the image-specific and container-specific binaries were moved out of the docker daemon and implemented as separate modules.

*Docker client takes the commands and sends these commands to daemon through API. Docker daemon is responsible for implementing those commands with the help of containerd and runc libs.*
  
![image](https://user-images.githubusercontent.com/33947539/147628028-9119d755-053d-48cd-af17-3ca364fa114e.png)

**Docker Client**: 
This is mostly CLI where you send docker commands to docker daemon. If you install Docker on any OS like Windows(cmd), mac(terminal), or Linux, This is where you actually interact with docker.

**Docker daemon**: 
This is the server that exposes Restful API to accept the docker commands from the Docker client. This is responsible for executing those commands, an image builds, authentication, security, networking, etc.

**Containerd**: This sits between daemon and runc at the OCI layer and this is responsible for the container lifecycle operations. It started as a lightweight container management But, it has more functionality other than container execution.

**runc**: It creates containers. containerd uses a new instance of runc every time we create a container. runc process exits as soon as it creates a container so that we don’t have to create as many runc instances as containers.

**Shim**: Shim is nothing but a library that enables daemonless containers. The reason why it is daemonless is runc process exits as soon as it creates a container. Shim is responsible for the container from there. It reports the container’s exit status back to the daemon.
  
#### What are containers?
👉 A running instance of an image is called a container. We can start the container with any image we built or pulled from the docker hub with the following command 

![image](https://user-images.githubusercontent.com/33947539/147628932-392d9a84-7758-4a96-b9b0-6802162e002a.png)

>docker container run
  
For instance, Let’s start a container from the node image. If the image is cached locally it will it from the local, if not it will pull it from the official docker repo.

>docker container run -it --name my-node node:latest /bin/bash
 
we don’t have a local node image so, it is pulled from the docker hub and process we are running /bin/bash as soon as the container starts. We can interact with the bash shell immediately to check the versions of the node like below. we can exit out with the exit command.

![image](https://user-images.githubusercontent.com/33947539/147628469-1568f0bd-38bf-440f-b65f-873df115879e.png)

We can start the container in a detached mode as well with -d flag. When we start the container like this, it just returns the id.

![image](https://user-images.githubusercontent.com/33947539/147628578-de948b1b-b640-4470-b485-a333df97c9b3.png)
  
#### Explain the Docker run command?

![image](https://user-images.githubusercontent.com/33947539/147628712-cb3a1e92-b165-4a67-a6cf-82ada7dced80.png)

![image](https://user-images.githubusercontent.com/33947539/147629133-4ba10ebd-8225-454d-b9fc-681f97c1fd84.png)

#### How do you interact with the running containers?

👉 We can interact with the running containers in two ways **exec and attach**
  
we can directly interact with the container when we start a container by providing the process we want to run at the end.
for instance, if we are running node, we can provide bash at the end to interact with it
>docker container run -it --name my-node node:latest /bin/bash

#### Explain the difference between attach and exec commands?
**attach**: 
This is used to interact with the container with the same process that container is running. Let’s run the following commands.

```yaml
// run the container in detached mode
docker container run -dit --name mynode1 node bash
// make sure container is up
docker container ls
// attach the running container
docker attach mynode
// exit out of the running process
exit
// check whether conatiner is running or not
docker ps
```
  
*if we exit with the attach command, it exits the present running process of the container. so, the container stops.*

![image](https://user-images.githubusercontent.com/33947539/147629403-10fe7f9b-7144-4155-8874-bca39564a1b1.png)

**exec**: 
This command is used to interact with the container with a separate process. So that if we exit out of the process, the container is still running in another process.

```yaml  
// run the container in detached mode
docker container run -dit --name mynode1 node bash
// make sure container is up
docker container ls
// using exec, takes two arguments
docker exec -it mynode bash
// exit out of the running process
exit
// check whether conatiner is running or not
docker ps
```
![image](https://user-images.githubusercontent.com/33947539/147629576-3535b70c-ddf0-4527-aa58-d7079aa54409.png)

#### What is the difference between the commands container stop and container rm?

The difference is that docker container stop sends SIGTERM to container process and docker container rm sends SIGKILL to container process. SIGKILL wait for 10 sec before it ends container’s life.  

#### What are some of the best practices using Docker containers?

```yaml  
Always stop containers before removing. Graceful shutdown is always the best practice
Use volumes for persistent data. Make sure removing volumes if not necessary
use one application per container. For instance, don’t use nodejs app and MongoDB in one container.
Use docker cache to our advantage while building images using dockerfiles
always build the smallest image possible so that container starts quickly and will be fewer disk usages.
make use of docker multi-stage dockerfile to build small and secure images.
when it comes to usage of public image, should be chosen carefully as we don’t have complete control over it.
tags are mutable so we should use these tags properly. Use digest since it is immutable.
Always build the image only with the necessary tools. Always reduce the usage of tools, it will have less surface for attacks.
whenever we are sharing data between containers with volumes, make sure only one container is writing into the volume to avoid concurrency.  
```
#### How do you see the logs of running containers?
   
```Dockerfile
We can access logs of the running container with the following command
// container logs
docker container logs mynode
// last 10 lines
docker container logs tail -10 mynode
// follow the logs
docker container logs --tail 10--follow mynode
```

#### Which command should you use to define volumes in the Dockerfile?
   
```Dockerfile   
VOLUME /dockerexample
```

#### What is the difference between CMD and ENTRYPOINT in a Dockerfile?

👉 *The ENTRYPOINT specifies a command that will always be executed when the container starts. The CMD specifies arguments that will be fed to the ENTRYPOINT.*
   
For example, if your Dockerfile is:
   
```Dockerfile
FROM debian:wheezy
ENTRYPOINT ["/bin/ping"]
CMD ["localhost"]
```

Running the image without any argument will ping the localhost:
   
```Doc
$ docker run -it test
PING localhost (127.0.0.1): 48 data bytes
56 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.096 ms
56 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.088 ms
56 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.088 ms
^C--- localhost ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.088/0.091/0.096/0.000 ms
Now, running the image with an argument will ping the argument:

$ docker run -it test google.com
PING google.com (173.194.45.70): 48 data bytes
56 bytes from 173.194.45.70: icmp_seq=0 ttl=55 time=32.583 ms
56 bytes from 173.194.45.70: icmp_seq=2 ttl=55 time=30.327 ms
56 bytes from 173.194.45.70: icmp_seq=4 ttl=55 time=46.379 ms
^C--- google.com ping statistics ---
5 packets transmitted, 3 packets received, 40% packet loss
round-trip min/avg/max/stddev = 30.327/36.430/46.379/7.095 ms
```
   
Anything that appears after the image name in the docker run command is passed to the container and treated as CMD arguments.

#### Docker Logging: Where Are Container Logs Stored

You see, by default, Docker containers emit logs to the stdout and stderr output streams. Containers are stateless, and the logs are stored on the Docker host in JSON files by default.

👉 You find these JSON log files in the **/var/lib/docker/containers/** directory on a Linux Docker host. The <container_id> here is the id of the running container.
/var/lib/docker/containers/<container_id>/<container_id>-json.log

![image](https://user-images.githubusercontent.com/33947539/147730223-479a5c82-a41f-4fe2-8683-2fe99ef45eb4.png)

#### which all can be used to provide environment variables to a container in docker-compose.yml file?
   
   1. - env_file : This can be used to supply a .env file containing all variables.
   
   2. - environment: This option is used to write down variables directly in docker-compose file.
#### 


  
