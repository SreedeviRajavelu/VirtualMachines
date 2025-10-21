Summarise steps to run the matlab docker container, requirements to run the matlab docker container and how I can check that my license fulfils the requirements. After running the docker container, how can i access the matlab docker container to add simulation files, configure data source, run simulation? Note that I will be running on a remote ubuntu vm that i have ssh-ed into using VPN tunnel and i can only view the browser on my local machine so what options do i have to access and use the matlab docker container and which is best ?


Here’s a **complete, structured summary** of what you need to know and do to run and access a **MATLAB Docker container** — tailored to your setup (remote Ubuntu VM accessed via SSH and VPN, with browser access only on your local machine).

---

## 🧩 1. Requirements to Run MATLAB Docker Container

### ✅ Software Requirements

* **Docker installed** on your **remote Ubuntu VM**.

  * Check: `docker --version`
  * If missing:

    ```bash
    sudo apt update
    sudo apt install docker.io -y
    sudo systemctl enable docker
    sudo systemctl start docker
    ```

* **MATLAB container image** from Docker Hub:

  ```bash
  docker pull mathworks/matlab:r2025a   # version I used on Macbook
  ```

---

### ✅ License Requirements

Your MATLAB license must:

1. **Be valid** for all MathWorks products you want to use.
2. **Be linked** to your MathWorks account.
3. **Be configured for cloud use**.

License types that already support cloud use:

* **Individual** license ✅
* **Campus-Wide** license ✅

For **Concurrent (Network) licenses**:

* You need to specify the license server:

  ```bash
  -e MLM_LICENSE_FILE=27000@MyLicenseServer
  ```

---

### 🔍 How to Check if Your License Meets the Requirements

