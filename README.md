# 🚀 KyetTran.site — Full Deployment Documentation

> Static Portfolio Website built with Hugo + PaperMod
> CI/CD via GitHub Actions
> Hosted on VPS with Nginx + HTTPS (Let's Encrypt)

---

# 1️⃣ PROJECT OVERVIEW

This project is a static portfolio website.

Tech stack:

* Hugo (Static Site Generator)
* PaperMod theme (Git submodule)
* GitHub Actions (CI/CD)
* VPS (Ubuntu)
* Nginx (Web server)
* Let's Encrypt (SSL)

Deployment strategy:

GitHub builds the site → uploads `/public` → VPS only serves static files.

---

# 2️⃣ SERVER SETUP (VPS)

## 2.1 Install Required Packages

```bash
apt update
apt install nginx git certbot python3-certbot-nginx -y
```

---

## 2.2 Project Directory

Website root:

```
/var/www/portfolio
```

Final static files served from:

```
/var/www/portfolio/public
```

---

# 3️⃣ NGINX CONFIGURATION

Check config:

```bash
cat /etc/nginx/sites-enabled/default
```

Must contain:

```
server_name kyettran.site www.kyettran.site;

root /var/www/portfolio/public;
index index.html;
```

Reload nginx:

```bash
systemctl reload nginx
```

---

# 4️⃣ SSL SETUP (HTTPS)

Command used:

```bash
certbot --nginx -d kyettran.site -d www.kyettran.site
```

Selected:

```
2 (Redirect HTTP → HTTPS)
```

Auto-renew tested:

```bash
certbot renew --dry-run
```

Result:
✔ SSL auto-renews automatically via system timer.

---

# 5️⃣ HUGO PROJECT STRUCTURE

```
portfolio/
├── archetypes/
├── content/
├── static/
├── themes/
│   └── PaperMod (submodule)
├── hugo.toml
├── .gitmodules
└── .github/workflows/deploy.yml
```

Important:

❌ `public/` is NOT committed
It is build output only.

---

# 6️⃣ REMOVE public/ FROM GIT (IMPORTANT FIX)

On local:

```bash
rm -rf public
```

Add to `.gitignore`:

```
public/
```

Remove from git tracking:

```bash
git rm -r --cached public
git commit -m "remove public folder"
git push
```

Reason:

`public/` is build output.
CI/CD should generate it, not Git.

---

# 7️⃣ THEME FIX (SUBMODULE ISSUE)

Problem:

Hugo build warning:

```
found no layout file
```

Cause:
Theme not loaded in GitHub Actions.

Fix in `.github/workflows/deploy.yml`:

```yaml
- uses: actions/checkout@v4
  with:
    submodules: recursive
```

Now theme loads correctly.

---

# 8️⃣ CI/CD PIPELINE (FINAL VERSION)

Workflow:

```
git push
   ↓
GitHub Actions
   ↓
Checkout repo + submodules
   ↓
Install Hugo
   ↓
hugo --minify
   ↓
Upload public/ to VPS
   ↓
Nginx serves updated site
```

---

# 9️⃣ GITHUB ACTIONS LOGIC

Main steps:

```yaml
- uses: actions/checkout@v4
  with:
    submodules: recursive

- uses: peaceiris/actions-hugo@v2

- run: hugo --minify

- uses: appleboy/scp-action@v0.1.7
  with:
    source: "public/*"
    target: "/var/www/portfolio/public"
```

Important:

VPS does NOT build Hugo anymore.
GitHub builds everything.

---

# 🔟 HOW TO UPDATE WEBSITE

Whenever you want to:

* Change subtitle
* Add blog post
* Update avatar
* Modify hugo.toml
* Edit content

Do:

```bash
git add .
git commit -m "update content"
git push
```

Wait ~20 seconds → website auto-updates.

No SSH needed.

---

# 1️⃣1️⃣ COMMON TROUBLESHOOTING

### Site not updating?

* Check GitHub Actions log
* Confirm build succeeded
* Confirm SCP step succeeded

---

### Theme broken?

* Check `submodules: recursive` exists
* Check `.gitmodules` file

---

### HTTPS not working?

Check:

```bash
certbot renew --dry-run
systemctl status nginx
```

---

# 1️⃣2️⃣ CURRENT PRODUCTION STATUS

✔ HTTPS enabled
✔ Auto-renew SSL
✔ CI/CD automated
✔ Hugo build on GitHub
✔ VPS serving static files only
✔ public/ removed from repo
✔ Submodule working

System is production-ready.

---

# 1️⃣3️⃣ FUTURE IMPROVEMENTS (NEXT PHASE)

Possible upgrades:

* Enable gzip / brotli
* Add security headers
* Enable cache-control headers
* Add analytics
* Add preview deployment branch
* Add staging environment

---

# 🧠 ARCHITECTURE SUMMARY

This is now a clean DevOps-style static deployment system:

GitHub = Builder
VPS = Static file server
Nginx = Web server
Certbot = SSL automation

Simple. Stable. Scalable.

---

# 📌 HOW TO DISCUSS FUTURE IMPROVEMENTS

When asking ChatGPT about improvements, reference:

* Current structure: GitHub builds, VPS serves
* Path: `/var/www/portfolio/public`
* Using Hugo + PaperMod
* CI/CD via GitHub Actions
* SSL via Certbot

This context ensures accurate advice.

---
