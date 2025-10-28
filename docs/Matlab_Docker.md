Summarise steps to run the matlab docker container, requirements to run the matlab docker container and how I can check that my license fulfils the requirements. After running the docker container, how can i access the matlab docker container to add simulation files, configure data source, run simulation? Note that I will be running on a remote ubuntu vm that i have ssh-ed into using VPN tunnel and i can only view the browser on my local machine so what options do i have to access and use the matlab docker container and which is best ?


Here’s a **complete, structured summary** of what you need to know and do to run and access a **MATLAB Docker container** — tailored to your setup (remote Ubuntu VM accessed via SSH and VPN, with browser access only on your local machine).

---

## 🧩 1. Requirements to Run MATLAB Docker Container

### ✅ Software Requirements

* **Docker installed** on your **remote Ubuntu VM**.

  * Check: `docker --version`
  * If missing:

    ```
    sudo apt update

    sudo apt install docker.io -y

    sudo systemctl enable docker

    sudo systemctl start docker
    ```

* **MATLAB container image** from Docker Hub:

  ```
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

  ```
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

```
docker run -it --rm -p 8888:8888 --shm-size=512M mathworks/matlab:r2025a -browser
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

```
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

```
docker run --init -it --rm -p 5901:5901 -p 6080:6080 --shm-size=512M mathworks/matlab:r2024b -vnc
```

Then:

* Via VNC client: connect to `remote_vm_ip:5901` (password: `matlab`)
* OR via browser (no VNC client): open

  ```
  http://localhost:6080
  ```

  → use SSH port forwarding:

  ```
  ssh -L 6080:localhost:6080 username@remote_vm_ip
  ```

---

## 🧰 3. Accessing and Using MATLAB Container

Once the container is running:

### 🗂 Add Simulation Files

Option 1: **Mount a directory** from your remote VM into the container:

```
docker run -it --rm \
  -p 8888:8888 \
  -v /path/to/your/files:/home/matlab/projects \
  --shm-size=512M mathworks/matlab:r2024b -browser
```

Inside MATLAB, access your files under `/home/matlab/projects`.

Option 2: **Copy files into container (temporary)**

```
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

```
ssh -L 8888:localhost:8888 user@remote_vm_ip
```

Then on the VM:

```
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

```
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

   ```
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

   ```
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

```
docker run --init -it --name matlab_container -p 5901:5901 -p 6080:6080 -v ~/matlab_projects:/home/matlab/projects --shm-size=512M mathworks/matlab:r2025a -vnc
```

Then after installation:

```
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


### Question

explain each part of the command below and what they do

docker run --init -it --name matlab_container -p 5901:5901 -p 6080:6080 -v ~/matlab_projects:/home/matlab/projects --shm-size=512M mathworks/matlab:r2025a -vnc

Perfect — let’s go through your MATLAB Docker command **line by line**, so you know exactly what each flag, option, and parameter does.

---

## 🧩 Full Command

```bash
docker run --init -it --name matlab_container \
  -p 5901:5901 -p 6080:6080 \
  -v ~/matlab_projects:/home/matlab/projects \
  --shm-size=512M \
  mathworks/matlab:r2025a -vnc
```

---

## 🧠 Step-by-Step Breakdown

### 🔹 `docker run`

This is the base Docker command that **creates and starts a new container** from an image.

Think of it like “booting up a virtual MATLAB machine” inside your remote Ubuntu VM.

---

### 🔹 `--init`

* Runs a lightweight **init process (tini)** inside the container.
* This small helper process:

  * Properly **handles zombie processes**.
  * Ensures **clean shutdowns** when you stop the container.
* Especially useful for long-running containers or ones that launch GUI processes (like MATLAB).

✅ Good practice — prevents lingering or orphaned processes.

---

### 🔹 `-it`

Combines two flags:

| Flag | Meaning                                                                                    |
| ---- | ------------------------------------------------------------------------------------------ |
| `-i` | **Interactive** mode – keeps STDIN open so you can interact with MATLAB in the terminal.   |
| `-t` | Allocates a **pseudo-TTY** – gives you a readable terminal interface (text-based console). |

Together, `-it` lets you see MATLAB logs or interact with the command-line interface if needed.

---

### 🔹 `--name matlab_container`

* Assigns a **custom name** (`matlab_container`) to the container.
* Makes management easier:

  * Stop container: `docker stop matlab_container`
  * Restart container: `docker start -ai matlab_container`
  * Inspect container: `docker inspect matlab_container`

If you omit this, Docker assigns a random name like `sleepy_fermi`.

---

### 🔹 `-p 5901:5901`