1. Go to your [MathWorks Account page](https://www.mathworks.com/mwaccount/).
2. Under **Licenses → Manage Licenses**, check:

   * **License type** (Individual, Campus-Wide, or Concurrent).
   * If **Concurrent**, note down the **license server details** (port and DNS address).
3. Confirm with your **license administrator** or MathWorks support if it is **enabled for cloud use**.

---

## ⚙️ 2. Steps to Run the MATLAB Docker Container

### 🧭 Option 1: Run MATLAB in **Browser Mode** (Simpler, Recommended)

Run this on your **remote Ubuntu VM**:

```bash
docker run -it --rm -p 8888:8888 --shm-size=512M mathworks/matlab:r2024b -browser
```

* `-p 8888:8888` → exposes port for browser connection
* `--shm-size=512M` → required for MATLAB GUI stability
* `-browser` → enables web-based MATLAB interface

After running, Docker will print a **URL** like:

```
http://localhost:8888/?token=abcd1234
```

Since your browser is **on your local machine**, you must **forward this port** through SSH.

#### 🛜 SSH Port Forwarding (from Local → Remote)

Run this **on your local machine** (not inside SSH):

```bash
ssh -L 8888:localhost:8888 username@remote_vm_ip
```

Then open in your **local browser**:

```
http://localhost:8888
```

Login with your MathWorks credentials when prompted.

---

### 🧭 Option 2: Run MATLAB in **VNC Mode** (for full desktop access)

Run this on your **remote Ubuntu VM**:

```bash
docker run --init -it --rm -p 5901:5901 -p 6080:6080 --shm-size=512M mathworks/matlab:r2024b -vnc
```

Then:

* Via VNC client: connect to `remote_vm_ip:5901` (password: `matlab`)
* OR via browser (no VNC client): open

  ```
  http://localhost:6080
  ```

  → use SSH port forwarding:

  ```bash
  ssh -L 6080:localhost:6080 username@remote_vm_ip
  ```

---

## 🧰 3. Accessing and Using MATLAB Container

Once the container is running:

### 🗂 Add Simulation Files

Option 1: **Mount a directory** from your remote VM into the container:

```bash
docker run -it --rm \
  -p 8888:8888 \
  -v /path/to/your/files:/home/matlab/projects \
  --shm-size=512M mathworks/matlab:r2024b -browser
```

Inside MATLAB, access your files under `/home/matlab/projects`.

Option 2: **Copy files into container (temporary)**

```bash
docker cp localfile.m container_id:/home/matlab/
```

---

### ⚙️ Configure Data Source

If your simulation depends on external data:

* Mount the data directory the same way using `-v`.
* If using a database or sensor stream, ensure the container has network access (use `--network host` if necessary).

---

### ▶️ Run Simulation

Once inside MATLAB (browser or VNC):

* Open `.slx` or `.m` files from your mounted directory.
* Run simulations normally using the GUI or MATLAB command line.

---

## 🌐 4. Access Options from Your Local Machine

| Access Method                 | Description                                | Pros                               | Cons                              | Recommended When              |
| ----------------------------- | ------------------------------------------ | ---------------------------------- | --------------------------------- | ----------------------------- |
| **Browser Mode** (`-browser`) | Use MATLAB Web interface through port 8888 | Easiest, lightweight, no VNC setup | Slightly limited desktop features | ✅ Best for general use        |
| **VNC Mode** (`-vnc`)         | Full MATLAB desktop via port 6080 or 5901  | Supports toolboxes/add-ons GUI     | Slightly slower over VPN          | When full GUI features needed |
| **Batch Mode** (no GUI)       | Run scripts or simulations directly        | Fastest, low resource use          | No graphical interface            | Automated simulations         |

---

## ✅ Best Setup for Your Case (Remote Ubuntu VM over VPN)

Given that:

* You SSH into the VM.
* You can only use your **local browser** (not remote desktop).

👉 **Recommended Setup:**

1. Use **Browser Mode (`-browser`)**.
2. Use **SSH port forwarding** to map `8888` from remote to local.
3. Mount your project directory via `-v`.
4. Access MATLAB via `http://localhost:8888` in your local browser.

Example final command:

```bash
ssh -L 8888:localhost:8888 user@remote_vm_ip
```

Then on the VM:

```bash
docker run -it --rm \
  -p 8888:8888 \
  -v ~/matlab_projects:/home/matlab/projects \
  --shm-size=512M mathworks/matlab:r2024b -browser
```

Access locally via:
👉 `http://localhost:8888`

---

Would you like me to include steps for **saving MATLAB session data and simulations** persistently (so it’s not lost when the container stops)?


### Question: I need to install the database toolbox, so will it be better to use the browser or vnc option to run and access the matlab docker container

Excellent question — and you’re absolutely right to pause and think about this before running the container.

When you need to **install toolboxes like the Database Toolbox**, the **VNC mode** is the better and more reliable option.

Let’s break down *why* and *how* 👇

---

## 🧩 1. Key Difference Between Browser and VNC Modes

| Mode                          | Interface                                  | Can Install Toolboxes?                                                | When to Use                                                      |
| ----------------------------- | ------------------------------------------ | --------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Browser Mode** (`-browser`) | MATLAB Web Desktop (lightweight interface) | ❌ **No** – the installer UI and Add-On Explorer are **not supported** | When you just want to run existing MATLAB scripts or simulations |
| **VNC Mode** (`-vnc`)         | Full MATLAB Desktop via virtual display    | ✅ **Yes** – supports Toolbox installation, updates, Add-On Explorer   | When you need to install or manage MATLAB toolboxes              |

---

## 🧰 2. How to Run MATLAB in VNC Mode (Recommended for Installing Toolboxes)

Run this on your **remote Ubuntu VM**:

```bash
docker run --init -it --name matlab_container \
  -p 5901:5901 -p 6080:6080 \
  --shm-size=512M \
  mathworks/matlab:r2024b -vnc
```

### Explanation:

* `--init` → handles clean process shutdown
* `-p 5901:5901` → for VNC client connection
* `-p 6080:6080` → for browser-based VNC (no separate client needed)
* `--shm-size=512M` → required shared memory
* `-vnc` → launches MATLAB with a full GUI desktop

Default VNC password: **`matlab`**

---

## 🌐 3. How to Access MATLAB GUI from Your Local Machine

Since you’re on a **remote Ubuntu VM** (accessed via SSH + VPN) and can only view browsers locally:

### ✅ Option 1: Use Browser to Access via Port 6080 (Simplest)

1. On your **local machine**, forward the VNC web port:

   ```bash
   ssh -L 6080:localhost:6080 username@remote_vm_ip
   ```
2. Open in your local browser:

   ```
   http://localhost:6080
   ```
3. Login with password `matlab`.
   You’ll see a full Ubuntu desktop running MATLAB.

→ From there, open **MATLAB → Home → Add-Ons → Get Add-Ons**, search for **Database Toolbox**, and install.

---

### 🧑‍💻 Option 2: Use a VNC Client (TigerVNC, RealVNC, etc.)

1. Forward the VNC port:

   ```bash
   ssh -L 5901:localhost:5901 username@remote_vm_ip
   ```
2. Connect using a VNC viewer to:

   ```
   localhost:5901
   ```
3. Password: `matlab`

This gives a smoother experience (less lag, clipboard support).

---

## 💾 4. Keep the Container Persistent

When installing toolboxes, you’ll definitely want to **keep the container persistent** so your installation isn’t lost:

```bash
docker run --init -it --name matlab_container \
  -p 5901:5901 -p 6080:6080 \
  -v ~/matlab_projects:/home/matlab/projects \
  --shm-size=512M \
  mathworks/matlab:r2024b -vnc
```

Then after installation:

```bash
docker stop matlab_container      # stop container
docker start -ai matlab_container # restart with toolboxes intact
```

---

## 🧠 Summary

| Task                                | Best Mode      | Why                       |
| ----------------------------------- | -------------- | ------------------------- |
| Running scripts/simulations         | **Browser**    | Lighter, easier to access |
| Installing toolboxes / Add-Ons      | **VNC**        | Full desktop needed       |
| Updating MATLAB / managing licenses | **VNC**        | GUI access required       |
| Automated jobs / headless execution | **Batch mode** | No GUI needed             |

---

✅ **Recommendation for you**
Since you need to install the **Database Toolbox**, start MATLAB in **VNC mode** (`-vnc`) and access it through your local browser (port 6080) or a VNC viewer.
Once installed, you can later switch back to **browser mode** (`-browser`) for daily use.

---

Would you like me to show the **exact steps to install the Database Toolbox inside the VNC MATLAB container**, including how to make sure the toolbox persists across container restarts?
