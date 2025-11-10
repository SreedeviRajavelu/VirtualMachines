## What is a Volume in Docker

Think of Docker container as a small isolated computer - when you stop or delete it, everything inside disappears (its 'disk' is temporary).

A Docker volume is like pluggin in an external hard drive or shared folder to that container.

It lets you:

1. **Keep data** even after container is stopped or removed.
2. **Share files** between real machine (the host) and the container.

There are 2 main types of volume mounts:

- Bind mount (links a folder from your host machine to the container)
- Named volume (managed entirely by Docker - stored somewhere under `/var/lib/docker/volumes`)

Explanation for use of volumes:

- Left side (`./sql`) -> a folder on your computer at the same level as your docker-compose.yml file
- Right side (`/docker-entrypoint-initdb.d`) -> a special folder *inside the container* that MySQL checks when it first starts.

When MySQL starts for the first time, it looks in `/docker-entrypoint-initdb.d` and automatically runs any `.sql` files it finds there - this is how it initializes your database.

When Docker starts the MySQL container, it copies that folder into the container.

MySQL executes your CCP_data.sql automatically - creating the tables, inserting initial data, etc.

This mount does not store your database data (the rows & tables you later insert). It only gives MySQL initialization scripts.

Once MySQL is running, the database data is stored inside the container's temporary filesystem. If you delete the container, the data is lost.

This is called a **bind mount** (it directly links your folder `./sql` from the host to the continer).

Question:  does the special folder **docker-entrypoint-initdb.d** exist in every docker container ? if not where does it come from ?

No, the folder **/docker-entrypoint-initdb.d** does not exist in every Docker container.
It exists only in **certain official** database images (like MySQL, PostgreSQL, MongoDB, etc.) because the image creator chose to include it in the image’s startup logic. 

Each image defines **its own conventions** about where to put configuration or initialization files.

So when you use:
```
    volumes:
      - ./sql:/docker-entrypoint-initdb.d
```
you are not telling Docker anything special - you are telling the MySQL container:
When you run your startup script, here is the folder where you can find initialization files.



## Content of docker-compose.yml file to use postgresql database as docker container 

This version does not persist volume and data will vanish if you remove the container.


```
services:
  postgres:
    image: postgres:16
    container_name: ccp-multi-ied-multi-plc
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: root
      POSTGRES_DB: powerplant
    ports:
      - "5432:5432"
    volumes:
    - ./postgres-sql:/docker-entrypoint-initdb.d

```

Use of 
**- volumes:**
  **- ./sql:/docker-entrypoint-initdb.d**

- This mounts your host directory ./sql into the container’s /docker-entrypoint-initdb.d directory.

- docker-entrypoint-initdb.d is a special folder: on first startup of the container, MySQL runs all .sql or .sh scripts there to initialize the DB.

- After initialization, new runs don’t automatically re-run scripts unless you remove the container and volume.

❗ This is not where live database data is stored. Live data is in /var/lib/mysql. Since you’re not mounting that, the data lives in a Docker-managed anonymous volume.

So: Changing this bind mount will not remove your MySQL data. It only affects which init scripts would run on container creation.



## Content of docker-compose.yml file to use postgresql database as docker container (with persistent volume)
```
services:
  postgres:
    image: postgres:16
    container_name: ccp-multi-ied-multi-plc-pg
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: root
      POSTGRES_DB: powerplant
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data   # <-- persistent volume for DB data
      - ./postgres-sql:/docker-entrypoint-initdb.d  # <-- init scripts

volumes:
  postgres_data:   # <-- named volume definition
```


### Use of Volumes inside PostgreSQL docker-compose.yml

```
volumes:
- postgres_data:/var/lib/postgresql/data # persistent volume for DB data
- ./postgres-sql:/docker-entrypoint-initdb.d # init scripts

volumes:
  postgres_data: # named volume definition 
```

Explanation of each line:

1. `-postgres_data:/var/lib/postgresql/data`

- left side (postgres_data) -> a named volume, managed by Docker

 - you don't see it as a folder on your host - Docker stores it internally (under /var/lib/docker/volumes/...)

- right side (/var/lib/postgresql/data) -> the main folder inside the container where the PostgreSQL stores all its databases.

This means your PostgreSQL data (tables, records, etc.) is stored in the `postgres_data` volume not inside the temporary cotainer filesystem.

Even if you:

- stop the container (docker compose down)

- rebuild it

- restart it

the data remains intact, because the volume is persistent.

If you ever want to delete the data, you explicitly run:
`docker compose down -v`

2. `- postgres-sql:/docker-entrypoint-initdb.d`

- it mounts your host folder `postgres-sql/` (next to your `docker-compose.yml`) into the container's `/docker-entrypoint-initdb.d` folder

- PostgreSQL automatically runs any `.sql` files there the first time the database is initialized (before any data exists).

So if you have `postgres-sql/CCP_data.sql`, it will create your tables and insert the initial values when the container is started for the first time.

