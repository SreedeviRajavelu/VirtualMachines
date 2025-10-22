## Linux System Administration

Perfect, Sree Devi — this is *exactly* the right area to master before your Meta SRE interview. The interviewer will **not just ask you definitions**, but test how you *think, observe, and troubleshoot* in a live Linux environment.

Let’s go step-by-step through the **core Linux system administration concepts** you mentioned — what each is, how it works, and how to use it in interview scenarios 👇

---

## 🧩 1. **Linux Processes**

### 🔹 What They Are

A **process** is an instance of a running program. Every time you run a command or application, Linux creates a process with a **PID (Process ID)**.

Each process has:

* A **parent process (PPID)** — the process that spawned it.
* A **user** who owns it.
* A **state** (running, sleeping, stopped, zombie).
* **CPU/memory usage** info.

---

### 🔹 Key Commands

#### **`ps` — process status**

Shows currently running processes.

```bash
ps aux        # show all processes with detailed info
ps -ef        # alternative format, common in system admin
ps -u sree    # show processes by specific user
```

Example output fields:

```
USER  PID  %CPU  %MEM  VSZ  RSS  TTY  STAT  START  TIME  COMMAND
```

* **%CPU / %MEM** — how much CPU/memory process uses
* **STAT** — process state (R = running, S = sleeping, Z = zombie)

> ✅ *Interview tip:* If asked “how to find what’s consuming high CPU?”, say:
> “I’d run `ps aux --sort=-%cpu | head` or use `top` for live view.”

---

#### **`top` — dynamic view of running processes**

Real-time system monitor (press `q` to quit).

```bash
top
```

Look for:

* **%CPU**, **%MEM** usage by each process
* **load average** (top right): indicates overall system load
* **Tasks** summary (running, sleeping, stopped, zombie)

**Useful keys in top:**

* `P` – sort by CPU
* `M` – sort by memory
* `k` – kill process (enter PID)
* `1` – show per-CPU usage

> ✅ *Scenario example:*
> “Server feels slow — I’d open `top` to check which process spikes CPU or memory, then investigate that PID with `ps -fp <pid>`.”

---

### 🔹 Process Management Commands

```bash
kill <pid>           # terminate a process
kill -9 <pid>        # force kill (SIGKILL)
pkill nginx          # kill all nginx processes
systemctl restart nginx   # restart a service safely
```

---

## ⚙️ 2. **systemd — Service and Process Manager**

### 🔹 What It Is

`systemd` is the init system responsible for **starting and managing services** after boot.

Services are called **units**, and they are defined in files ending with `.service` under `/etc/systemd/system` or `/lib/systemd/system`.

---

### 🔹 Common Commands

```bash
systemctl status nginx        # view service status
systemctl start nginx         # start a service
systemctl stop nginx          # stop a service
systemctl restart nginx       # restart a service
systemctl enable nginx        # enable on boot
systemctl disable nginx       # disable on boot
systemctl list-units --type=service   # list active services
```

> ✅ *Interview tip:*
> “If a web app doesn’t respond, I’d check if its service is active using `systemctl status <service>`, review its logs using `journalctl -u <service>`.”

---

## 🧾 3. **Logs and journalctl**

### 🔹 Where Logs Live

Linux logs are in `/var/log`. Common ones:

```
/var/log/syslog        # system messages (Ubuntu/Debian)
/var/log/messages      # general logs (CentOS/RHEL)
/var/log/auth.log      # login attempts, sudo usage
/var/log/nginx/error.log  # web server logs
```

### 🔹 `journalctl`

Used with `systemd` to view logs.

```bash
journalctl -xe                   # view latest logs with explanations
journalctl -u nginx.service      # view logs for specific service
journalctl --since "1 hour ago"  # filter by time
journalctl -f                    # follow logs in real time (like tail -f)
```

> ✅ *Scenario example:*
> “When debugging a failed service restart, I’d run `systemctl status` and `journalctl -u <service>` to find detailed failure logs.”

---

## 🔐 4. **Permissions and Ownership**

### 🔹 Linux Permission Model

Every file has:

* **Owner**
* **Group**
* **Others**

Permissions:

* **r** – read
* **w** – write
* **x** – execute

### 🔹 Commands

```bash
ls -l                 # view permissions
chmod 755 filename    # set permissions (rwxr-xr-x)
chown user:group file # change ownership
sudo                  # run command as superuser
```

### 🔹 Example

