✅ Good — `docker system prune -a` ran successfully and reclaimed **2.26 GB**.

This is expected: most of the 62 GB in `/var/lib/docker` is still being used by:

* **Running containers** → `/var/lib/docker/overlay2` layers
* **Container logs** → `/var/lib/docker/containers`

The biggest remaining culprit is usually **logs from running containers**, which can grow very large, especially for long-running PLC/IED/SCADA containers.

---

# ✅ Step 2 — Clear container logs (safe, does NOT stop containers)

Check which logs are large:

```bash
sudo du -sh /var/lib/docker/containers/*/*-json.log | sort -h
```

Then truncate all logs:

```bash
sudo sh -c 'truncate -s 0 /var/lib/docker/containers/*/*-json.log'
```

✅ This frees space **immediately**
✅ Does not restart or break containers
✅ Very safe for persistent workloads

---

# ✅ Step 3 — Optional: prune unused Docker volumes

If you have volumes that are **truly unused**, you can run:

```bash
sudo docker volume prune
```

⚠️ Be careful — volumes may contain persistent databases or configs. Only delete if you are sure.

---

# ✅ Step 4 — Recheck disk usage

After truncating logs and pruning:

```bash
df -h
sudo du -h -d1 /var/lib/docker | sort -h
```

You should see **several tens of GB freed** and `/` no longer at 100%.

---

If you want, I can give you a **one-line set of commands** to safely reclaim the **maximum possible space** from your Docker setup **without deleting running containers or important volumes**.

Do you want me to do that?
