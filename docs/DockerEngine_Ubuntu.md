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