* Maps **port 5901** from your **host (Ubuntu VM)** to the same port inside the **container**.
* This is the standard port for **VNC (Virtual Network Computing)**.
* Lets you connect to MATLAB’s virtual desktop via a VNC client (e.g., RealVNC, TigerVNC).

🖥️ Example:
Connect from your local machine via SSH tunnel:

```bash
ssh -L 5901:localhost:5901 user@remote_vm_ip
```

Then open your VNC viewer → connect to `localhost:5901`.

---

### 🔹 `-p 6080:6080`

* Exposes port **6080**, which is used for **browser-based VNC access** (no client needed).
* Lets you access the MATLAB desktop using a **web browser**.

🌐 Example:
After SSH port forwarding:

```bash
ssh -L 6080:localhost:6080 user@remote_vm_ip
```

Open in your browser:

```
http://localhost:6080
```

Default password: `matlab`

---

### 🔹 `-v ~/matlab_projects:/home/matlab/projects`

* Mounts a **host directory** (`~/matlab_projects` on your Ubuntu VM) into the **container** directory `/home/matlab/projects`.

| Side                    | Meaning                                             |
| ----------------------- | --------------------------------------------------- |
| `~/matlab_projects`     | Folder on your Ubuntu host (where your files live). |
| `/home/matlab/projects` | Folder *inside* the container, visible to MATLAB.   |

This allows:

* MATLAB inside the container to **read/write files** that persist outside the container.
* You to edit `.m` or `.slx` files from the host and see them instantly inside MATLAB.

✅ **Critical for persistence** — your files survive even if you delete or recreate the container.

---

### 🔹 `--shm-size=512M`

* Allocates **512 MB of shared memory** to the container.
* MATLAB’s graphical interface (and Simulink) require a certain amount of shared memory to run reliably.
* Without this, you may get errors like “Out of memory” or crashes when running GUI or simulations.

You can increase it if needed:

```bash
--shm-size=1G
```

---

### 🔹 `mathworks/matlab:r2025a`

* Specifies the **Docker image** to use.
* `mathworks/matlab` → the official MATLAB image from Docker Hub.
* `:r2025a` → the specific MATLAB release tag (e.g., R2025a).

If not present locally, Docker will **pull** (download) it automatically from Docker Hub.

---

### 🔹 `-vnc`

* Argument passed **to MATLAB inside the container** (not a Docker flag).
* Tells MATLAB to **start in VNC desktop mode**, not browser mode or batch mode.
* This launches a full graphical desktop environment (Ubuntu + MATLAB GUI).

So, inside the container, the `matlab` command effectively runs:

```
matlab -vnc
```

---

## 🧾 Summary Table

| Part                                         | Function                                |
| -------------------------------------------- | --------------------------------------- |
| `docker run`                                 | Start a new container                   |
| `--init`                                     | Adds a small init process for stability |
| `-it`                                        | Run interactively with a terminal       |
| `--name matlab_container`                    | Give the container a custom name        |
| `-p 5901:5901`                               | Expose VNC port for desktop access      |
| `-p 6080:6080`                               | Expose web-VNC port for browser access  |
| `-v ~/matlab_projects:/home/matlab/projects` | Mount host directory into container     |
| `--shm-size=512M`                            | Allocate shared memory for MATLAB GUI   |
| `mathworks/matlab:r2025a`                    | MATLAB Docker image and version         |
| `-vnc`                                       | Start MATLAB in VNC (desktop) mode      |

---

Would you like me to show the **complementary commands** to:

* stop/start this container safely, and
* connect to it from your **local browser** step-by-step via SSH port forwarding?


#### Does the folder have to be created before running the command above:

🧩 The part in question
-v ~/matlab_projects:/home/matlab/projects


is a bind mount, meaning:

The left side (~/matlab_projects) is a folder on your host (your remote Ubuntu VM).

The right side (/home/matlab/projects) is the location inside the container where that folder will appear.


## Database Toolbox - Persist Database Toolbox across Container Restarts 

Excellent — this is exactly the right question to ask 👍 because by default, **MATLAB containers are ephemeral**, meaning any toolbox installed inside them **disappears when the container stops** unless you persist it properly.

Let’s go through all the ways to make the **Database Toolbox installation persistent**, and then I’ll recommend **the best approach for your remote Ubuntu VM setup.**

---

## 🧩 Why toolboxes disappear

When you install the Database Toolbox from the MATLAB GUI inside a container:

* It gets installed to a path inside the container’s filesystem (typically `/usr/local/MATLAB/R2025a/toolbox`).
* That directory **is part of the container image layer**, not your mounted volume.
* So when you stop or remove the container, those changes are lost.

To make toolboxes *persistent*, you have two main options:

---

