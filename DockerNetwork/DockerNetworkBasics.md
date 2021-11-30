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

> 




