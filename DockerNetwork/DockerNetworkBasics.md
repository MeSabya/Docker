# Docker Networking

## Networking Fundamentals:

- **Layer 2**: is the data link layer, a protocol layer that transfers frames between nodes in a typical wide area network. An example protocol in this layer would be ARP, which   discovers MAC addresses with its IP address.

- **Layer 3**: is the network layer, a routing layer that transfers packets between local area network hosts. Example protocols in this layer would be IP and ICMP (ping   
  command).

- **NAT**: Network Address Translation, provides a simple mapping from one IP address (or subnet) to another. Typically, the NAT gives the kernel the ability to provide  
  “virtual” large private networks to connect to the Internet using a single public IP address. To achieve the above, the NAT maintains a set of rules (generally speaking, ports   masquerade and translation).

- **Bridge**: the Network Bridge is a device (can also be a virtual one) that creates a communication surface which connects two or more interfaces to a single flat broadcast 
  network. The Bridge uses a table, forwarding information base, maintains a forwarding pairs entries (for example, record might look like MAC_1 → IF_1).

- **Network Namespace**: by namespacing, the kernel is able to logically separate its processes to multiple different network “areas”. Each network looks like a “standalone”  
  network area, with its own stack, Ethernet devices, routes and firewall rules. By default, the kernel provides a “default” namespace in its bootstrap (if not stating  
  otherwise, every process will be spawned/forked in the default namespace). Every child process inherits its namespace from its ancestors.

- **Veth**: Virtual Ethernet device is a virtual device that acts as a tunnel between network namespaces. These devices create interconnected peering between the two connected 
  links and pass direct traffic between them. This concept mainly belongs to UNIX OS. On Windows OS it works differently.
  
  >Let’s assume that you have two computers. You probably know you can create a network link between those two computers via an ethernet cable, and you don’t need a network switch for that.
You can have a similar setup in your Linux server via VETH (Virtual Ethernet Devices) interfaces. You can create a virtual ethernet device that works like an actual ethernet network adapter. The virtual ethernet device will act as a tunnel between the network namespaces that we will create.

##### Reference for bridge and veth  : https://adil.medium.com/container-networking-under-the-hood-network-namespaces-6b2b8fe8dc2a 

## Docker Network Basics

> When you install docker it creates three networks automatically - Bridge, Host, and None. 
> **Bridge is the default network** a container gets attached to when it is run. 
> To attach the container to any other network you can use the **--network flag** of the run command.