## ✅ **Option 1: Create a new custom Docker image with Database Toolbox preinstalled**

This is the **recommended and professional method**.

It ensures:

* The Database Toolbox is installed permanently inside the image.
* You can restart or recreate containers anytime — the toolbox remains.
* The setup is reproducible for others or future VMs.

---

### **Steps**

#### 1️⃣ Create a `Dockerfile` like this:

```dockerfile
FROM mathworks/matlab:r2025a

# Install Database Toolbox inside MATLAB
RUN matlab -batch "matlab.addons.installToolbox('/home/matlab/Database_Toolbox.mltbx'); exit"

# Optional: Set default working directory
WORKDIR /home/matlab/projects
```

#### 2️⃣ Place the `.mltbx` file inside the same directory as the Dockerfile.

You can download the `Database_Toolbox.mltbx` file from MathWorks (via Add-Ons or File Exchange).
If you already installed it once, MATLAB’s Add-On Manager might have cached it in:

```
~/.matlab/Add-Ons/Downloads/
```

---

#### 3️⃣ Build your custom image

Run on your remote Ubuntu VM:

```bash
docker build -t matlab-with-dbtoolbox .
```

This will:

* Pull the official MATLAB image.
* Install the Database Toolbox inside it.
* Create a new image named `matlab-with-dbtoolbox`.

---

#### 4️⃣ Run your container from the new image

```bash
docker run --init -it --name matlab_container \
  -p 5901:5901 -p 6080:6080 \
  -v ~/matlab_projects:/home/matlab/projects \
  --shm-size=512M \
  matlab-with-dbtoolbox -vnc
```

✅ Now every time you start or restart the container from this image, **Database Toolbox will already be installed.**

---

## ✅ **Option 2: Use a persistent volume for the MATLAB installation directory**

If you don’t want to build a new image, you can mount the MATLAB installation directory to a **Docker volume** or **bind mount** so that installed toolboxes remain there between runs.

---

### Steps

1️⃣ First, find where MATLAB installs toolboxes inside the container:
It’s usually under:

```
/usr/local/MATLAB/R2025a/toolbox
```

2️⃣ Create a directory on your host for MATLAB installation persistence:

```bash
mkdir -p ~/matlab_installation
```

3️⃣ Run the container with that directory mounted:

```bash
docker run --init -it --name matlab_container \
  -p 5901:5901 -p 6080:6080 \
  -v ~/matlab_projects:/home/matlab/projects \
  -v ~/matlab_installation:/usr/local/MATLAB/R2025a/toolbox \
  --shm-size=512M \
  mathworks/matlab:r2025a -vnc
```

4️⃣ Now when you install the Database Toolbox (via GUI or script), it’s saved in your host directory `~/matlab_installation`, and it will persist between restarts.

⚠️ However: MATLAB itself might not have permission to modify `/usr/local` as a non-root user, so you might need to run MATLAB as root the first time for installation:

```bash
sudo matlab &
```

---

## ✅ **Option 3: Commit your running container to a new image**

If you already have a container where you successfully installed the Database Toolbox once, you can **turn that running container into a new image** that keeps everything as-is.

---

### Steps

1️⃣ After successfully installing the Database Toolbox, check your running container name or ID:

```bash
docker ps
```

2️⃣ Commit the container as a new image:

```bash
docker commit matlab_container matlab-with-dbtoolbox
```

3️⃣ Verify:

```bash
docker images
```

You should now see:

```
matlab-with-dbtoolbox   latest   <image_id>   ...
```

4️⃣ Next time, run containers from that image:

```bash
docker run --init -it --name matlab_container \
  -p 5901:5901 -p 6080:6080 \
  -v ~/matlab_projects:/home/matlab/projects \
  --shm-size=512M \
  matlab-with-dbtoolbox -vnc
```

✅ Now Database Toolbox persists because it’s baked into the new image layer.

---

## 💡 Recommended Approach for You

Since you’re using a **remote Ubuntu VM** and want a clean, reusable setup:

👉 **Best Choice:**
**Option 3 — Commit your configured container to a new image.**

It’s fast, doesn’t require a new Dockerfile, and keeps everything you just installed.

---

### ✅ Summary Table

| Option                       | Description                                | Toolbox Persists?  | Complexity | Recommended For                      |
| ---------------------------- | ------------------------------------------ | ------------------ | ---------- | ------------------------------------ |
| 1. Custom Image (Dockerfile) | Install via script during image build      | ✅ Permanent        | Medium     | Reusable environments                |
| 2. Persistent Volume         | Mount `/usr/local/MATLAB/R2025a/toolbox`   | ✅ Between restarts | Medium     | Manual installs                      |
| 3. Commit Container          | Save your current container as a new image | ✅ Permanent        | 🔹Easy     | One-time setup (recommended for you) |