```
-rwxr-xr--  1 sree  devteam  1024 Oct 18 10:00 script.sh
```

→ Owner can read/write/execute
→ Group can read/execute
→ Others can only read

> ✅ *Scenario example:*
> “A script fails with ‘Permission denied’. I’d check permissions with `ls -l` and use `chmod +x script.sh` to make it executable.”

---

## 💻 5. **Resource Monitoring**

### 🔹 Commands

| Command           | Purpose                                             |
| ----------------- | --------------------------------------------------- |
| `top`             | View CPU, memory, and load average in real time     |
| `free -h`         | Check memory usage (RAM, swap)                      |
| `df -h`           | Check disk space usage                              |
| `du -sh /var/log` | Check folder size                                   |
| `uptime`          | Show system uptime and load average                 |
| `vmstat`          | CPU, memory, I/O stats                              |
| `iostat`          | Disk I/O performance                                |
| `sar`             | Historical performance (requires `sysstat` package) |

> ✅ *Scenario:*
> “If a service is lagging, I’d check CPU/memory usage via `top`, disk space via `df -h`, and whether swap usage is high via `free -h`.”

---

## 🔍 6. **Useful Diagnostic Tools**

### **`lsof` — List Open Files**

Shows what files or network sockets a process has open.

```bash
lsof -p <pid>           # files opened by specific process
lsof -i :80             # processes using port 80
```

> ✅ *Use Case:* “Port 8080 is in use — I’d run `lsof -i :8080` to find which process occupies it.”

---

### **`netstat` or `ss` — Network Connections**

Show network sockets and listening ports.

```bash
netstat -tulnp          # list all listening ports with process IDs
ss -tuln                # modern alternative to netstat
```

| Option | Meaning                |
| ------ | ---------------------- |
| `-t`   | TCP connections        |
| `-u`   | UDP connections        |
| `-l`   | Listening ports        |
| `-n`   | Show numeric addresses |
| `-p`   | Show PID/program name  |

> ✅ *Scenario:*
> “To verify if Nginx is listening on port 443, I’d use `ss -tulnp | grep 443`.”

---

## 📊 7. **Load Average**

### 🔹 What It Means

Appears at the top of `top` or via `uptime`:

```
load average: 1.24, 0.87, 0.75
```

These are **average runnable processes over 1, 5, and 15 minutes**.

**Interpretation:**

* If you have 4 CPU cores and a load average of 1.0 → system is under 25% load.
* Load average ≈ number of processes waiting for CPU.
* High load (> number of cores) means CPU is bottlenecked or too many I/O waits.

> ✅ *Scenario example:*
> “If load average stays above 8 on a 4-core server, I’d check CPU utilization in `top` and disk I/O using `iostat` or `vmstat` to see what’s causing the wait.”

---

## 🧠 8. **Interview Demonstration Example**

> **Question:** A user reports the web service is down. What’s your first step?
> **Answer Walkthrough:**
>
> 1. Check if the service is active: `systemctl status nginx`
> 2. If inactive, restart it: `systemctl restart nginx`
> 3. If it fails, view logs: `journalctl -u nginx -xe`
> 4. Verify port 80 is listening: `ss -tulnp | grep 80`
> 5. Check disk space: `df -h`
> 6. Check memory & CPU: `top`
> 7. Confirm no permission or config file issues: `ls -l /etc/nginx/nginx.conf`

That’s the kind of structured, **root-cause-first reasoning** Meta’s Lead SRE will look for.

---

Would you like me to create a **hands-on practice guide** next — with example tasks (like “simulate high CPU”, “analyze logs”, “fix service failure”) and commands you can try on your own Linux VM or WSL?
That would help you *practice interactively* before your interview.


### Explain the use and how to use each of the commands you mentioned here

✅ Answer Outline: Check system resource usage (top, free -m, iostat, vmstat). Inspect web server logs (/var/log/nginx/access.log, /var/log/syslog). Test connectivity (ping, curl, traceroute) and DNS (dig, nslookup). Use netstat -tulpen or ss to check open ports and connections. If containerized, check container CPU/memory with docker stats. Isolate if it’s network latency, disk I/O, CPU load, or app issue.


Excellent follow-up, Sree Devi 👏 — these commands are exactly what an SRE at Meta (or anywhere) uses **daily** to troubleshoot production systems.

Let’s go **step-by-step through each command**, with:

