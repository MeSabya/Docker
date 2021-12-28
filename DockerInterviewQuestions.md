# Docker Interview Questions 

#### Containers Vs Virtual Machines — Horizontal Comparison?


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