---

Would you like me to give you **the exact commands for Option 3 (commit method)** so you can make your current container persist the Database Toolbox installation immediately after you reinstall it?




## Installing Database Toolbox on MATLAB Docker container

Excellent — that tells us the Database Toolbox files you copied over are **not fully functional** by themselves (they lack the compiled components and metadata MATLAB uses to register licensed toolboxes).

Let’s clarify the situation, then go over **two reliable ways to install the Database Toolbox** in your MATLAB Docker container so it actually works and persists.

---

## 🚨 Why your current copy doesn’t work

The folder `/Applications/MATLAB_R2025a.app/toolbox/database/` on macOS contains **only the high-level MATLAB scripts and doc files**, but the actual toolbox relies on:

* Internal compiled libraries (`.p`, `.mex*`, `.jar`, `.xml`)
* License registration info stored in MATLAB’s internal database
* Installation metadata from the **Add-On Manager**

That’s why copying just the folder doesn’t make MATLAB recognize it — it’s not a standalone package.

---

## ✅ You have two correct ways to install it

### **Option 1 — Use the MATLAB Add-On Installer inside the container (recommended if license allows it)**

This is the **official** way and ensures proper registration and license validation.

#### Steps:

1. **Run MATLAB Docker container with VNC:**

   ```bash
   docker start -ai matlab_container
   ```

   or if not created:

   ```bash
   docker run --init -it --name matlab_container -p 5901:5901 -p 6080:6080 \
   -v ~/matlab_projects:/home/matlab/projects \
   --shm-size=512M mathworks/matlab:r2025a -vnc
   ```

2. **Access GUI:**
   Go to your browser and open:

   ```
   http://<your_vm_ip>:6080
   ```

   (Default password: `matlab`)

3. **Install from MATLAB GUI:**

   * In MATLAB, go to **Home → Add-Ons → Get Add-Ons**
   * Search for **Database Toolbox**
   * Click **Install**

4. MATLAB may restart automatically after installation — **that’s normal**.
   If it doesn’t reopen, restart manually:

   ```bash
   matlab &
   ```

5. After restart, verify:

   ```matlab
   ver
   which database
   ```

✅ If successful, MATLAB will now have Database Toolbox **registered properly**, and it will persist inside the container (since installation writes to `/usr/local/MATLAB`).

---

### **Option 2 — Download the `.mltbx` file on your Mac and transfer it**

If the Add-On GUI fails (e.g., no internet access inside container), you can manually download and install the toolbox.

#### Steps:

1. On your **Mac**, open MATLAB → Home → Add-Ons → Manage Add-Ons.
   Find **Database Toolbox**, click **View Details**, and note the installation source path.

   Or go to:

   ```
   https://www.mathworks.com/matlabcentral/fileexchange/
   ```

   (or the Add-On Explorer web link for Database Toolbox if you have MathWorks access)

2. Download the **Database Toolbox .mltbx** installer file.

3. Copy it to your Ubuntu VM:

   ```bash
   scp /path/to/Database_Toolbox.mltbx ubuntu@<vm_ip>:/home/ubuntu/
   ```

4. Copy it into the container:

   ```bash
   docker cp /home/ubuntu/Database_Toolbox.mltbx matlab_container:/home/matlab/
   ```

5. Access the container:

   ```bash
   docker exec -it matlab_container bash
   ```

6. Run MATLAB inside:

   ```bash
   matlab -batch "matlab.addons.install('/home/matlab/Database_Toolbox.mltbx');"
   ```

7. Then verify:

   ```bash
   matlab -batch "ver"
   ```

✅ This method installs the toolbox properly, registers it with MATLAB’s license manager, and persists across container restarts.

---

## 🧠 Summary

| Method                     | Description                                  | Requires Internet in Container | Persists |
| -------------------------- | -------------------------------------------- | ------------------------------ | -------- |
| **Add-On GUI**             | Install Database Toolbox from MATLAB Add-Ons | ✅ Yes                          | ✅ Yes    |
| **.mltbx Manual Install**  | Install from local `.mltbx` file             | ❌ No                           | ✅ Yes    |
| **Copying toolbox folder** | What you tried (won’t register toolbox)      | N/A                            | ❌ No     |

---

👉 Since you already have the container running with GUI access, I recommend retrying **Option 1 (Add-On GUI)**.
If it fails again (e.g., after restart the toolbox doesn’t show up), proceed with **Option 2** — I can guide you to get the `.mltbx` file from your Mac MATLAB installation directory if needed.

Would you like me to show you exactly where to find the `.mltbx` file on your Mac?
