## Docker Bridge
- 2 segments that the bridge connects is:
  - local host e.g. your laptop, or linux server where you run your Docker containers
  - virtual network created by Docker
  -       

### When you install Docker for the first time, it will create a default bridge network on the host (slightly different on MacOS as it will 
- all containers you create on that host will get an IP address on that range

On a Linux host, Docker Engine directly creates a docker0 bridge network interface in the host’s network stack. That’s the familiar default bridge you see with ifconfig or ip a (usually 172.17.0.1/16).
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
