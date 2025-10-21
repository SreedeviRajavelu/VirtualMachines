Excellent question — this is a **network/DNS resolution issue**, not a Docker configuration problem. Let’s break down what’s happening and how to fix it properly.

---

### 🧩 **What the error means**

```
Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io on 127.0.0.53:53: read udp 127.0.0.1:46176->127.0.0.53:53: i/o timeout
```

**Explanation:**

* Docker is trying to pull the image from Docker Hub (`registry-1.docker.io`).
* It’s using the system DNS resolver (`127.0.0.53`, which is `systemd-resolved`).
* The lookup request timed out, meaning **the VM can’t resolve DNS names** (even though it may have internet connectivity, e.g., `ping 8.8.8.8` works).

---

### ✅ **Step 1: Test DNS resolution on your Ubuntu VM**

Run:

```bash
ping -c 3 google.com
```

If it fails with “Temporary failure in name resolution,” DNS is the issue.

---

### ✅ **Step 2: Fix your `/etc/resolv.conf`**

Run this command to check the current content:

```bash
cat /etc/resolv.conf
```

You’ll likely see something like:

```
nameserver 127.0.0.53
options edns0 trust-ad
search ...
```

That’s **not helpful** for Docker containers when DNS forwarding breaks.

Replace it with **public DNS servers**:

```bash
sudo bash -c 'cat > /etc/resolv.conf <<EOF
nameserver 8.8.8.8
nameserver 1.1.1.1
EOF'
```

---

### ✅ **Step 3: Restart networking and Docker**

```bash
sudo systemctl restart systemd-resolved
sudo systemctl restart docker
```

Then try again:

```bash
docker compose up -d
```

---

### ⚙️ **Optional: Persist DNS changes (if using cloud-init or Netplan)**

Since your `/etc/netplan/50-cloud-init.yaml` file had:

```yaml
nameservers:
  addresses:
    - 127.0.0.53
    - 8.8.8.8
    - 8.8.4.4
```

Update it to **remove 127.0.0.53**, like this:

```yaml
nameservers:
  addresses:
    - 8.8.8.8
    - 8.8.4.4
```

Then apply the changes:

```bash
sudo netplan apply
```

---

### 🧠 **Why this fixes it**

Docker relies on the host’s DNS resolution to fetch images.
When Ubuntu uses `systemd-resolved` (127.0.0.53) and that resolver breaks (common on VMs), Docker can’t resolve `registry-1.docker.io`.
Setting `/etc/resolv.conf` to external DNS servers like Google or Cloudflare bypasses that problem.

---

Would you like me to give you a **version of this fix that persists across reboots** (since cloud-init can overwrite `/etc/resolv.conf` each time)?


Perfect — that’s a **very common situation** on Ubuntu 20.04+ and 22.04 systems.
You’re correct not to directly edit `/etc/resolv.conf` because it’s **a symlink managed by `systemd-resolved`**, not a static file.

Here’s exactly what you should do 👇

---

## 🧭 Step-by-step fix for DNS resolution (recommended way)

### 🧩 Step 1: Confirm the symlink

Run:

```bash
ls -l /etc/resolv.conf
```

You’ll likely see something like:

```
/etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
```

This confirms `systemd-resolved` is managing it.

---

### 🧩 Step 2: Tell systemd-resolved to use public DNS servers

Create or edit this file:

```bash
sudo nano /etc/systemd/resolved.conf
```

Find or add these lines under `[Resolve]`:

```ini
[Resolve]
DNS=8.8.8.8 8.8.4.4
FallbackDNS=1.1.1.1
DNSStubListener=no
```

> 💡 The `DNSStubListener=no` line disables the local 127.0.0.53 stub that’s currently breaking Docker lookups.

Save and exit (`Ctrl+O`, `Enter`, then `Ctrl+X`).

---

### 🧩 Step 3: Point `/etc/resolv.conf` to the *real* resolv file

Replace the current symlink:

```bash
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

This file (`/run/systemd/resolve/resolv.conf`) contains the actual upstream DNS servers rather than the local stub.

---

### 🧩 Step 4: Restart systemd-resolved

```bash
sudo systemctl restart systemd-resolved
```

Then verify:

```bash
resolvectl status
```

You should now see:

```
DNS Servers: 8.8.8.8 8.8.4.4 1.1.1.1
```

---

### 🧩 Step 5: Test DNS resolution

```bash
ping -c 3 google.com
```

If that works, try:

```bash
docker compose up -d
```

It should now be able to pull from Docker Hub successfully.

---

### 🧠 Why this method is best

This approach:

* Keeps `systemd-resolved` in control (so no conflicts with Ubuntu updates).
* Permanently replaces the stub DNS (127.0.0.53) with direct DNS servers.
* Persists across reboots and cloud-init updates.

---

Would you like me to show you **the persistent fix that also survives a full VM reboot or cloud-init regeneration** (so it’s never reset back to 127.0.0.53)?

## Persistent Fix 

Excellent — let’s make your DNS fix **fully persistent**, even across reboots and `cloud-init` regenerations.

Here’s the **robust setup** that ensures your VM *always* keeps working DNS (and Docker can always pull images).

---

## 🧭 Goal

Stop Ubuntu’s `cloud-init` and `systemd-resolved` from resetting `/etc/resolv.conf` to `127.0.0.53`
✅ Survives reboots
✅ Survives `cloud-init` reconfiguration
✅ Keeps Docker name resolution reliable

---

## 🔧 Step 1: Disable `cloud-init` network management (optional but recommended)

Cloud-init regenerates `/etc/netplan/50-cloud-init.yaml` and resets DNS.
To prevent that:

```bash
sudo mkdir -p /etc/cloud/cloud.cfg.d/
echo "network: {config: disabled}" | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

This stops cloud-init from overwriting your network configuration or DNS settings.

---

## 🔧 Step 2: Configure permanent DNS in `systemd-resolved`

Edit the config file:

```bash
sudo nano /etc/systemd/resolved.conf
```

Update it to include your preferred DNS servers:

```ini
[Resolve]
DNS=8.8.8.8 8.8.4.4
FallbackDNS=1.1.1.1
DNSStubListener=no
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

---

## 🔧 Step 3: Replace `/etc/resolv.conf` with the real resolver file

Right now `/etc/resolv.conf` likely points to:

```
/run/systemd/resolve/stub-resolv.conf
```

…which uses the stub `127.0.0.53` listener.

Replace it permanently:

```bash
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

That file (`/run/systemd/resolve/resolv.conf`) always reflects the real upstream DNS servers (like 8.8.8.8).

---

## 🔧 Step 4: Restart services

```bash
sudo systemctl restart systemd-resolved
```

Then check status:

```bash
resolvectl status
```

You should now see:

```
DNS Servers: 8.8.8.8 8.8.4.4 1.1.1.1
DNS Stub Listener: no
```

---

## 🔧 Step 5: Verify DNS

Run:

```bash
ping -c 3 google.com
```

and:

```bash
docker pull hello-world
```

Both should now succeed instantly.

---

## 🧠 Why this is persistent and safe

| Component          | Problem                         | What we did                           |
| ------------------ | ------------------------------- | ------------------------------------- |
| `cloud-init`       | Rewrites network config         | Disabled network reconfiguration      |
| `systemd-resolved` | Uses stub resolver (127.0.0.53) | Pointed it to public DNS directly     |
| `/etc/resolv.conf` | Symlinked to stub file          | Re-linked to the real resolver        |
| Reboot resets      | Temporary files regenerated     | `/etc/systemd/resolved.conf` persists |

---

## 🧩 Optional: make sure `/etc/netplan` matches your setup

If your `/etc/netplan/50-cloud-init.yaml` defines DNS like:

```yaml
nameservers:
  addresses:
    - 127.0.0.53
```

You can safely change it to:

```yaml
nameservers:
  addresses:
    - 8.8.8.8
    - 8.8.4.4
```

Then apply it:

```bash
sudo netplan apply
```

---

✅ **After doing all this:**
Your DNS resolution (and Docker access) will remain correct permanently — even after reboot, `apt upgrade`, or VM image regeneration.

---

Would you like me to show how to **verify after reboot** that your changes persist (a quick 2-command check)?


### Persistent DNS + network configuration fix across reboots and cloud-init resets.