![image](https://user-images.githubusercontent.com/33947539/144038314-fa8c0ed9-ba1b-4043-bbf6-02e6f14422d2.png)

### Bridge network
- The Bridge network assigns IPs in the range of 172.17.x.x to the containers within it
- To access these containers from outside you need to map the ports of these containers to the ports on the host.
- As an example, consider you can have a Docker container running a web service on port  80  . Because this container is attached to the bridge network on a private subnet, 
  a port on the host system like  8000  needs to be mapped to port  80 on the container for outside traffic to reach the web service.
  
  ![image](https://user-images.githubusercontent.com/33947539/144073898-166d3703-a627-4f4e-b622-614ed0b74cf9.png)

#### what's the docker0? is it an interface or bridge? if is an interface, does it have an IP address?
> The docker0 bridge is virtual interface created by docker, it randomly chooses an address and subnet from the private range defined by RFC 1918 that are not in use on the host machine, and assigns it to docker0.
**All the docker containers will be connected to the docker0 bridge by default**, the docker containers connnected to the docker0 bridge could use the iptables NAT rules created by docker to communicate with the outside world.

![image](https://user-images.githubusercontent.com/33947539/144084414-150314eb-a97f-4de1-b9da-f3c2b07e6b7d.png)

> By default Docker will attach all containers to the docker0 bridge, so you do not need to specify any additional flag with docker run command to connect the docker containers to the docker0 bridge, unless the DOCKER_OPTS in docker configuration file explicitly specifies to use the other network than docker0 bridge, in this case you could use –net=bridge with docker run command to connect the containers to the docker0 bridge.

> It is private virtual network internal to the host so containers on this network can communicate with each other.

You can inspect the bridge network to see all containers that are a part of it.

```bash
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "11a5802de2f923b1e532f210f54d9c153ed9d5922d941252c287b7e7d5893e66",
... //Output truncated
"Containers": {
            "bcdd99ca3f9d5bddf71bb7e504d7eac76accc080ff3f2058c6a2802b5881ab05": {
                "Name": "vigilant_solomon",
                "EndpointID": "cf28b42339427ab01086969056bcc490e779ed5ea2a7a89e3a3b5d6c82f20117",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "d8ba83cd292c3a58a39e6e3f915ebaf4e1c677e2a9bf68741cc5593fb32455d4": {
                "Name": "cranky_tharp",
                "EndpointID": "226e236f506b2290b82eaed49328edb39a170c22098e0f1f427659e56e2e1261",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
```

### IPAddresses of Host and docker0

docker0 is a network configured on host itself. You can verify this by running the following command

```
$ ip address show   
//Output    
2: wlp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 14:4f:8a:1b:da:2d brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.28/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp1s0
       valid_lft 86009sec preferred_lft 86009sec
    inet6 fe80::628b:6852:a41c:3c0c/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
//Output truncated
5: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:4b:af:1f:14 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:4bff:feaf:1f14/64 scope link 
       valid_lft forever preferred_lft forever
10: br-e9785df9979a: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:f7:0a:b7:08 brd ff:ff:ff:ff:ff:ff
//Output truncated

```
👉 ***Here we can clearly see that host has a different ip from that of the nginx container we created and docker0.***

## Creating our own network

👉 **Most Importantly**: *Let’s consider a scenario where we have two different apps running, which are not related to each other. How would we ensure no communication should be allowed between containers of different apps in such a scenario?*

- We can create our own networks to isolate containers logically.

👉 **Most Importantly**: Containers in the same network can talk to each other.

![image](https://user-images.githubusercontent.com/33947539/149340608-4c8e0e4b-7f5c-4277-be7c-e4b1bfdd0480.png)

```bash
$ docker network create app
72121d1686e9ec5fdccbb56aca08856b833a2f99fadde81f934ef34152013efb
$ docker network create web
73bf7444ce0c92291548deb3ec960d9ff784f95012fe43e1e7204f5615adf735

$ docker network ls        
NETWORK ID          NAME                DRIVER              SCOPE
72121d1686e9        app                 bridge              local
86dfc4dde9c5        bridge              bridge              local
f42988e2906f        host                host                local
faf8b8018ad9        none                null                local
73bf7444ce0c        web                 bridge              local

$ docker container run -d --network app httpd 
fd47b8faf3b1e7219eac13659a4a8718fe17c53ea49bbbbf4819a3ccdb740efa

$ docker container run -d --name db --network app mongo
b599adadd446d3cbc37f8994b5ceed05d0e16eb6b5dcda8d2be019c0c6e9cf2e

$ docker container run -d --name webdb --network web mongo
257de44d7863ae4c922bb29f6675f48ba277ac2b4822a22ea66e9baaee9b0a3c

//Start a bash session inside the apache container
$ docker exec -it fd4 bash     
root@4f6bb4dcd47b:/usr/local/apache2#

//Install iputils-ping to ping to talk to mongo container
root@4f6bb4dcd47b:/usr/local/apache2# apt update && apt install iputils-ping -y

//Find out the ip address of db container
$ docker container inspect --format '{{json .NetworkSettings.Networks}}' db
{"app":{"IPAMConfig":null,"Links":null,"Aliases":["b599adadd446"],"NetworkID":"72121d1686e9ec5fdccbb56aca08856b833a2f99fadde81f934ef34152013efb","EndpointID":"e5967f439c3b2cb7d7f34407b9512c404187dbe726973c978fc2cb129795a8dc","Gateway":"172.21.0.1","IPAddress":"172.21.0.3","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:15:00:03","DriverOpts":null}}

//ping mongo container using the ip address we saw above
root@fd47b8faf3b1:/usr/local/apache2# ping 172.21.0.3
PING 172.21.0.3 (172.21.0.3) 56(84) bytes of data.
64 bytes from 172.21.0.3: icmp_seq=1 ttl=64 time=0.267 ms
64 bytes from 172.21.0.3: icmp_seq=2 ttl=64 time=0.109 ms
64 bytes from 172.21.0.3: icmp_seq=3 ttl=64 time=0.132 ms
```

>Great! we are able talk to the mongo container in the same network.


Now, let’s try to ping webdb container of other network. Find out the ip address the same way we did before

```bash
root@fd47b8faf3b1:/usr/local/apache2# ping 172.22.0.2
PING 172.22.0.2 (172.22.0.2) 56(84) bytes of data.
//It will not return anything
```

👉 **Most Importantly**: Thus we can ensure containers from different apps can’t talk to each other.
>Bonus info : Containers in different networks can’t talk to each other as they lie in different subnet.