3. section at the bottom:
```
volumes:
  postgres_data:
```

This **declares** the named volume used above, so Docker knows it should create and manage it.


# Question: how can I ensure that these 3 containers always stay running ? before I stop and remove the containers how can I verify whether the data in the containers are persisted ? ubuntu@pgt:~$ docker ps | grep ccp d38cf688db43 postgres:16 "docker-entrypoint.s…" 2 weeks ago Up 37 minutes 0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp ccp-multi-ied-multi-plc-pg ubuntu@pgt:~$ docker ps | grep scada ubuntu@pgt:~$ docker ps -a | grep scada 017a83b809c7 scadalts/scadalts:latest "catalina.sh run" 2 days ago Exited (143) 45 minutes ago scadalts ubuntu@pgt:~$ docker ps -a | grep mysql 8ddf0f1a81bf mysql/mysql-server:5.7 "/entrypoint.sh mysq…" 2 days ago Exited (0) 45 minutes ago mysql ubuntu@pgt:~$

Here is the **safe and correct workflow** to ensure your 3 containers always stay running **WITHOUT LOSING DATA**.

We will do it in 3 steps:

✅ **Step 1 — Check if your containers already use volumes**
✅ **Step 2 — Back up data (if needed)**
✅ **Step 3 — Recreate with restart policies (`--restart unless-stopped`)**

---

# ✅ Step 1 — Check whether your containers use volumes

Before removing anything, you MUST confirm that:

✅ the data lives in Docker volumes (safe)
❌ or the data is stored inside the container filesystem (unsafe — will be lost if container is deleted)

---

## ✅ 1.1 Check volumes for each container

### **ScadaLTS**

```bash
docker inspect scadalts | grep -i mount -A 10
```

### **MySQL 5.7**

```bash
docker inspect mysql | grep -i mount -A 10
```

### **MATLAB container**

```bash
docker inspect matlab_container | grep -i mount -A 10
```

---

## ✅ What you want to see:

Example (this is good):

```
"Mounts": [
    {
        "Type": "volume",
        "Name": "mysql_data",
        "Destination": "/var/lib/mysql"
    }
]
```

If you see `"Type": "volume"` → **your data is safe**.
If instead you see `"Type": "bind"` → data is also safe (stored on host).
If you see **empty Mounts:** → dangerous, all data is inside the container and will be lost if removed.

Let me know what your actual outputs are.

---

# ✅ Step 2 — Back up the data (if your volumes are missing)

If Mounts is empty, you must back up first.

### ✅ MySQL backup

```bash
docker start mysql
docker exec -it mysql mysqldump -u root -p --all-databases > mysql_backup.sql
```

### ✅ ScadaLTS backup

It stores data inside:

* `/usr/local/tomcat/webapps/Scada-LTS/WEB-INF/classes`
* `/opt/scadalts` depending on version

You can copy the data out:

```bash
docker cp scadalts:/usr/local/tomcat/webapps/Scada-LTS ./scadalts_backup
```

### ✅ MATLAB container

Only needed if you stored data inside the container:

```bash
docker cp matlab_container:/home/matlab ./matlab_backup
```

---

# ✅ Step 3 — Make containers **always running**

Once volume safety is confirmed, you can recreate the containers with:

✅ auto-restart
✅ persistent data volumes
✅ correct ports

---

# ✅ 3.1 Apply restart policies without recreating (if possible)

You can add restart policies **even to existing containers**:

```bash
docker update --restart unless-stopped scadalts
docker update --restart unless-stopped mysql
docker update --restart unless-stopped matlab_container
```

### ✅ Check they’re applied

```bash
docker inspect scadalts | grep RestartPolicy -A 3
```

Expected:

```
"RestartPolicy": {
    "Name": "unless-stopped",
    "MaximumRetryCount": 0
}
```

This avoids deleting the containers at all.
**Data stays untouched.**

---

# ✅ 3.2 Ensure they start now

Start them manually:

```bash
docker start scadalts
docker start mysql
docker start matlab_container
```

---

# ✅ 3.3 Reboot test

```bash
sudo reboot
```

Then run:

```bash
docker ps -a
```

All three should show: **Up X minutes (restart policy: unless-stopped)**

---

# ✅ Summary — safest path for you

✅ Check existing volumes (**prevent data loss**)
✅ Add restart policies using `docker update`
✅ No need to delete or recreate containers
✅ Reboot VM → containers auto-start

---

Reply with the **output of:**

```
docker inspect scadalts | grep -i mount -A 10
docker inspect mysql | grep -i mount -A 10
docker inspect matlab_container | grep -i mount -A 10
```

I will tell you **exactly** whether your data is safe and what to do next.



# Output of commands above

based on the output below is my data safe in all 3 containers if I stop and restart them?

