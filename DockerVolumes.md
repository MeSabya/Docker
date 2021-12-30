# Understanding Docker Volumes

*By default, all files created inside a container are stored on a writable container layer. This means that:*

✔ The data doesn’t persist when that container no longer exists, and it can be difficult to get the data out of the container if another process needs it.

✔ A container’s writable layer is tightly coupled to the host machine where the container is running. You can’t easily move the data somewhere else.

✔Writing into a container’s writable layer requires a storage driver to manage the filesystem. The storage driver provides a union filesystem, using the Linux kernel. This extra abstraction reduces performance as compared to using data volumes, which write directly to the host filesystem.

### But Why data is lost when a container deleted?

In order to understand what a Docker volume is, we first need to be clear about how the filesystem normally works in Docker. 

>Docker images are stored as series of read-only layers. 
>When we start a container, Docker takes the read-only image and adds a read-write layer on top. 
>If the running container modifies an existing file, the file is copied out of the underlying read-only layer and into the top-most read-write layer where the changes are applied. 
>The version in the read-write layer hides the underlying file, but does not destroy it -- it still exists in the underlying layer. When a Docker container is deleted, relaunching the image will start a fresh container without any of the changes made in the previously running container -- those changes are lost. 

👉 **Docker calls this combination of read-only layers with a read-write layer on top a Union File System.**

### Temporary Data

So Data can be kept temporarily inside a Docker container in two ways:

**Way 1**: 
By default, files created by an application inside a container are stored in the writable layer of the container as mentioned above.

**Way2**:
If you don’t need your data to persist beyond the life of the container, a **tmpfs mount is a temporary mount that uses the host’s memory**. 
A tmpfs mount has the benefit of faster read and write operations.

### Persistent Data

There are **two ways to persist data** beyond the life of the container. 

**Way 1: Bind Mount**:
One way is to bind mount a file system to the container. With a bind mount, processes outside Docker also can modify the data.
But Bind mounts are difficult to back up, migrate, or share with other Containers. 


### Need of Volume:
In order to be able to save (persist) data and also to share data between containers, Docker came up with the concept of volumes. Quite simply, volumes are directories (or files) that are outside of the default Union File System and exist as normal directories and files on the host filesystem.

Volumes are:

```Doc
persistent
free-floating filesystems, separate from any one container
sharable with other containers
efficient for input and output
able to be hosted on remote cloud providers
encryptable
nameable
able to have their content pre-populated by a container
handy for testing
```
### Data Persistent Problem in action:

```Dockerfile
// pull the nginx image
docker pull nginx
// run the container
docker run -it --name=webApp -d -p 80:80 nginx

// list the running containers
docker ps
// exec command
docker exec -it webApp bash

// cd to welcome page and edit it
cd /usr/share/nginx/html
echo "I changed this file while running the conatiner" > index.html
```

👉 *Let’s stop the container and start it again. we can still see the changes that we made. what if we stop this container and start another one and load the page. There is no way that we could access the file that we have changed in another container.*

### How Volumes can solve above issues?

👉 *Volumes are saved in the host filesystem (/var/lib/docker/volumes/) which is owned and maintained by docker.*
Any other nondocker process can’t access it. But, As depicted in the below other docker processes/containers can still access the data even container is stopped since it is isolated from the container file system.

![image](https://user-images.githubusercontent.com/33947539/147725853-5ca00eb1-738a-416e-8011-f7a615e5efc7.png)

#### Let’s see the same example with volumes 
```Dockerfile
docker run -d --name=webApp1 --mount source=new_vol,destination=/usr/share/nginx/html -p 80:80 nginx
```

Let’s go and change the index.html from the new_vol location by ssh into the docker.

![image](https://user-images.githubusercontent.com/33947539/147726454-8ae36f0c-0cef-4f8e-897c-b57ce0851ac1.png)

Let’s stop this container and start another one with the same command

```Dockerfile
docker stop webApp1
docker run -d --name=webApp2 --mount source=new_vol,destination=/usr/share/nginx/html -p 80:80 nginx
```
we can load the page again localhost:80 and still see the html file that we edited in the volume.
So, with the help of volumes, we can easily access the data even we stop the container and it’s very easy to access data and import the data to anywhere.



