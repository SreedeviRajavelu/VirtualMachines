Steps to install Docker Engine on ubuntu 24.04.2 :

- https://docs.docker.com/engine/install/ubuntu/

Check ubuntu version using:

- `lsb_release -a`
- `cat /etc/os-release`


<img width="867" height="462" alt="image" src="https://github.com/user-attachments/assets/dc736769-198d-4213-b71e-260b3690c557" />


Check CPU architecture
- `uname -m`
  
- x86_64 → 64-bit Intel/AMD (amd64) ✅ supported
- aarch64 → 64-bit ARM (arm64) ✅ supported
- armv7l → 32-bit ARM ❌ not supported by Docker Engine
- s390x, ppc64le → supported but only on special hardware

On a VirtualBox VM on a MacBook, you’ll almost certainly see x86_64 if your Mac is Intel-based, or aarch64 if your MacBook is Apple Silicon (M1/M2/M3).

<img width="372" height="71" alt="image" src="https://github.com/user-attachments/assets/639531f5-55df-496a-874c-16440626e7e2" />



Steps taken from https://docs.docker.com/engine/install/ubuntu/ 

Run the following command to uninstall all conflicting packages:

```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

**Install using the apt repository**

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker apt repository. Afterward, you can install and update Docker from the repository.

1. Set up Docker's apt repository.

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. Install the Docker packages

To install the latest version, run:

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Note:
The Docker service starts automatically after installation. To verify that Docker is running, use:


```sudo systemctl status docker```

Some systems may have this behavior disabled and will require a manual start:

```sudo systemctl start docker```

## Issues and Fixes

`docker --version` works but 

for `docker ps` gives **permission denied error** meaning current user **ubuntu** does not have permission to access the Docker socket 

**Solution:** 

1. Check the socket permissions 

`ls -l /var/run/docker.sock`

If you see something like:
`srw-rw---- 1 root docker 0 Oct 10 13:20 /var/run/docker.sock`

That means only users in the docker group can use it.

2. Add your user to the docker group

`sudo usermod -aG docker $USER`

Then log out and log back in - the new group membership only takes effect after re-login