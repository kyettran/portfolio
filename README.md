# 🚀 kyettran.site — Production-Grade System Documentation

> Static portfolio website
> Built with Hugo (PaperMod theme)
> CI/CD via GitHub Actions
> Hosted on Ubuntu VPS
> Served by Nginx
> TLS via Let's Encrypt

This document is written at **production-grade level**.
Any engineer should be able to:

* Rebuild the system
* Operate it
* Debug it
* Understand its limitations
* Extend it safely

---

# 1️⃣ SYSTEM OVERVIEW

## 1.1 Purpose

This system hosts a static portfolio website using a fully automated CI/CD pipeline.

The VPS acts strictly as a static file server.
All build logic occurs in CI.

---

## 1.2 High-Level Architecture

### 🌐 Runtime Flow

```text id="flow001"
Client
  ↓
DNS (A record → VPS Public IP)
  ↓
Ubuntu VPS
  ↓
Nginx (Port 443)
  ↓
/var/www/portfolio/public
  ↓
Static Response
```

---

### 🚀 Deployment Flow

```text id="flow002"
Local Git Commit
  ↓
Push to GitHub (main)
  ↓
GitHub Actions Triggered
  ↓
Checkout (with submodules)
  ↓
Install Hugo
  ↓
hugo --minify
  ↓
Generate /public
  ↓
SCP to VPS
  ↓
Nginx serves new version
```

---

# 2️⃣ INFRASTRUCTURE DETAILS

## 2.1 VPS Baseline

* OS: Ubuntu LTS
* Web server: Nginx
* TLS: Certbot (Let's Encrypt)
* Open ports: 22, 80, 443

Recommended minimum:

* 1 vCPU
* 1 GB RAM
* 20 GB Disk

---

## 2.2 Directory Layout (VPS)

```text id="dir001"
/var/www/
└── portfolio/
    └── public/
```

Nginx root must match:

```nginx id="nginxroot"
root /var/www/portfolio/public;
```

---

## 2.3 Repository Layout

```text id="repo001"
portfolio/
├── content/
├── static/
├── themes/
│   └── PaperMod (submodule)
├── hugo.toml
├── .github/workflows/deploy.yml
├── .gitmodules
├── .gitignore
└── README.md
```

`.gitignore` must include:

```text id="gitignore001"
public/
```

`public/` is a build artifact and must never be committed.

---

# 3️⃣ DEPLOYMENT PROCESS

## 3.1 CI/CD Trigger

Deployment occurs when:

* Code is pushed to `main` branch.

---

## 3.2 Workflow Responsibilities

GitHub Actions performs:

1. Checkout repository (with submodules)
2. Install Hugo (version pinned)
3. Run:

```bash id="buildcmd"
hugo --minify
```

4. Upload `/public` to:

```text id="deploypath"
/var/www/portfolio/public
```

via SCP using SSH key authentication.

---

## 3.3 GitHub Secrets Required

* VPS_HOST
* VPS_USERNAME
* VPS_SSH_KEY
* (Optional) VPS_PORT

Secrets must never be hardcoded.

---

## 3.4 Emergency Manual Deploy

If CI fails:

```bash id="manualdeploy"
hugo --minify
scp -r public/* user@VPS_IP:/var/www/portfolio/public
```

---

# 4️⃣ NGINX CONFIGURATION

## 4.1 HTTP Redirect

```nginx id="nginxhttp"
server {
    listen 80;
    server_name kyettran.site www.kyettran.site;
    return 301 https://$host$request_uri;
}
```

## 4.2 HTTPS Block

```nginx id="nginxhttps"
server {
    listen 443 ssl;
    server_name kyettran.site www.kyettran.site;

    root /var/www/portfolio/public;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    ssl_certificate /etc/letsencrypt/live/kyettran.site/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/kyettran.site/privkey.pem;
}
```

Reload after change:

```bash id="reloadnginx"
systemctl reload nginx
```

---

# 5️⃣ SECURITY MODEL

## 5.1 SSH Hardening

Recommended configuration:

```text id="sshconfig"
PasswordAuthentication no
PermitRootLogin no
```

Restart SSH:

```bash id="restartssh"
systemctl restart ssh
```

---

## 5.2 Firewall

```bash id="firewall"
ufw allow 22
ufw allow 80
ufw allow 443
ufw enable
```

---

## 5.3 File Permissions

```bash id="permissions"
chown -R www-data:www-data /var/www/portfolio/public
chmod -R 755 /var/www/portfolio/public
```

---

## 5.4 TLS

Install:

```bash id="certinstall"
apt install certbot python3-certbot-nginx
```

Issue certificate:

```bash id="certissue"
certbot --nginx -d kyettran.site -d www.kyettran.site
```

Test renewal:

```bash id="certrenew"
certbot renew --dry-run
```

---

# 6️⃣ LOGGING & OBSERVABILITY

## 6.1 Log Locations

```text id="logs"
/var/log/nginx/access.log
/var/log/nginx/error.log
```

System logs:

```bash id="journal"
journalctl -u nginx
```

---

## 6.2 What To Monitor

* SSL expiration
* Disk usage
* Nginx status
* SSH login attempts
* CI/CD failure alerts

---

# 7️⃣ FAILURE HANDLING

## 7.1 Website Down

Debug order:

1. DNS resolution
2. VPS reachable?
3. Nginx running?
4. SSL valid?
5. Correct root path?
6. Logs check

---

## 7.2 GitHub Action Failure

Common causes:

* Submodule not fetched
* Hugo build error
* SSH key revoked
* Permission denied

---

## 7.3 Rollback Strategy

Option 1:

```bash id="rollback1"
git revert <commit>
git push
```

Option 2:

Re-run last successful workflow.

---

# 8️⃣ EXTERNAL DEPENDENCIES

System relies on:

* GitHub (CI/CD)
* VPS Provider
* DNS Provider
* Let's Encrypt

If any external service fails, system availability may be impacted.

---

# 9️⃣ SINGLE POINTS OF FAILURE

Current system has:

* Single VPS
* Single region
* Single CI pipeline
* No CDN
* No load balancer

If VPS fails → site unavailable.

This is acceptable for portfolio-level traffic.

---

# 🔟 LIMITATIONS

* No auto rollback
* No monitoring alerts
* No redundancy
* No blue/green deployment
* No infrastructure as code

---

# 1️⃣1️⃣ MAINTENANCE PLAN

Weekly:

* Check SSL renewal
* Check disk usage
* Check logs

Monthly:

* Update system packages
* Update Hugo version (carefully)
* Update theme submodule

---

# 1️⃣2️⃣ FUTURE IMPROVEMENTS

* Add CDN (e.g., Cloudflare)
* Add staging environment
* Add monitoring (UptimeRobot, Prometheus, etc.)
* Add Infrastructure as Code (Terraform)
* Add artifact versioning
* Implement Blue/Green deployment

---

# 1️⃣3️⃣ REBUILD CHECKLIST

If VPS is destroyed:

1. Provision Ubuntu VPS
2. Install Nginx
3. Configure firewall
4. Setup SSH keys
5. Configure Nginx root path
6. Setup Certbot
7. Configure GitHub Secrets
8. Push commit → auto deploy

System restored.

---

# 1️⃣4️⃣ SYSTEM MATURITY

This system demonstrates:

* Static site CI/CD pipeline
* Secure file transfer
* Reverse proxy configuration
* TLS automation
* Production-aware documentation

It is suitable for:

* Personal portfolio
* Low-traffic static website
* DevOps learning environment

---
