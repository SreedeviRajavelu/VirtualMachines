## Docker Networking Tutorial (Bridge - None - Host - IPvlan - Macvlan - Overlay)

Reference: https://youtu.be/fBRgw5dyBd4?si=LodjJDx4AdnjYOS6

## Docker Bridge
- 2 segments that the bridge connects is:
  - local host e.g. your laptop, or linux server where you run your Docker containers
  - virtual network created by Docker
  -       

### When you install Docker for the first time, it will create a default bridge network on the host (slightly different on MacOS as it will run al containers inside a Linux VM, virtualization framework)  
- all containers you create on that host will get an IP address on that range

On a Linux host, Docker Engine directly creates a docker0 bridge network interface in the host’s network stack. That’s the familiar default bridge you see with ifconfig or ip a (usually 172.17.0.1/16).
<img width="884" height="610" alt="image" src="https://github.com/user-attachments/assets/0f59a653-6033-4d16-b14b-aaaf61bfb04f" />


On a Mac (macOS) or Windows, things are different:

Docker Desktop does not create a native docker0 bridge interface on your MacBook’s network stack.

Instead, Docker Desktop runs all containers inside a lightweight Linux VM (on macOS, this is managed using Apple’s HyperKit or now a Virtualization Framework).

Inside that VM, Docker does create a docker0 bridge (just like on Linux). But this bridge is inside the VM, not on macOS itself.

From your MacBook’s perspective, you won’t see docker0 if you run ifconfig or ip a. Instead, Docker Desktop sets up a special networking layer that forwards traffic between your MacBook and the VM.

So to answer directly:

👉 No, Docker Desktop does not create a default bridge network on your MacBook’s host network stack.
👉 Yes, it does create a default bridge network inside the Linux VM where your containers run.

If you run:
` docker network ls `
you’ll still see bridge, host, and none listed, because those networks exist in the Docker VM environment, even though your Mac itself doesn’t expose them as interfaces.

Do you want me to also explain how you can check the actual IP range of the bridge network inside Docker Desktop on your Mac?


## Containers created in the default bridge network
- can communicate with each other
- default bridge network has restrictions and is not recommended for production, better to use user-defined bridge network.
- Default bridge network: Cannot use container DNS from host 
- In default bridge network, DNS is not supported. Also cannot use DNS to send requests between containers.

## User defined bridge network**: 
- **Can use DNS to send requests to containers.** 
- **This only applies to communications inside the bridge network between containers. You still will NOT be able to use the container DNS from the host.**


## Host mode - networking option 
- container will not get its own IP and instead share the same networking namespace as the host where you run the container
- will appear as if you were running a regular application on that host
- thus any application running on a different server will be able to access the container using the host's IP address

## IPvlan network:
- Traditionally, to expose a container to the outside world we used bridge network. But it adds additional complexity and performance penalty. Packet needs to go through additional hop, need to map ports from the container to the host to expose it to other applications.  
- Does not use a bridge for isolation and is associated directly with the Linux network interface. No need for port mappings in these scenarios.


## MacVLAN network:
- Some legacy applications and those that monitor network traffic expect to be directly connected to the physical network.
- In this case, can use the MacVLAN network driver to assign a MAC address to each container's virtual network interface.
- It will appear as a physical network interface directly connected to the physical network.
- to create it, must specify the subnet that the host uses, the gateway, and the parent network interface
- compare the parent MAC address (parent interface) -> e.g. ip addr show ens33
- MAC address of the container -> ssh into it and do `ip addr`

### Difference between IPVLAN and MACVLAN:
- if use IPVLAN: container will get the same MAC address as your host
- if use MACVLAN: container will have a different MAC address from host


## Overlay network
- When you deploy your applications to production, you will need more than one physical or virtual server, each with a Docker daemon installed.
- The overlay network driver creates a distributed network among multiple Docker daemon hosts.
- This network sits on top of (overlays) the host-specific networks, allowing containers connected to it to communicate securely, especially when encryption is enabled.
-  Most frequently, this type of network is used with **Docker Swarm** , but it is also possible to connect individual containers.
-  Comment from youtuber: to manage containers at scale, especially in production, consider using Kubernetes.
-  need to disable
-  Example of usage of overlay network:
  -  2 ubuntu VMs
  -  find network interfaces on both VMs, e.g. ens33 on both VMs -> disable this on the network interface on both VMs
  -  `sudo ethtool -K ens33 tx-checksum-ip-generic off` (not persistent across restarts so might need to use something like a script using systemd service to automatically run it on boot)
  -  Even if you want to connect individual containers to overlay network, we still need to initialize the Docker Swarm
  -  on the first VM, run `docker swarm init`
  -  will give a command you can execute on other VMs to join the Docker Swarm
  -  each VM must have docker installed
  -  manager and worker nodes (the VMs)
  -  on manager, create an overlay network and an attachable flag for individual containers to be able to use this network. Otherwise only swarm services will be able to use it.
  -  on the worker node (VM) if you do `docker network ls`, you will only see the overlay network when you start a container that uses the network (can start on the manager node)
  -  to verify that we can access containers on that overlay network deployed on different VMs, can SSH to the second container and use curl to send a request to the first container on the manager node
  -  
  -  

