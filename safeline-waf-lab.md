# Lab: Free Cybersecurity Lab with SafeLine WAF (Offensive + Defensive)

## Overview
This lab demonstrates how to deploy an open-source Web Application Firewall (WAF), protect a vulnerable demo web app behind it, launch common web attacks against that app, and confirm detection/blocking via the WAF's logs. It covers both the **offensive** side (attacker perspective) and the **defensive** side (SOC analyst / security engineer perspective).

## Learning Objectives
By the end of this lab you will be able to:
- Deploy a self-hosted WAF using Docker
- Explain how a reverse-proxy WAF sits between client and server
- Simulate XSS, SQL Injection, and Path Traversal attacks
- Read and interpret WAF logs like a SOC analyst
- Differentiate "security engineering" (rules/architecture) from "security analysis" (log review)

## Architecture
```
Attacker (curl/browser)
        |
        v
  SafeLine WAF (reverse proxy, port 9443 mgmt / custom app port)
        |
        v
 Demo Target App (Python http.server, 127.0.0.1:8000)
```
All traffic to the demo app is routed through SafeLine first. SafeLine inspects each request, and either forwards it (safe) or blocks it (malicious) and logs the event.

## Prerequisites

| Requirement | Minimum |
|---|---|
| OS | Linux (Kali, Ubuntu, or any Debian-based distro) |
| CPU | 1 core |
| RAM | 1 GB |
| Disk | 5 GB free |
| Docker | v20.10.6+ |
| Docker Compose | v2.0.0+ |
| Access | root / sudo |

Check your versions before starting:
```bash
docker --version
docker compose version
```
If Docker isn't installed:
```bash
curl -fsSL https://get.docker.com | sudo sh
```

---

## Part 1: Install SafeLine WAF

Run the official one-click installer as root:
```bash
sudo bash -c "$(curl -fsSLk https://waf.chaitin.com/release/latest/manager.sh)" -- --en
```

**What this script does:**
1. Downloads SafeLine's Docker Compose configuration to `/data/safeline`
2. Pulls all required Docker images (management console, detection engine, database, cache)
3. Generates random secure credentials for internal services
4. Starts all containers
5. Prints your login URL, username, and password

**Expected output (example):**
```
SafeLine Management Console: https://192.168.1.50:9443
Username: admin
Password: aB3xK9mZ...
```
📌 **Copy this password immediately** — it's unique per install and shown only once.

### Verify installation
```bash
docker ps
```
You should see multiple `safeline-*` containers with status `Up`:
```
safeline-mgt
safeline-detector
safeline-postgres
safeline-cache
safeline-tengine
```

If any container is not "Up", check logs:
```bash
docker logs safeline-mgt
```

---

## Part 2: Log into the Admin Console

1. Open a browser and go to the URL from Part 1 (e.g. `https://192.168.1.50:9443`)
2. You'll see a certificate warning (self-signed cert) → click **Advanced → Proceed anyway**
3. Enter:
   - Username: `admin`
   - Password: (from install output)
4. On first login, you may be prompted to set a permanent password — do so and store it safely (e.g., password manager, not plaintext).

You're now looking at the **SafeLine Dashboard**, showing request volume, visitor counts, blocked requests, and geo-location of traffic.

---

## Part 3: Deploy a Vulnerable/Demo Target App

We need something for the WAF to protect. A minimal static site is enough.

```bash
mkdir -p ~/demo-site && cd ~/demo-site
echo "<h1>Hello from demo</h1>" > index.html
python3 -m http.server 8000
```
Leave this terminal running — it serves your site on `127.0.0.1:8000`.

Verify it works (in a second terminal):
```bash
curl http://127.0.0.1:8000
```
You should see the HTML back.

---

## Part 4: Register the Site in SafeLine

1. In the SafeLine console, go to **Applications → Add Application**
2. Fill in:
   - **Name:** `demo-site`
   - **Upstream server:** `127.0.0.1:8000` (your target app)
   - **Listen port:** SafeLine will assign or let you choose a port (e.g. `8080`)
3. Save.

From now on, traffic must go through SafeLine's assigned port to reach your app — this is your **protected endpoint**.

Verify the proxy works:
```bash
curl http://127.0.0.1:8080
```
You should get the same "Hello from demo" response, now passing through the WAF.

---

## Part 5: Offensive Testing — Simulate Attacks

Replace `<port>` with the SafeLine-assigned port (e.g. `8080`) in every command below.

### Attack 1: Cross-Site Scripting (XSS)
XSS injects malicious JavaScript into a page via unsanitized input.
```bash
curl "http://127.0.0.1:<port>/?q=<script>alert(1)</script>"
```
**Expected result:** `403 Forbidden` (blocked)

### Attack 2: SQL Injection (SQLi)
SQLi manipulates a backend database query via unsanitized input.
```bash
curl "http://127.0.0.1:<port>/?id=1' OR '1'='1"
```
**Expected result:** `403 Forbidden` (blocked)

### Attack 3: Path Traversal
Attempts to escape the web root and read arbitrary files.
```bash
curl "http://127.0.0.1:<port>/../../../../etc/passwd"
```
**Expected result:** `404 Not Found` or `403 Forbidden`

---

## Part 6: Defensive Analysis — Review the Logs

Back in the SafeLine console:
1. Go to **Attacks → Logs** (or **Events**)
2. Click **Refresh** — you should see 3 new entries, one per attack
3. Click into each entry to inspect:
   - Source IP
   - Timestamp
   - Attack type (XSS / SQLi / Path Traversal)
   - Full request payload
   - Action taken (blocked)

Also check **Statistics** for a broader view: total requests, blocked count, and status code breakdown (4xx spikes = your test traffic).

**This is exactly what a SOC analyst does daily** — reviewing alerts, confirming they're legitimate detections (not false positives), and documenting them.

---

## Part 7: Cleanup (Optional)

Stop the demo site:
```
Ctrl + C   # in the python3 http.server terminal
```

Stop and remove SafeLine:
```bash
cd /data/safeline
docker compose down
```

To fully remove all data:
```bash
sudo rm -rf /data/safeline
```

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Can't reach console on 9443 | Check firewall: `sudo ufw allow 9443/tcp` |
| `docker ps` shows containers restarting | Check RAM/disk space; `docker logs <container>` |
| Attacks return `200 OK` instead of blocked | Confirm app is registered correctly in SafeLine and you're hitting the SafeLine port, not the raw app port (8000) |
| Forgot admin password | Check official docs for the current admin-reset command |

---

## Key Takeaways
- A **WAF** is a reverse proxy that filters malicious HTTP/S traffic before it reaches your app.
- **Security engineering** = designing rules, architecture, and detection logic.
- **SOC analysis** = reviewing logs, confirming detections, and responding to incidents.
- This same workflow (deploy → protect → attack → detect → analyze) mirrors real-world SOC/AppSec work.

## Skills Demonstrated
`Web Application Firewall` `SQL Injection` `XSS` `Path Traversal` `Docker` `Linux` `Log Analysis` `SOC Fundamentals` `Reverse Proxy Architecture`

## References
- SafeLine official docs: https://docs.waf.chaitin.com
- SafeLine GitHub: https://github.com/chaitin/safeline
- Source video: https://www.youtube.com/watch?v=Cp2sWmiXt3k