```
ubuntu@pgt:~$ docker inspect scadalts | grep -i mount -A 10
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "scadalts_scadalts-data:/usr/local/tomcat/webapps/Scada-LTS:rw"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
--
        "Mounts": [
            {
                "Type": "volume",
                "Name": "scadalts_scadalts-data",
                "Source": "/var/lib/docker/volumes/scadalts_scadalts-data/_data",
                "Destination": "/usr/local/tomcat/webapps/Scada-LTS",
                "Driver": "local",
                "Mode": "rw",
                "RW": true,
                "Propagation": ""
            }
ubuntu@pgt:~$ docker inspect mysql | grep -i mount -A 10
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "scadalts_mysql-data:/var/lib/mysql:rw",
                "/home/ubuntu/scadaLTS/scadalts_dump.sql:/docker-entrypoint-initdb.d/scadalts_dump.sql:rw"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
--
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/home/ubuntu/scadaLTS/scadalts_dump.sql",
                "Destination": "/docker-entrypoint-initdb.d/scadalts_dump.sql",
                "Mode": "rw",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "volume",
ubuntu@pgt:~$ docker inspect matlab_container | grep -i mount -A 10
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "/home/ubuntu/matlab_projects:/home/matlab/projects",
                "/home/ubuntu/database_toolbox:/usr/local/MATLAB/R2025a/toolbox/database",
                "/home/ubuntu/matlab_licenses:/usr/local/MATLAB/R2025a/licenses",
                "/home/ubuntu/matlab_local:/usr/local/MATLAB/R2025a/toolbox/local"
            ],
--
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/home/ubuntu/matlab_projects",
                "Destination": "/home/matlab/projects",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
ubuntu@pgt:~$ 
```

✅ **Yes — all your data is safe in all 3 containers.**
Here is the breakdown, container by container, so you can be confident nothing will be lost even if you recreate or restart them.

---

# ✅ 1. SCADA-LTS Container — **Data is SAFE**

You have this mount:

```
"Type": "volume",
"Name": "scadalts_scadalts-data",
"Destination": "/usr/local/tomcat/webapps/Scada-LTS"
```

✅ This is a **Docker-managed volume**
✅ All SCADA-LTS application files, configs, DB files (for embedded DB, if any), and runtime data are stored in:

```
/var/lib/docker/volumes/scadalts_scadalts-data/_data
```

This will **persist across container removal, recreation, and reboots**.

✅ **SCADALTS data is safe.**

---

# ✅ 2. MySQL Container — **Data is SAFE**

You have two mounts:

### ✅ MySQL data directory volume (critical one)

```
"Type": "volume",
"Name": "scadalts_mysql-data",
"Destination": "/var/lib/mysql"
```

This means your actual MySQL database files (`.frm`, `.ibd`, logs, binary logs, etc.) are safely stored in a Docker volume.

✅ This is the most important part — your database content is protected.

### ✅ SQL dump bind mount

```
"Type": "bind",
"Source": "/home/ubuntu/scadaLTS/scadalts_dump.sql"
```

This is only for *initial database import* and is not overwritten.
It does *not* affect or replace your real MySQL data.

✅ Your MySQL database is safe and future-proof.

---

# ✅ 3. MATLAB Container — **Data is SAFE**

Everything important is mounted via bind mounts:

```
/home/ubuntu/matlab_projects → /home/matlab/projects
/home/ubuntu/database_toolbox → /usr/local/MATLAB/...
/home/ubuntu/matlab_licenses → /usr/local/MATLAB/...
/home/ubuntu/matlab_local → /usr/local/MATLAB/...
```

✅ All your projects, toolbox files, and license files are stored on the host filesystem.

None of your work is inside the container’s internal filesystem.

✅ MATLAB data is safe.

---

# ✅ ✅ Final Summary — You are SAFE to Restart, Recreate, or Apply Restart Policies

| Container     | Uses Docker Volume? | Uses Bind Mount?          | Is Data Safe? |
| ------------- | ------------------- | ------------------------- | ------------- |
| **SCADA-LTS** | ✅ Yes               | ❌ No                      | ✅ Safe        |
| **MySQL**     | ✅ Yes (DB data)     | ✅ Yes (SQL file only)     | ✅ Safe        |
| **MATLAB**    | ❌ No volume         | ✅ All data in bind mounts | ✅ Safe        |

👉 **All 3 containers store their important data outside the container filesystem.
So even if you delete the containers (`docker rm`), your data stays intact.**

---

# ✅ Next Steps (Safe to Do Now)

You can now safely apply restart policies:

```
docker update --restart unless-stopped scadalts
docker update --restart unless-stopped mysql
docker update --restart unless-stopped matlab_container
```

Then:

```
docker start scadalts
docker start mysql
docker start matlab_container
```

After reboot:

```
docker ps -a
```

All three should show “Up …” — meaning they auto-started.

---

If you want, I can also help you:

✅ convert this setup into a **docker-compose.yml**
✅ simplify all services into a clean, reproducible stack
✅ automatically restore on reboot

Just let me know.

