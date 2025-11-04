Below is **the exact step-by-step way to dockerize your first IED (ST_IED)** using the networking architecture you selected.

✅ After this, you will be able to repeat the process for GT, EG, HRSG by copying 1 folder and changing:
– the Python script filename
– the Modbus port
– the MACVLAN IP

Everything else stays identical.

---

# ✅ **PART 1 — Prepare the first IED for Dockerizing (ST IED)**

We will create this structure:

```
st_ied/
│
├── Dockerfile
├── requirements.txt
└── ST_IED_pg.py
```

---

# ✅ **1. Create `requirements.txt`**

`requirements.txt`:

```
psycopg2
pymodbus
```

---

# ✅ **2. Create the Dockerfile**

`Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install Postgres client dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq-dev gcc \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python3", "ST_IED_pg.py"]
```

✅ This Dockerfile is reusable for all IEDs.
You only replace the Python filename inside `CMD`.

---

# ✅ **3. Modify your ST_IED Python script to use environment variables**

Replace your `establish_connection()` function with this:

```python
import os

def establish_connection():
    conn = psycopg2.connect(
        host=os.getenv("DB_HOST", "ccp-multi-ied-multi-plc-pg"),
        port=os.getenv("DB_PORT", "5432"),
        user=os.getenv("DB_USER", "root"),
        password=os.getenv("DB_PASS", "root"),
        database=os.getenv("DB_NAME", "powerplant")
    )
    return conn
```

✅ No more hardcoded IP
✅ Container resolves Postgres by **container name**
✅ DB host changes don’t affect your script anymore

---

# ✅ **PART 2 — Create the docker-compose entry for ST IED**

Since you already created the two networks:

✅ `ied_pg_net` (for DB access)
✅ `plc-macvlan-40` (for PLC access)

Your docker-compose for **one IED** looks like this:

`docker-compose.yml`:

```yaml
version: "3.9"

services:
  st_ied:
    build: ./st_ied
    container_name: st_ied
    networks:
      ied_pg_net: {}                 # bridge network for DB
      plc-macvlan-40:
        ipv4_address: 192.168.40.210  # example IP
    environment:
      DB_HOST: ccp-multi-ied-multi-plc-pg
      DB_PORT: "5432"
      DB_USER: root
      DB_PASS: root
      DB_NAME: powerplant
    ports:
      - "504:504"   # expose for debugging (optional)
      - "50001:50001"  # TCP listener (optional)

networks:
  ied_pg_net:
    external: true

  plc-macvlan-40:
    external: true
```

### ✅ Why this works

✔ ST IED joins `ied_pg_net`, so it can reach PostgreSQL by:

```
host = "ccp-multi-ied-multi-plc-pg"
```

✔ ST IED joins `plc-macvlan-40` with a static IP:

```
192.168.40.210
```

✔ PLC (at e.g. 192.168.40.202) can reach the IED via Modbus at:

```
192.168.40.210:504
```

---

# ✅ **PART 3 — Build + Run your first dockerized IED**

Build:

```bash
docker compose build st_ied
```

Run:

```bash
docker compose up -d st_ied
```

Check its IP inside the macvlan network:

```bash
docker inspect st_ied | grep IPAddress
```

Check logs:

```bash
docker logs -f st_ied
```

✅ If you see `[ST] Initial Modbus context values…` → the IED is working
✅ If you see `[ST: DB sync] Updated Modbus reg…` → DB polling works
✅ PLC should now read registers

---

# ✅ **PART 4 — How to dockerize the remaining IEDs**

Now simply repeat this **exact same folder pattern**:

```
eg_ied/
│── Dockerfile
│── requirements.txt
│── EG_IED_pg.py

gt_ied/
│── Dockerfile
│── requirements.txt
│── GT_IED_pg.py

hrsg_ied/
│── Dockerfile
│── requirements.txt
│── HRSG_IED_pg.py
```

And add each service to docker-compose:

✅ EG IED Example

```yaml
  eg_ied:
    build: ./eg_ied
    container_name: eg_ied
    networks:
      ied_pg_net: {}
      plc-macvlan-40:
        ipv4_address: 192.168.40.211
    environment:
      DB_HOST: ccp-multi-ied-multi-plc-pg
      DB_PORT: "5432"
      DB_USER: root
      DB_PASS: root
      DB_NAME: powerplant
    ports:
      - "505:505"
```

✅ GT IED

```yaml
  gt_ied:
    build: ./gt_ied
    container_name: gt_ied
    networks:
      ied_pg_net: {}
      plc-macvlan-40:
        ipv4_address: 192.168.40.212
    environment:
      DB_HOST: ccp-multi-ied-multi-plc-pg
      DB_PORT: "5432"
      DB_USER: root
      DB_PASS: root
      DB_NAME: powerplant
    ports:
      - "502:502"
```

✅ HRSG IED

```yaml
  hrsg_ied:
    build: ./hrsg_ied
    container_name: hrsg_ied
    networks:
      ied_pg_net: {}
      plc-macvlan-40:
        ipv4_address: 192.168.40.213
    environment:
      DB_HOST: ccp-multi-ied-multi-plc-pg
      DB_PORT: "5432"
      DB_USER: root
      DB_PASS: root
      DB_NAME: powerplant
    ports:
      - "503:503"
      - "50000:50000"
```

---

# ✅ **PART 5 — Final Networking Diagram**

```
         (ied_pg_net)
  +------------------------------+
  |    Postgres (container)     |
  +------------------------------+
               |
               |
         +-----+------+
         |   IEDs     | (all also join plc-macvlan-40)
         +------------+
          |    |    |
    192.168.40.210  (ST)
    192.168.40.211  (EG)
    192.168.40.212  (GT)
    192.168.40.213  (HRSG)
          |
          |  (physical MACVLAN)
   +------+------------------+
   |          PLC           |
   |   192.168.40.202       |
   +------------------------+
```

✅ **PLC ↔ IED** on MACVLAN subnet
✅ **IED ↔ Postgres** on shared bridge network
✅ **PLCs cannot reach PostgreSQL** (secure)

---

# ✅ **If you want:**

I can generate **the full production-ready docker-compose** containing:

✅ All 4 IEDs
✅ Naming conventions
✅ Restart policies
✅ Healthchecks
✅ Logs mounted to local files
✅ Automatic DB wait-for initialization

Just say:
**“Generate full multi-IED docker-compose.”**
