# Docker-Compose Explanation

Docker-compose is a tool that combines and runs multiple containers of interrelated services with a single command. 
It is a tool to define all the application dependencies in one place and let the Docker take care of running all of those services in just one simple command docker-compose up.

```yaml
version: '3'
services:
  web:
    # Path to dockerfile.
    # '.' represents the current directory in which
    # docker-compose.yml is present.
    build: .

    # Mapping of container port to host
    
    ports:
      - "5000:5000"
    # Mount volume 
    volumes:
      - "/usercode/:/code"

    # Link database container to app container 
    # for reachability.
    links:
      - "database:backenddb"
    
  database:

    # image to fetch from docker hub
    image: mysql/mysql-server:5.7

    # Environment variables for startup script
    # container will use these variables
    # to start the container with these define variables. 
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
      - "MYSQL_USER=testuser"
      - "MYSQL_PASSWORD=admin123"
      - "MYSQL_DATABASE=backend"
    # Mount init.sql file to automatically run 
    # and create tables for us.
    # everything in docker-entrypoint-initdb.d folder
    # is executed as soon as container is up nd running.
    volumes:
      - "/usercode/db/init.sql:/docker-entrypoint-initdb.d/init.sql"
    
```
## Docker Compose file In Detail

**version ‘3’**: Like other software, docker-compose also started with version 1.0. At the time of writing this course, the current latest version of Compose file is 3.7.

**services**: The services section defines all the docker images required and need to be built for the application to work. 
              In short, it’s the collection of all different components of the application that are dependent on each other.
              We have two services namely, web and database. In Compose version 3, we can have multiple containers of the same service as well.

**web**: The name web is the name of our Flask app service. It can be anything. Docker Compose will create containers with this name.

**build**: This clause specifies the Dockerfile location. ‘.’ represents the current directory where the docker-compose.yml file is located and Dockerfile is used to build an image and run the container from it. We can also provide the absolute path to Dockerfile instead of the current working directory symbol.

**ports**: The ports clause is used to map the container ports to the host machine’s port. It creates a tunnel from the specified container port to the provided host machine’s port.
           This is the same as using the -p 5000:5000 option to map the container’s 5000 port to the host machine’s 5000 port while running the container using the docker run command.

**volumes**: This is the same as the -v option used to mount disks in docker run command. 
             Here, we are attaching our code files directory to the container’s /code directory so that we don’t have to rebuild the images for every change in the files.

**links**: Links literally link one service to another. In the bridge network, we have to specify which container should be accessible to what container using a link to the respective containers.

Here, we are linking the database container to the web container, so that our web container can reach the database in the bridge network.

**image**: If we don’t have a Dockerfile and want to run a service directly using an already built docker image, then specify the image location using the ‘image’ clause. Compose will pull the image and fork a container from it.

**environment**: Any environment variable that should be present in the container can be created using the environment clause. This does the same work as the -e argument in the docker run command while running a container.

## Docker Compose Commands Overview
##### docker-compose build 
This command builds images of the mentioned services in the docker-compose.yml file for which a Dockerfile is provided.

##### docker-compose images
This command lists images built using the current docker-compose file.

##### docker-compose run
Similar to docker run command, this one creates containers from images built for the services mentioned in the compose file.

##### docker-compose up
This does the job of the docker-compose build and docker-compose run commands. It initially builds the images if they are not located locally and then starts the containers.

##### docker-compose stop
This command stops the running containers of the specified services in the docker-compose file.

##### docker-compose ps
This lists all the containers for services mentioned in the current docker-compose file. The containers can either be running or stopped.

##### docker-compose logs
This command is similar to docker logs <container ID>
  

## Working with Multiple Dockerfiles

Suppose we have directory structure like this: 

![image](https://user-images.githubusercontent.com/33947539/147734790-f840b7e5-a5ed-48d7-85ea-768810de8e34.png)

```yaml
version: '3'
services:
  web:
    # Path to dockerfile.
    # '.' represents the current directory in which
    # docker-compose.yml is present.
    build: .

    # Mapping of container port to host
    
    ports:
      - "5000:5000"
    # Mount volume 
    volumes:
      - "/usercode/:/code"

    # Link database container to app container 
    # for rechability.
    links:
      - "database:backenddb"
    depends_on:
      - database
    
  database:

    # image to fetch from docker hub
    build:
      context: ./db
      dockerfile: Dockerfile-db
    #image: mysql/mysql-server:5.7

    # Environment variables for startup script
    # container will use these variables
    # to start the container with these define variables. 
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
      - "MYSQL_USER=testuser"
      - "MYSQL_PASSWORD=admin123"
      - "MYSQL_DATABASE=backend"
    # Mount init.sql file to automatically run 
    # and create tables for us.
    # everything in docker-entrypoint-initdb.d folder
    # is executed as soon as container is up nd running.
    volumes:
      - "/usercode/db/init.sql:/docker-entrypoint-initdb.d/init.sql"
    
```
In the above yaml , we are building web image using :
```yaml
   # docker-compose.yml is present.
    build: .
```
Here "." specifies the current directory where Dockerfile is present.
  
👉 But how to specify the Dockerfile inside "db" Folder.
  
```yaml
   # image to fetch from docker hub
    build:
      context: ./db
      dockerfile: Dockerfile-db
 ```
  