'''
explain each of these commands :

sudo mkdir -p /etc/cloud/cloud.cfg.d/
echo "network: {config: disabled}" | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

this is the content of /etc/netplan/50-cloud-init.yaml , and there is a comment that says any changes will not persist across an instance reboot  , what should I do to persist changes on this file too? summarise and consolidate all changes that I need to do to resolve the error here , to persist and to verify it after reboot 

ubuntu@pgt:~/ccp_multi_IED_multi_plc_database_pg$ sudo cat /etc/netplan/50-cloud-init.yaml 
[sudo] password for ubuntu: 
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        enp0s3:
            dhcp4: false
            addresses:
              - 192.168.30.60/24
            nameservers:
              addresses:
                - 127.0.0.53
                - 8.8.8.8
                - 8.8.4.4
            routes:
              - to: default
                via: 192.168.30.1
    version: 2
ubuntu@pgt:~/ccp_multi_IED_multi_plc_database_pg$ 


drwxr-xr-x 6 ubuntu ubuntu 4096 Oct 21 02:20 ccp_multi_IED_multi_plc_database_pg
ubuntu@pgt:~$ cd ccp_multi_IED_multi_plc_database_pg/
ubuntu@pgt:~/ccp_multi_IED_multi_plc_database_pg$ docker compose up -d
[+] Running 1/1
 ✘ postgres Error Get "https://registry-1.docker.io/v2/...                10.2s 
Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io on 127.0.0.53:53: read udp 127.0.0.1:46176->127.0.0.53:53: i/o timeout
ubuntu@pgt:~/ccp_multi_IED_multi_plc_database_pg$ ping -c 3 google.com
ping: google.com: Temporary failure in name resolution
ubuntu@pgt:~/ccp_multi_IED_multi_plc_database_pg$ ping -c 3 google.com
ping: google.com: Temporary failure in name resolution
ubuntu@pgt:~/ccp_multi_IED_multi_plc_database_pg$ cat /etc/resolv.conf
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search .
ubuntu@pgt:~/ccp_multi_IED_multi_plc_database_pg$ ls -l /etc/resolv.conf
lrwxrwxrwx 1 root root 39 Sep 11  2024 /etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
ubuntu@pgt:~/ccp_multi_IED_multi_plc_database_pg$ ls /etc/netplan/
50-cloud-init.yaml
content of file /etc/netplan/50-cloud-init.yaml:^C
'''



Perfect — you’ve given me everything I need. Let’s walk through **what’s happening**, **what those commands do**, and then I’ll give you a **complete, persistent DNS + network configuration fix** that **survives reboots and cloud-init resets**.

---

## 🧩 What’s happening

* Your VM’s `/etc/resolv.conf` is **symlinked** to a dynamically managed file:

  ```
  /etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
  ```

  This is controlled by **systemd-resolved**.
  So even if you edit it manually, it gets overwritten after reboot or service restart.

* The file `/etc/netplan/50-cloud-init.yaml` is **auto-generated by cloud-init**.
  The comment:

  > “Changes to it will not persist across an instance reboot”
  > means cloud-init rewrites it every time the VM boots.

* Your DNS (`127.0.0.53`) is a **local DNS stub resolver**, which is failing to resolve queries.

---

## 🧠 Step 1: Explanation of the commands you mentioned

```bash
sudo mkdir -p /etc/cloud/cloud.cfg.d/
```

* Creates the directory `/etc/cloud/cloud.cfg.d/` if it doesn’t exist.
* This directory stores override configurations for **cloud-init**.

```bash
echo "network: {config: disabled}" | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

* This disables **cloud-init’s automatic network configuration**, preventing it from regenerating `/etc/netplan/50-cloud-init.yaml` at each reboot.
* Once disabled, **you control networking manually via Netplan**.

---

## ⚙️ Step 2: Apply persistent DNS + network fix

Follow these steps exactly in order:

### 1️⃣ Disable cloud-init’s network regeneration

```bash
sudo mkdir -p /etc/cloud/cloud.cfg.d/
echo "network: {config: disabled}" | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

### 2️⃣ Create your own persistent netplan file

Create `/etc/netplan/01-netcfg.yaml` (custom, not cloud-init generated):

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Paste the following content:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.30.60/24
      gateway4: 192.168.30.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

> ✅ Explanation:
>
> * `renderer: networkd` ensures systemd-networkd handles the interface.
> * We’ve **removed 127.0.0.53** so DNS goes directly to Google’s servers.
> * This file is **not** managed by cloud-init, so it persists.

### 3️⃣ Apply and verify

```bash
sudo netplan apply
```

Then test:

```bash
ping -c 3 google.com
```

If ping works — DNS is fixed.

---

## 🔁 Step 3: Make `/etc/resolv.conf` static and persistent

To break the dynamic symlink and use a static resolv.conf:

```bash
sudo rm /etc/resolv.conf
sudo bash -c 'cat > /etc/resolv.conf <<EOF
nameserver 8.8.8.8
nameserver 8.8.4.4
EOF'
```

