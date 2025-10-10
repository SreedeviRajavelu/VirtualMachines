## Docker Macvlan Network 

A macvlan nerwork allows each Docker container to appear on the same Layer 2 network as your host (Ubuntu VM).

- Each container gets its **own unique IP address** on your **physical LAN** (e.g. 192.168.30.x)

- The containers can be reached directly by other devices on that LAN - without NAT.

Because of that, the macvlan network must be bound to your host's physical network interface (the one actually connected to your LAN or VM bridge).




