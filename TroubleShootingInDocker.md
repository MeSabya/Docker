# Troubleshooting in Docker

### Wrong build path

```
docker build .
unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat /Users/venkateshachintalwar/Documents/Online_Projects/Dockerfile: no
such file or directory
```
If you are accidentally not in the directory where Dockerfile is located, Docker will throw an error. cd into the directory where Dockerfile is located and that should solve the error.

### Permission issues
```
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```
If you face this kind of permission issues, you should either run the command with sudo or do the following:
```
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```
This will add the current user to the Docker group which gives the current user the entire super-user level access for Docker.

### Port issues

```
$ docker run -p 5000:5000 flask_app:1.0
docker: Error response from daemon: driver failed programming external connectivity on endpoint wizardly_ramanujan (d51f0d6f47ed3d559e767191adcd3b71fc364d3d4ae3433787e0e2555948dcc8): Bind for 0.0.0.0:5000 failed: port is already allocated.
```

This happens when you try to map an already-occupied host port to a container port.

There are two ways to solve this:

Type docker ps which will list all the running containers on the machine.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
a15ad5a56847        flask_app:1.0       "flask run"         14 hours ago        Up 14 hours         0.0.0.0:5000->5000/tcp   festive_ganguly
```
If you notice the ports column, you can see the 5000 port is already in use. Stop the container using docker stop <CONTAINER ID> and then retry.

When you don’t see the port in docker ps command output, it means the port is used by some other application. Use lsof -i TCP:<PORT>
This will print all the services using the specified port, then, you can kill one of them using pkill utility, but take precautions before killing a process.

  ```
$ lsof -i TCP:5000
COMMAND     PID                 USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
com.docke   999 venkateshachintalwar   43u  IPv6 0xa23249e6f915f25b      0t0  TCP *:commplex-main (LISTEN)
Python    43184 venkateshachintalwar    9u  IPv4 0xa23249e6f91699a3      0t0  TCP *:commplex-main (LISTEN)
```
### Space issues
>When building an image, Docker pulls a lot of prebuilt images from the Docker registry and stores them on the local machine. Building images frequently makes Docker pull a lot of images and can result in space issues on the disk.

There are a couple of ways to tackle this problem:

Figure out how much space is used by Docker. Type docker system df. This will print space used by Docker in a human-readable format.
```
$ docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              36                  6                   12.19GB             11.48GB (94%)
Containers          18                  1                   28.31MB             28.06MB (99%)
Local Volumes       33                  0                   1.818GB             1.818GB (100%)
Build Cache         0                   0                   0B                  0B
```
This is my system’s stat. Notice the total reclaimable column. We can reclaim that much space for Docker.

Type docker system prune. This will prompt you to notify what it should remove.

```
$ docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

Are you sure you want to continue? [y/N] y
Deleted Containers:
bd4fb9779be4c7a4306312a24ad9b3ff0e8eaf6bb3adccfc6611c9d4a44c716b
2db0f9c68e21897b4238392fe2947749f8b57d52227c80e29293a5ebcb39a180
43f3d18d03f5c5616e9b261163599695d90c52034411f688cc4d086b51aa08e7
```
### Container inspection
You can get all configurations of a container, an image or a network. 
>docker inspect <Container/Image/Network ID>
  
### How can I debug "ImagePullBackOff"?
For OpenShift use:

```bash  
oc describe pod <pod-id>  
  
For vanilla Kubernetes:

kubectl describe pod <pod-id>  
```
Examine the events of the output. In my case it shows Back-off pulling image unreachableserver/nginx:1.14.22222

In this case the image unreachableserver/nginx:1.14.22222 can not be pulled from the Internet because there is no Docker registry unreachableserver and the image nginx:1.14.22222 does not exist.

NB: If you do not see any events of interest and the pod has been in the 'ImagePullBackOff' status for a while (seems like more than 60 minutes), you need to delete the pod and look at the events from the new pod.

For OpenShift use:

```bash
oc delete pod <pod-id>
oc get pods
oc get pod <new-pod-id>
For vanilla Kubernetes:

kubectl delete pod <pod-id>  
kubectl get pods
kubectl get pod <new-pod-id>
```
  
Sample output:

  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  32s                default-scheduler  Successfully assigned rk/nginx-deployment-6c879b5f64-2xrmt to aks-agentpool-x
  Normal   Pulling    17s (x2 over 30s)  kubelet            Pulling image "unreachableserver/nginx:1.14.22222"
  Warning  Failed     16s (x2 over 29s)  kubelet            Failed to pull image "unreachableserver/nginx:1.14.22222": rpc error: code = Unknown desc = Error response from daemon: pull access denied for unreachableserver/nginx, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
  Warning  Failed     16s (x2 over 29s)  kubelet            Error: ErrImagePull
  Normal   BackOff    5s (x2 over 28s)   kubelet            Back-off pulling image "unreachableserver/nginx:1.14.22222"
  Warning  Failed     5s (x2 over 28s)   kubelet            Error: ImagePullBackOff


**Additional debugging steps**:

- try to pull the docker image and tag manually on your computer
- Identify the node by doing a 'kubectl/oc get pods -o wide'
- ssh into the node (if you can) that can not pull the docker image
- check that the node can resolve the DNS of the docker registry by performing a ping.
- try to pull the docker image manually on the node
- If you are using a private registry, check that your secret exists and the secret is correct. Your secret should also be in the same namespace. Thanks swenzel
- Some registries have firewalls that limit ip address access. The firewall may block the pull
- Some CIs create deployments with temporary docker secrets. So the secret expires after a few days (You are asking for production failures...)

  
  