Then **make sure it isn’t replaced** by systemd-resolved:

```bash
sudo systemctl disable systemd-resolved.service
sudo systemctl stop systemd-resolved.service
```

Now `/etc/resolv.conf` will remain **static and persistent**.

---

## ✅ Step 4: Verify persistence after reboot

After reboot:

```bash
cat /etc/resolv.conf
ls -l /etc/netplan/
cat /etc/netplan/01-netcfg.yaml
```

You should see:

* `resolv.conf` no longer symlinked.
* DNS pointing to 8.8.8.8 and 8.8.4.4.
* `netplan` using your custom YAML.
* Network still working (`ping google.com` succeeds).

---

## 🧾 Summary of all changes (to persist DNS & fix error)

| Step | Action                                                                                      | Purpose                                 | Persistent? |
| ---- | ------------------------------------------------------------------------------------------- | --------------------------------------- | ----------- |
| 1    | `sudo mkdir -p /etc/cloud/cloud.cfg.d/`                                                     | Ensure override directory exists        | ✅           |
| 2    | `echo "network: {config: disabled}" > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg` | Disable cloud-init network resets       | ✅           |
| 3    | Create `/etc/netplan/01-netcfg.yaml`                                                        | Manual static IP + DNS setup            | ✅           |
| 4    | `sudo netplan apply`                                                                        | Apply new config                        | ✅           |
| 5    | Remove symlink `/etc/resolv.conf` and create static file                                    | Fix DNS permanently                     | ✅           |
| 6    | `sudo systemctl disable systemd-resolved`                                                   | Prevent it from overwriting resolv.conf | ✅           |

---

Would you like me to show a **minimal “copy-paste-ready” version** of all commands you can run in sequence (no explanations, just working setup)?


## After sudo netplan apply in the above steps, still got error in DNS name resolution

After doing sudo netplan apply, below is the output I get , why do I still get failure in name resolution and how to resolve

```
ubuntu@pgt:~$ ping -c 3 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=2.22 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=2.30 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=115 time=2.43 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 2.224/2.316/2.425/0.082 ms
ubuntu@pgt:~$ ping -c 3 google.com
ping: google.com: Temporary failure in name resolution
ubuntu@pgt:~$ 
```

Good observation — this tells us **exactly what’s working and what’s not** 👇

* ✅ `ping 8.8.8.8` works → **your network routing and internet connectivity are fine.**
* ❌ `ping google.com` fails → **your DNS resolution is broken.**

Let’s go step-by-step to fix and persist it properly.

---

### 🧩 Step 1: Confirm what `/etc/resolv.conf` points to

Run:

```bash
ls -l /etc/resolv.conf
cat /etc/resolv.conf
```

You will almost certainly see this:

```
/etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
nameserver 127.0.0.53
```

That means DNS queries are still going through the **systemd-resolved stub** (`127.0.0.53`), which is not using your Netplan `8.8.8.8` entries.

---

### 🧰 Step 2: Point resolv.conf to the real DNS configuration

You can safely replace that symlink with one that points to the correct file managed by `systemd-resolved`:

```bash
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

That file (`/run/systemd/resolve/resolv.conf`) contains the **real upstream nameservers** that systemd-resolved learned from Netplan.

Now check again:

```bash
cat /etc/resolv.conf
```

✅ Expected:

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Then test:

```bash
ping -c3 google.com
```

If it resolves, DNS is now fixed.

---

### 🧩 Step 3: Make sure this survives reboot

You’ve already disabled Cloud-Init network overwriting with:

```bash
echo "network: {config: disabled}" | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

and moved `50-cloud-init.yaml`, so this DNS configuration will persist.

The `/etc/resolv.conf` symlink fix also persists automatically because it’s a filesystem change, not a runtime file.

---

### 🧩 Step 4: Optional verification after reboot

After you reboot:

```bash
cat /etc/resolv.conf
resolvectl status
ping -c3 google.com
```

All should show Google’s DNS and successful resolution.

---

✅ **Summary of what was wrong and what fixed it**

* Problem: `/etc/resolv.conf` still pointed to the stub at `127.0.0.53`, ignoring your Netplan DNS.
* Fix: Replace that symlink to point to `/run/systemd/resolve/resolv.conf`, which contains your correct DNS servers.

After that, Docker and all other applications should be able to resolve hostnames normally.


