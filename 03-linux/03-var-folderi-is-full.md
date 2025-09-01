## Question  
`/var` is almost 90% full. What will be your next steps?

### 📝 Short Explanation  
This question checks your troubleshooting and disk management skills. The `/var` directory is commonly used for logs, spools, caches, and runtime data — so issues here can break system processes or fill up disks silently.

## ✅ Answer  
My first step is to identify what’s consuming the space inside `/var`. Then I would clean up unnecessary files like rotated logs, caches, or orphaned packages — and put alerts or log rotation in place to avoid recurrence.

### 📘 Detailed Explanation  

---

### ✅ Step 1: Inspect Disk Usage Under `/var`

```bash
sudo du -sh /var/* | sort -hr | head -10
```
This will show which directories inside `/var` are consuming the most space — usually it’s `/var/log`, `/var/cache`, or `/var/lib/docker`.

---

### ✅ Step 2: Clean Log Files  
If `/var/log` is the culprit:

```bash
sudo journalctl --vacuum-size=200M   #(This command tells journalctl (which manages systemd journal logs to reduce the size of the logs to 200MB. It will delete the oldest logs until the total size of the journal logs is under 200MB.)

sudo rm -rf /var/log/*.gz /var/log/*.[0-9]  #(This command forcefully removes:
# All compressed log files (*.gz) in /var/log.
# All rotated log files (e.g., syslog.1, auth.log.2, etc.))
```

Or truncate large log files:
```bash
sudo truncate -s 0 /var/log/syslog
```

---

### ✅ Step 3: Clear Package Cache  
If using `apt` or `yum`, clear the package manager cache:

```bash
sudo apt clean         # Debian/Ubuntu
sudo yum clean all     # RHEL/CentOS
```

---

### ✅ Step 4: Check Docker Artifacts  
If the server runs containers:

```bash
docker system df        # See what’s taking space
docker system prune -a  # Remove unused containers/images
```

**⚠️ Warning:** Prune removes *unused* images and volumes — be cautious on production systems.

---

### ✅ Step 5: Consider Moving or Archiving Data  
If data in `/var` is needed but rarely accessed:
- Archive old logs to `/home` or S3
- Use `logrotate` to compress and limit logs:
  ```bash
  sudo nano /etc/logrotate.conf
  ```

---

### ✅ Step 6: Set Up Alerts and Monitoring  
- Install `ncdu`, `duf`, or setup Prometheus/Grafana alerts for disk usage thresholds.
- Automate cleanup with cron or systemd timers if appropriate.

---

### 🧠 Why `/var` Fills Up:
- Verbose logging (e.g., failed cron jobs, app debug logs)
- Docker images/layers
- Orphaned cache files
- Email spools or crash dumps

> Summary:  
> Quickly inspect, clean, and automate monitoring. Ensure critical services like journald, docker, and package managers are not starved of space.

---
