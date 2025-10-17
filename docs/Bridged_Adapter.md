Excellent question — and one that a lot of people get tripped up on when mixing **VirtualBox/VMware networking modes** with **Docker networking**.

Let’s go step by step to make it crystal clear **why we can conclude that `enp0s3` is the interface tied to the bridged adapter** in your Ubuntu VM setup.

---

### 🧩 Step 1. How VirtualBox (or other hypervisors) name interfaces

When you create a **VM in VirtualBox**, each “Adapter” (NAT, Bridged, Host-only, etc.) is exposed inside the guest OS (Ubuntu) as a **network interface** — typically named:

```
enp0s3, enp0s8, enp0s9, etc.
```

The **first adapter** (Adapter 1) almost always appears as **`enp0s3`** inside the VM.
So if Adapter 1 in VirtualBox is configured as **Bridged Adapter**, `enp0s3` is the bridged interface.

---

### 🧩 Step 2. Confirming with your VM’s IP

You ran:

```bash
ip addr show enp0s3
```

and got:

```
inet 192.168.30.60/24 brd 192.168.30.255 scope global enp0s3
```

Your **Remote Desktop network** and **host network** are also on the `192.168.30.0/24` subnet.
➡️ That means `enp0s3` is **connected to the same LAN as your host and remote desktop** — typical of a **Bridged Adapter** connection.

If it were a **NAT adapter**, you’d see a **different subnet**, like `10.0.2.x` (the default NAT range VirtualBox uses).

✅ **Therefore:** The fact that `enp0s3` has `192.168.30.60` — a LAN IP — means it’s bridged to your physical network card.

---

### 🧩 Step 3. How you used it in Docker

You created the Docker macvlan using:

```bash
docker network create -d macvlan \
  --subnet=192.168.40.0/24 \
  --gateway=192.168.40.1 \
  -o parent=enp0s3 \
  plc-macvlan-40
```

The `-o parent=enp0s3` tells Docker:

> “Attach this macvlan network to the same Layer 2 segment as `enp0s3`.”

Since `enp0s3` is your bridged adapter to the real network, each container’s IP (e.g., `192.168.40.201`) appears as a **separate device** on that LAN (from the VM’s perspective).

---

### 🧩 Step 4. How you verified it works

You were able to access PLC containers from outside (your host):

```
http://192.168.30.60:8081/dashboard
```

That’s only possible because:

* `enp0s3` connects your Ubuntu VM directly to the **same physical network** as your host and remote desktop (bridged).
* iptables DNAT forwards traffic from `192.168.30.60` (on enp0s3) → PLC containers (on macvlan).

If `enp0s3` were a NAT adapter, your host could not reach `192.168.30.60:8081` directly.

---

### ✅ Summary

| Interface         | Source                              | Meaning                                                             |
| ----------------- | ----------------------------------- | ------------------------------------------------------------------- |
| `enp0s3`          | First network adapter in VirtualBox | This is the **Bridged Adapter** connected to your LAN               |
| `192.168.30.60`   | IP address assigned to `enp0s3`     | Same subnet as your Remote Desktop and host — confirms it’s bridged |
| Docker parent     | `-o parent=enp0s3`                  | Macvlan attaches to this physical NIC                               |
| `192.168.40.0/24` | macvlan subnet                      | Separate subnet for Docker containers                               |
| iptables rules    | Map 8081–8084 to 192.168.40.201–204 | Enable access from LAN to macvlan network via enp0s3                |

---

### 🧠 TL;DR

You know `enp0s3` is the **bridged adapter** because:

1. It’s the first adapter (VirtualBox convention).
2. It has an IP in your LAN subnet (`192.168.30.x`).
3. Traffic from your host can reach it directly.
4. Your iptables DNAT setup works only because it’s bridged.

---

If you want, I can show you exactly how to **verify this conclusively** with a one-liner using:

```bash
ip route show default
```

Would you like me to include that check?
