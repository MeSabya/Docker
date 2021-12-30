# What Are Namespaces and cgroups, and How Do They Work?

## What Are Namespaces?

Namespaces are a feature of the Linux kernel that they isolate processes from each other. 
On a server where you are running many different services, isolating each service and its associated processes from other services.

- Namespaces provide a layer of isolation for containers.

- Each aspect of a container runs in a separate namespace and its access is limited to that namespace.

- When you run a container, Docker creates a set of namespaces for that container.

- Namespace makes processes running inside that namespace believe they have their own instance of that resource.

- A namespace can limit visibility to certain process trees, network interfaces, user IDs, or filesystem mounts.
### Types of Namespaces

Namespace Types:
  1. Process ID: in a pid namespace you become PID 1 and then your children are other processes. All the other programs are gone

  2. Mount: in a mount namespace you can mount and unmount filesystems without it affecting the host filesystem. So you can have a totally different set of devices mounted (usually less)
  
  3. IPC (Interprocess communication)
  
  4. User (currently experimental support for)
  
  5. Network: in a networking namespace you can run programs on any port you want without it conflicting with what’s already running.


## Cgroups 

Docker also makes use of kernel control groups for resource allocation and isolation. A cgroup limits an application to a specific set of resources. Control groups allow Docker Engine to share available hardware resources to containers and optionally enforce limits and constraints.

**Docker Engine uses the following cgroups:**

```Doc
Memory cgroup for managing accounting, limits and notifications.
HugeTBL cgroup for accounting usage of huge pages by process group.
CPU group for managing user / system CPU time and usage.
CPUSet cgroup for binding a group to specific CPU. Useful for real time applications and NUMA systems with localized memory per CPU.
BlkIO cgroup for measuring & limiting amount of blckIO by group.
net_cls and net_prio cgroup for tagging the traffic control.
Devices cgroup for reading / writing access devices.
Freezer cgroup for freezing a group. Useful for cluster batch scheduling, process migration and debugging without affecting prtrace.
```

![image](https://user-images.githubusercontent.com/33947539/147728822-74217d3e-57c2-4f20-bcd9-d13290b51ef4.png)
