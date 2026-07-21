# Lab: Free Cybersecurity Lab with SafeLine WAF (Offensive + Defensive)

## Objective
Set up a self-hosted Web Application Firewall (WAF), attack a demo web app, and analyze detection logs — covering both offensive (attacker) and defensive (SOC/engineering) perspectives.

## Environment
- **Attacker/Host:** Kali Linux (VM)
- **Tool:** [SafeLine WAF](https://waf.chaitin.com) (open-source, Docker-based, free tier)
- **Target:** Local demo Python web app ("Hello from demo")

## Setup Steps
1. Install Docker on Kali Linux.
2. Install SafeLine via official one-command script:
   ```bash
   bash -c "$(curl -fsSLk https://waf-ce.chaitin.cn/release/latest/setup.sh)"
   ```
3. Note the generated admin URL, username, and initial password from terminal output.
4. Log into SafeLine admin console (accept self-signed cert warning).
5. Create a new application in SafeLine, pointing to the demo site's upstream IP (e.g., `127.0.0.1:80`).
6. Deploy a simple Python demo website as the attack target.

## Attacks Simulated

| # | Attack Type | Method | Result |
|---|---|---|---|
| 1 | XSS | Injected classic `<script>`-style payload via URL | Blocked — "Access Forbidden" |
| 2 | SQL Injection | Classic `' OR 1=1--` style payload | Blocked — "Access Forbidden" |
| 3 | Path Traversal | `../../etc/passwd` style request | 404 response |

## Verification
- Checked **SafeLine → Logs / Attacks** tab after each attack.
- Confirmed each log entry recorded: payload, source IP, timestamp, action taken (blocked), attack type classification.
- Reviewed **Statistics** tab for request counts and error codes (4xx).

## Key Concepts Learned
- **WAF fundamentals:** reverse proxy that filters HTTP/S traffic between client and web app.
- **Security Engineering** = setting up rules/architecture (custom rules, allow/deny lists).
- **SOC Analyst role** = analyzing logs, identifying what was blocked and when.
- Practical detection flow for XSS, SQLi, and path traversal attacks.

## Skills Demonstrated
`Web Application Firewall` `SQL Injection` `XSS` `Path Traversal` `Docker` `Kali Linux` `Log Analysis` `SOC Basics`

## References
- SafeLine WAF: https://waf.chaitin.com
- GitHub: https://github.com/chaitin/safeline
- Source video: https://www.youtube.com/watch?v=Cp2sWmiXt3k