* ✅ *What it does*
* ⚙️ *How to use it (with examples)*
* 🧠 *How to interpret output in an interview scenario*

---

## 🧩 1. **System Resource Usage Commands**

---

### **`top`** — Monitor processes and resource usage in real time

✅ **Purpose:** See which processes are consuming CPU, memory, and how the system load changes dynamically.

```bash
top
```

**Key output fields:**

* **%CPU, %MEM** → show process resource usage
* **load average:** → number of runnable processes (system load)
* **Tasks:** → running/sleeping/zombie processes

**Interactive keys:**

* `P` → sort by CPU
* `M` → sort by memory
* `k` → kill a process (enter PID)
* `1` → show per-CPU stats

**Example:**
If the system feels slow, look for a process consuming >90% CPU or memory.
You might say in the interview:

> “I’d run `top` to check for processes consuming high CPU or memory and use `k` to kill runaway ones.”

---

### **`free -m`** — Show memory usage

✅ **Purpose:** Display total, used, and available RAM.

```bash
free -m
```

* `-m` → show memory in megabytes (`-h` for human-readable)

**Output example:**

```
              total    used    free   shared  buff/cache  available
Mem:           7972    2564    1504      120        3904       5120
Swap:          2048       0    2048
```

**Interpretation:**

* `used` → memory actively in use
* `buff/cache` → memory used for caching (reclaimable)
* `available` → memory apps can still use

> ✅ “If memory usage is near 100% and swap is active, it’s likely a memory leak or overloaded system.”

---

### **`iostat`** — Monitor disk I/O (input/output performance)

✅ **Purpose:** Identify I/O bottlenecks — slow disks or too many reads/writes.

Install (if missing):

```bash
sudo apt install sysstat
```

Run:

```bash
iostat -x 2 3
```

* `-x` → extended stats
* `2 3` → update every 2 seconds, 3 times

**Key columns:**

* `%util` → how busy the disk is (close to 100% = bottleneck)
* `r/s`, `w/s` → read/write requests per second
* `await` → average wait time per I/O request

> ✅ “If `%util` > 90%, the disk is saturated — I’d check which process is writing heavily using `iotop` or `lsof`.”

---

### **`vmstat`** — System performance overview (CPU, memory, I/O)

✅ **Purpose:** Gives a quick view of CPU usage, swap activity, and system processes.

```bash
vmstat 2 5
```

* Every 2 seconds, show stats 5 times.

**Key columns:**

* `r` → runnable processes
* `b` → processes in uninterruptible sleep (I/O wait)
* `si/so` → swap in/out (should be 0 ideally)
* `us`/`sy` → CPU time spent in user/system mode
* `wa` → time spent waiting on I/O

> ✅ “If I see high `wa` (I/O wait) in `vmstat`, I know disk I/O is the bottleneck, not CPU.”

---

## 🧾 2. **Inspect Logs**

---

### **Check web server and system logs**

**Default paths:**

```bash
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/syslog
```

**Common commands:**

```bash
tail -n 50 /var/log/nginx/error.log      # show last 50 lines
tail -f /var/log/nginx/access.log        # live log updates
grep "500" /var/log/nginx/access.log     # filter HTTP 500 errors
less /var/log/syslog                     # browse system logs
```

> ✅ *Interview answer example:*
> “If the app returns 500 errors, I’d check `/var/log/nginx/error.log` and use `grep` to filter out error patterns.”

---

## 🌐 3. **Test Connectivity and DNS**

---

### **`ping`** — Check if a host is reachable

✅ **Purpose:** Test basic network connectivity using ICMP.

```bash
ping google.com
```

* Sends ICMP echo requests; reports response time and packet loss.
* `Ctrl + C` to stop.

**Output example:**

```
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=14.3 ms
```

> ✅ “If ping fails, it could be a DNS issue, routing problem, or firewall blocking ICMP.”

---

### **`curl`** — Test HTTP(S) connectivity

✅ **Purpose:** Simulate web requests and see responses (great for web app debugging).

```bash
curl -I https://example.com
```

* `-I` → fetch headers only
* `-v` → verbose mode
* `-L` → follow redirects

**Example:**

```bash
curl -v http://localhost:8080
```

> ✅ “If the app returns 500, I’d curl the endpoint to check if backend is reachable or misconfigured.”

---

### **`traceroute`** — Trace network path to a destination

✅ **Purpose:** Identify where packets are delayed or dropped along a route.

