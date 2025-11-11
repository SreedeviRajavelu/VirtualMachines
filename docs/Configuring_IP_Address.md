You **cannot** have two VMs using the **same IP address** on the same network — just like two computers cannot share the same IP.
If you try, you’ll get:

* ARP conflicts
* Connection drops
* One VM unreachable
* Network instability

✅ **Each VM on the bridged network must have its own unique IP.**

---

# ✅ Why both VMs showed the same IP earlier

This means one of the following:

### **1. The new VM was cloned without changing the MAC address**

VirtualBox clones sometimes copy the network interface MAC →
DHCP sees the same MAC → **assigns the same IP.**

### **2. You imported an appliance (.ova) that preserved the old MAC**

The OVA contains the old VM’s MAC address.

✅ Fix: **Give the VM a new MAC**, then it will get a new IP.

---

# ✅ How to fix and get a unique IP for your new VM

### ✅ **Step 1 — Change the MAC Address in VirtualBox**

1. Power off the VM.
2. Go to **VirtualBox → Settings → Network → Adapter 1**
3. Ensure:

   * Attached to: **Bridged Adapter**
4. Click the **refresh icon** next to MAC Address to generate a new one.

Example:
Old MAC: `0800275A3C1D`
New MAC: `080027B9A622`

5. Save and start the VM.

---

# ✅ Step 2 — Inside the VM, ask DHCP for a new IP

Inside the VM:

```bash
sudo dhclient -r
sudo dhclient
```

Then check the new IP:

```bash
ip a | grep 192
```

You should now see **a different IP** such as:

```
192.168.30.61
192.168.30.72
192.168.30.85
```

---

# ✅ Step 3 — SSH into the new IP

From your Mac:

```bash
ssh ubuntu@<new-ip>
```

---

# ✅ Step 4 — (Optional) Make the new IP static

If you want, you can set:

* VM1 → 192.168.30.60
* VM2 → 192.168.30.61

I can guide you through setting static IP if needed.

---

# ✅ Summary

| Issue                 | Solution                         |
| --------------------- | -------------------------------- |
| Both VMs have same IP | Change MAC address in VirtualBox |
| DHPC gives same IP    | Release/renew with `dhclient`    |
| VM unreachable        | Restart VM after MAC change      |

✅ After the MAC change, your VM **will always receive a different unique IP**.

---

If you want, send me:

✅ Your current VirtualBox Network settings screenshot
✅ `ip a` output from both VMs

…and I’ll verify the correct configuration for you.

# Question : if I do the steps above, will the ip address of the VM change each time I log out and log off or reboot the VM ?

Short answer: **No — the IP will NOT keep changing**, as long as you do *one thing*:
✅ **Keep the MAC address fixed after you set it**.

Let me explain clearly:

---

# ✅ After you change the MAC once → DHCP assigns a fixed IP and remembers it

On most networks (home routers, office DHCP), DHCP uses the **MAC address** to give the same IP every time.

So:

### ✅ If the MAC stays the same

Then your VM will usually get **the same IP every reboot**.

Example:
New MAC → 192.168.30.72
Next reboot → DHCP will likely give **192.168.30.72 again**.

### ❌ If you keep changing the MAC

The VM will get **different IPs each time**.

---

# ✅ What each step actually changes

### ✅ Step 1 — Change MAC in VirtualBox

Purpose: ensure the VM gets a *unique* IP different from your other VM.
Once changed, **leave it as is**.

### ✅ Step 2 — dhclient

This only refreshes the IP **once**.
It does not make it dynamic or unstable.

### ✅ Step 3 — SSH into VM

Normal usage.

### ✅ Step 4 — Static IP (optional)

If you want **absolute stability**, assign a static IP:

* VM1 → 192.168.30.60
* VM2 → 192.168.30.61

Then it will *never* change, even if DHCP fails.

---

# ✅ TL;DR — What will happen after reboot?

✅ **IP stays the same**, because DHCP assigns IP based on the VM’s new MAC.
❌ IP will change **only** if:

* you change the MAC again, or
* your network regularly resets its DHCP leases (rare).

---

# ✅ Do you want the IP of the extracted VM to always stay the same?

If yes, I can give you a simple static IP config file for Ubuntu 22.04 (netplan).