```bash
sudo apt install traceroute
traceroute google.com
```

Shows each router hop with latency times.

**Interpretation:**
If delay starts increasing after a certain hop, that’s where the problem lies (network congestion or bad route).

> ✅ “If latency is high, I’d use `traceroute` to pinpoint where in the path the delay starts.”

---

### **`dig`** — DNS lookup utility

✅ **Purpose:** Query DNS servers to resolve domain names.

```bash
dig google.com
```

**Output fields:**

* **ANSWER SECTION** → IP address (A or AAAA record)
* **Query time:** → how long it took
* **SERVER:** → which DNS server replied

**Example:**

```bash
dig @8.8.8.8 example.com +short
```

> ✅ “If DNS seems broken, I’d run `dig` to see if the domain resolves and which DNS server responds.”

---

### **`nslookup`** — Simpler DNS tool

✅ **Purpose:** Same as `dig` but simpler format.

```bash
nslookup google.com
```

**Output:**

```
Server:  8.8.8.8
Address: 8.8.8.8#53
Name:    google.com
Address: 142.250.72.206
```

> ✅ “I use `nslookup` or `dig` to confirm if DNS is returning the right IP before suspecting network issues.”

---

## 🔒 4. **Check Open Ports and Active Connections**

---

### **`netstat -tulpen`** — Show active network ports and processes

✅ **Purpose:** See which processes are listening on which ports.

```bash
sudo netstat -tulpen
```

| Option | Description             |
| ------ | ----------------------- |
| `-t`   | TCP connections         |
| `-u`   | UDP connections         |
| `-l`   | Listening sockets       |
| `-p`   | Show PID/program        |
| `-e`   | Extended info           |
| `-n`   | Don’t resolve hostnames |

**Output example:**

```
Proto Recv-Q Send-Q Local Address  Foreign Address  State  PID/Program name
tcp   0      0 0.0.0.0:80         0.0.0.0:*        LISTEN 1234/nginx
```

> ✅ “To verify if Nginx is running, I’d use `netstat -tulpen | grep 80`.”

---

### **`ss`** — Modern replacement for netstat

✅ **Purpose:** Faster, newer way to view sockets.

```bash
sudo ss -tulnp
```

Shows all listening ports and which service owns them.

> ✅ “I’d use `ss -tulnp` instead of `netstat` on modern systems — it’s faster and doesn’t require extra packages.”

---

## 🐳 5. **Container Resource Monitoring**

---

### **`docker stats`** — Monitor container CPU/memory usage

✅ **Purpose:** Real-time resource stats for running containers.

```bash
docker stats
```

**Output example:**

```
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT
a1b2c3d4e5f6   nginx_web    2.35%     120MiB / 2GiB
b7c8d9e0f1g2   db_service   10.52%    450MiB / 4GiB
```

> ✅ “If a containerized app is slow, I’d check `docker stats` to see if a container is hitting its resource limits.”

---

### **Bonus Commands for Containers**

```bash
docker ps                       # list running containers
docker logs <container_id>       # view container logs
docker inspect <container_id>    # show container config/network info
docker exec -it <container_id> bash  # open shell inside container
```

---

## 🧠 6. **Putting It All Together — Interview Scenario Example**

> **Scenario:** Users report your web app is slow.

### Your investigation flow (and what to say in interview):

1. **Check server load:**
   `top` → CPU/memory usage
   `free -m` → memory status
   `iostat` / `vmstat` → I/O bottlenecks

2. **Check logs:**
   `tail -f /var/log/nginx/error.log`
   `grep "500" /var/log/nginx/access.log`

3. **Test connectivity:**
   `ping app-server` → reachability
   `curl -I http://localhost:8080` → check app response
   `traceroute` → trace route if remote latency suspected

4. **Check DNS resolution:**
   `dig app.company.com +short`

5. **Verify open ports:**
   `ss -tulnp | grep 8080` → ensure app is listening

6. **If containerized:**
   `docker stats` → check resource use
   `docker logs <id>` → application errors

> ✅ *What the interviewer sees:* You follow a logical, layered troubleshooting process — from system → network → app → container — just like a real SRE.

---

Would you like me to create a **practice lab script** (like a guided “break and fix” Linux exercise) where you simulate an outage and use these commands to diagnose and fix it?
That kind of hands-on simulation would prepare you *exactly* for the troubleshooting part of the Meta SRE interview.


top vs htop 