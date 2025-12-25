# 🌐 Complete VPS Hosting Guide (MEAN Stack)

## Quick Deploy Option
[![vpsify](https://raw.githubusercontent.com/shaishab316/vpsify/refs/heads/main/public/icons/logo.svg)](https://github.com/shaishab316/vpsify.git)

---

## 📋 Prerequisites

Before you begin, ensure you have:
- A VPS with Ubuntu 20.04+ or Debian 11+ (minimum 2GB RAM recommended)
- Root or sudo access to your VPS
- A domain name
- Your project code ready (Frontend, Backend, Dashboard)

---

## ✅ Step 0: Configure Domain DNS

### DNS Configuration Steps:

1. Log into your domain provider's dashboard (Namecheap, GoDaddy, Cloudflare, Hostinger, Porkbun, etc.)
2. Navigate to **Domain List → DNS Management/Zone**
3. **Check for existing A records:**
   - Look for records named `@`, `www`, `api`, or `dashboard`
   - **Delete all existing A records** to avoid conflicts

> ⚠️ **Critical:** Multiple A records with the same name will cause DNS resolution failures!

4. **Add three new A records** (replace `YOUR_VPS_IP` with your actual VPS IP address):
4. Now add three new A records as shown below (use your VPS IP):
<img width="1232" height="791" alt="image" src="https://gist.github.com/user-attachments/assets/3c5ca11f-42b4-4e99-8818-91baf6bc569e" />

| Type | Name      | Value (IP)     | TTL  |
|------|-----------|----------------|------|
| A    | @         | YOUR_VPS_IP    | 3600 |
| A    | www       | YOUR_VPS_IP    | 3600 |
| A    | api       | YOUR_VPS_IP    | 3600 |
| A    | dashboard | YOUR_VPS_IP    | 3600 |

5. **Save changes** and wait 5-30 minutes for DNS propagation

### Verify DNS Propagation:

```bash
# Check if DNS is propagated (run on your local machine)
nslookup yourdomain.com
nslookup api.yourdomain.com
nslookup dashboard.yourdomain.com
```

---

## ✅ Step 1: Connect to VPS Using VS Code

### Installation Steps:

1. Open **Visual Studio Code**
2. Press `Ctrl + Shift + X` (or `Cmd + Shift + X` on Mac) to open Extensions
3. Search for **"Remote - SSH"** by Microsoft
4. Click **Install**

### Connect to VPS:

1. Press `Ctrl + Shift + P` (or `Cmd + Shift + P` on Mac)
2. Type and select: **"Remote-SSH: Connect to Host..."**
3. Enter your connection string:
   ```
   root@YOUR_VPS_IP
   ```
4. Press **Enter**
5. Select **Linux** as the platform
6. Enter your VPS password when prompted
7. Wait for the connection to establish

> 💡 **Tip:** For password-less login, set up SSH keys for better security.

---

## ✅ Step 2: Install Required Software

### Run Initial System Update:

```bash
# Update package lists and upgrade existing packages
apt update && apt upgrade -y
```

### Install Core Dependencies:

```bash
# Install NGINX, Git, Certbot, and other essentials
apt install -y nginx curl unzip git certbot python3-certbot-nginx ufw
```

### Install Node.js (Latest LTS):

```bash
# Install Node.js 24.x (or use 20.x for LTS)
curl -fsSL https://deb.nodesource.com/setup_24.x | bash -
apt install -y nodejs

# Verify installation
node --version
npm --version
```

### Install PM2 (Process Manager):

```bash
# Install PM2 globally
npm install -g pm2

# Verify installation
pm2 --version
```

### Configure NGINX:

```bash
# Enable and start NGINX
systemctl enable nginx
systemctl start nginx
systemctl status nginx
```

### Configure Firewall (Optional but Recommended):

```bash
# Allow SSH, HTTP, and HTTPS
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
ufw status
```

---

## ✅ Step 3: Upload and Prepare Your Projects

### Create Project Directories:

```bash
# Create directories if they don't exist
mkdir -p /var/www/frontend
mkdir -p /var/www/backend
mkdir -p /var/www/dashboard
```

### Upload Projects via VS Code:

1. In VS Code, open **File Explorer** (Ctrl + Shift + E)
2. Click **"Open Folder"** and navigate to `/var/www/`
3. **Drag and drop** your project folders:
   - **Frontend** (React/Angular/Vue) → `/var/www/frontend`
   - **Backend** (Node.js/Express) → `/var/www/backend`
   - **Dashboard** (React/Angular/Vue) → `/var/www/dashboard`

### Alternative: Upload via Git:

```bash
# If your projects are on GitHub/GitLab
cd /var/www/frontend
git clone https://github.com/yourusername/frontend.git .

cd /var/www/backend
git clone https://github.com/yourusername/backend.git .

cd /var/www/dashboard
git clone https://github.com/yourusername/dashboard.git .
```

### Install Dependencies:

```bash
# Frontend
cd /var/www/frontend
npm install
npm run build  # If it's a production build

# Backend
cd /var/www/backend
npm install

# Dashboard
cd /var/www/dashboard
npm install
npm run build  # If it's a production build
```

### Create Environment Files:

```bash
# Create .env file for backend
cd /var/www/backend
nano .env
```

Example `.env` file:
```
PORT=5000
NODE_ENV=production
DATABASE_URL=mongodb://localhost:27017/yourdb
JWT_SECRET=your_secret_key_here
```

> 💡 **Security Tip:** Never commit `.env` files to Git! Add them to `.gitignore`.

---

## ✅ Step 4: Run Projects with PM2

### Start Applications:

```bash
# Frontend (if using Next.js or similar)
cd /var/www/frontend
pm2 start npm --name "frontend" -- start

# Backend (Node.js/Express)
cd /var/www/backend
pm2 start npm --name "backend" -- start
# OR if you have a specific entry file:
pm2 start server.js --name "backend"

# Dashboard
cd /var/www/dashboard
pm2 start npm --name "dashboard" -- start
```

### For Static Builds (React/Vue/Angular):

If your frontend and dashboard are static builds, serve them directly with NGINX (skip PM2 for these).

### PM2 Configuration:

```bash
# Save current PM2 process list
pm2 save

# Set PM2 to start on system boot
pm2 startup

# Copy and run the command that PM2 outputs
```

### Verify Running Processes:

```bash
# List all PM2 processes
pm2 list

# Check logs
pm2 logs

# Check specific app logs
pm2 logs backend
```

---

## ✅ Step 5: Configure NGINX

### Navigate to NGINX Configuration:

```bash
cd /etc/nginx/sites-available/
```

### Remove Default Configuration:

```bash
# CRITICAL: Delete the default file to avoid conflicts
rm /etc/nginx/sites-available/default
rm /etc/nginx/sites-enabled/default
```

> ⚠️ **Important:** The default NGINX page will override your configurations if not removed!

### Create Configuration Files:

#### 1. Main Website (`root`)

```bash
nano /etc/nginx/sites-available/root
```

**For Static Build (React/Vue/Angular):**
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;

    root /var/www/frontend/build;  # or /dist for Vue/Angular
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
}
```

**For Node.js Server (Next.js, etc.):**
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

#### 2. API Backend (`api`)

```bash
nano /etc/nginx/sites-available/api
```

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name api.yourdomain.com;

    # Increase timeout for long-running requests
    proxy_read_timeout 300;
    proxy_connect_timeout 300;
    proxy_send_timeout 300;

    location / {
        proxy_pass http://localhost:5000;  # Change port if needed
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # CORS headers (if needed)
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization' always;
    }
}
```

#### 3. Dashboard (`dashboard`)

```bash
nano /etc/nginx/sites-available/dashboard
```

**For Static Build:**
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name dashboard.yourdomain.com;

    root /var/www/dashboard/build;  # or /dist
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
}
```

**For Node.js Server:**
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name dashboard.yourdomain.com;

    location / {
        proxy_pass http://localhost:3001;  # Different port from main app
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Enable Configurations:

```bash
# Create symbolic links to enable sites
ln -s /etc/nginx/sites-available/root /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/api /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/dashboard /etc/nginx/sites-enabled/
```

### Test and Reload NGINX:

```bash
# Test configuration syntax
nginx -t

# If test passes, reload NGINX
systemctl reload nginx

# Check NGINX status
systemctl status nginx
```

---

## ✅ Step 6: Install SSL Certificates (HTTPS)

### Install Certbot (if not already installed):

```bash
apt install certbot python3-certbot-nginx -y
```

### Obtain SSL Certificates:

```bash
# For all domains at once
certbot --nginx -d yourdomain.com -d www.yourdomain.com -d api.yourdomain.com -d dashboard.yourdomain.com
```

**Or obtain separately:**

```bash
# Main domain
certbot --nginx -d yourdomain.com -d www.yourdomain.com

# API subdomain
certbot --nginx -d api.yourdomain.com

# Dashboard subdomain
certbot --nginx -d dashboard.yourdomain.com
```

### During Installation:

1. Enter your email address
2. Agree to Terms of Service (Y)
3. Choose whether to share email with EFF (optional)
4. **Choose option 2:** Redirect HTTP to HTTPS (recommended)

### Verify SSL Installation:

```bash
# Check certificate status
certbot certificates
```

### Set Up Auto-Renewal:

```bash
# Test renewal process
certbot renew --dry-run

# Renewal is automatic via systemd timer, but you can check:
systemctl status certbot.timer
```

---

## 🎉 Deployment Complete!

### Your URLs:

| URL                                  | Purpose          | Protocol |
|--------------------------------------|------------------|----------|
| `https://yourdomain.com`            | Main Website     | HTTPS ✅  |
| `https://www.yourdomain.com`        | Main Website     | HTTPS ✅  |
| `https://api.yourdomain.com`        | Backend API      | HTTPS ✅  |
| `https://dashboard.yourdomain.com`  | Admin Dashboard  | HTTPS ✅  |

---

## ⚙️ Essential Commands Reference

### 🟢 PM2 Process Management

```bash
# List all processes
pm2 list

# View logs (all processes)
pm2 logs

# View logs (specific process)
pm2 logs backend

# Restart process
pm2 restart backend
pm2 restart all

# Stop process
pm2 stop backend
pm2 stop all

# Delete process
pm2 delete backend
pm2 delete all

# Monitor processes
pm2 monit

# Save process list
pm2 save

# Setup startup script
pm2 startup

# Update PM2
npm install pm2 -g && pm2 update
```

### 🌐 NGINX Management

```bash
# Test configuration
nginx -t

# Reload configuration
systemctl reload nginx

# Restart NGINX
systemctl restart nginx

# Stop NGINX
systemctl stop nginx

# Start NGINX
systemctl start nginx

# Check status
systemctl status nginx

# View error logs
tail -f /var/log/nginx/error.log

# View access logs
tail -f /var/log/nginx/access.log
```

### 🔐 SSL/Certbot Commands

```bash
# List all certificates
certbot certificates

# Renew all certificates
certbot renew

# Test renewal (dry run)
certbot renew --dry-run

# Revoke certificate
certbot revoke --cert-path /etc/letsencrypt/live/yourdomain.com/fullchain.pem

# Delete certificate
certbot delete --cert-name yourdomain.com
```

### 🔄 Git Deployment

```bash
# Pull latest changes
cd /var/www/backend
git pull origin main

# Rebuild and restart
npm install
pm2 restart backend

# For frontend/dashboard (if static)
cd /var/www/frontend
git pull origin main
npm install
npm run build
systemctl reload nginx
```

### 📊 System Monitoring

```bash
# Check disk usage
df -h

# Check memory usage
free -h

# Check CPU usage
top

# Check running processes
ps aux | grep node

# Check port usage
netstat -tulpn | grep LISTEN
# or
ss -tulpn | grep LISTEN

# Check system logs
journalctl -xe
```

---

## 🔧 Troubleshooting Guide

### Problem: Website not loading

**Solutions:**
```bash
# Check if NGINX is running
systemctl status nginx

# Check NGINX error logs
tail -f /var/log/nginx/error.log

# Verify DNS is working
nslookup yourdomain.com

# Check if port 80/443 is accessible
ufw status
```

### Problem: PM2 app not starting

**Solutions:**
```bash
# Check PM2 logs
pm2 logs backend

# Restart the app
pm2 restart backend

# Delete and re-add
pm2 delete backend
cd /var/www/backend
pm2 start npm --name backend -- start
```

### Problem: SSL certificate issues

**Solutions:**
```bash
# Check certificate status
certbot certificates

# Force renew
certbot renew --force-renewal

# Check NGINX configuration
nginx -t
```

### Problem: Port already in use

**Solutions:**
```bash
# Find what's using the port
lsof -i :5000

# Kill the process (replace PID)
kill -9 PID

# Or use PM2 to restart
pm2 restart backend
```

---

## 🔒 Security Best Practices

### 1. Change Default SSH Port:

```bash
nano /etc/ssh/sshd_config
# Change Port 22 to Port 2222
systemctl restart sshd
```

### 2. Disable Root Login:

```bash
nano /etc/ssh/sshd_config
# Set PermitRootLogin no
systemctl restart sshd
```

### 3. Set Up Automatic Updates:

```bash
apt install unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades
```

### 4. Install Fail2Ban:

```bash
apt install fail2ban -y
systemctl enable fail2ban
systemctl start fail2ban
```

### 5. Regular Backups:

```bash
# Backup script example
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
tar -czf /backup/website_$DATE.tar.gz /var/www/
```

---

## 📚 Additional Resources

- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [NGINX Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

---

## 💡 Pro Tips

1. **Use environment variables** for sensitive data
2. **Enable NGINX caching** for better performance
3. **Set up monitoring** with tools like PM2 Plus or New Relic
4. **Use CDN** (Cloudflare) for static assets
5. **Regular backups** of both code and database
6. **Monitor logs** regularly for errors and security issues
7. **Keep system updated** with `apt update && apt upgrade`

---

**Need help?** Open an issue on the [vpsify repository](https://github.com/shaishab316/vpsify/issues).

---
# Easy steps

<!-- -----------------------Easy steps ------------------- -->
# 🌐 Easy Hosting Guide on Vps (MEAN stack)

## One click deploy using
[![vpsify](https://raw.githubusercontent.com/shaishab316/vpsify/refs/heads/main/public/icons/logo.svg)](https://github.com/shaishab316/vpsify.git)

---

## ✅ Step 0: Set Up Domain First

1. Open your domain provider’s dashboard
(e.g., Namecheap, GoDaddy, Cloudflare, Hostinger, Porkbun, etc.)
2. Go to **Domain List → DNS Zone**
3. Now observe:
   🔍 If there are old A records named **@, api or dashboard**
   👉 **Delete them first.**

> ⚠️ DNS will NOT work if there are multiple A records with the same name.

4. Now add three new A records as shown below (use your VPS IP):
<img width="1232" height="791" alt="image" src="https://gist.github.com/user-attachments/assets/3c5ca11f-42b4-4e99-8818-91baf6bc569e" />


---

## ✅ Step 1: Log in to VPS Using VS Code

1. Open VS Code
2. Go to Extensions (`Ctrl + Shift + X`)
3. Search **Remote - SSH** → Install
<img width="813" height="152" alt="image" src="https://gist.github.com/user-attachments/assets/c9a4347c-8142-4e48-88b8-c5b6425cc9e2" />

4. Press `Ctrl + Shift + P` → search:
<img width="606" height="64" alt="image" src="https://gist.github.com/user-attachments/assets/62e0e33c-7fe1-4bff-ae98-fa2476cc01ca" />

5. When asked, enter:
<img width="604" height="90" alt="image" src="https://gist.github.com/user-attachments/assets/0e7be8f5-0cd8-4298-a57d-3326bac2c80e" />

6. Press Enter and you will be connected 🎉

---

## ✅ Step 2: Install Necessary Software in VPS

Copy and run:

```bash
apt update && apt upgrade -y
apt install -y nginx curl unzip git certbot python3-certbot-nginx

curl -fsSL https://deb.nodesource.com/setup_24.x | bash -
apt install -y nodejs
npm i -g pm2

systemctl enable nginx
systemctl start nginx
```

👉 This installs all required packages & starts NGINX.

---

## ✅ Step 3: Upload Your Projects (Drag & Drop)

In VS Code, open `/var/www/` folder.

Drag your project folders here:

| Project           | Upload To            |
| ----------------- | -------------------- |
| Frontend (React)  | `/var/www/frontend`  |
| Backend (Node.js) | `/var/www/backend`   |
| Dashboard (React) | `/var/www/dashboard` |

Then go inside each folder and run:

```bash
npm install
```

---

## ✅ Step 4: Run Projects Using PM2

```bash
cd /var/www/frontend
pm2 start npm --name frontend -- start

cd /var/www/backend
pm2 start npm --name backend -- start

cd /var/www/dashboard
pm2 start npm --name dashboard -- start
```

Save and enable autostart:

```bash
pm2 save
pm2 startup
```

---

## ✅ Step 5: Configure NGINX

### 📁 Go to `/etc/nginx/sites-available/`

---

### ⚠️ Important

🔥 Delete the file named `default`.
🛑 Do **not** delete any other file.

**Why?**
If it stays, your subdomains will not work and you will see the default NGINX page.

---

### 🛠 Create 3 files:

* `root`
* `api`
* `dashboard`

Put the following configuration (change subdomain + port):

```nginx
server {
    listen 80;
    server_name (subdomain.)yourdomain.com;

    location / {
        proxy_pass http://localhost:(port);
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable them:

```bash
sudo ln -s /etc/nginx/sites-available/root /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/api /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/dashboard /etc/nginx/sites-enabled/
```
<img width="815" height="372" alt="image" src="https://gist.github.com/user-attachments/assets/35ef8a47-375f-4fc6-8e51-2b7c320f7997" />

---

### 🔁 Reload NGINX

```bash
nginx -t
systemctl reload nginx
```

---

## ✅ Step 6: Install SSL (HTTPS Free Certificate)

```bash
certbot --nginx -d yourdomain.com -d www.yourdomain.com -d api.yourdomain.com -d dashboard.yourdomain.com
```

Choose **Redirect HTTP to HTTPS** when asked.

---

## 🎉 Done!

| URL                                                    | Shows       |
| ------------------------------------------------------ | ----------- |
| `https://yourdomain.com`, `https://www.yourdomain.com` | Main App    |
| `https://api.yourdomain.com`                           | Backend API |
| `https://dashboard.yourdomain.com`                     | Dashboard   |

---

## ⚙️ Useful Commands

### 🟢 PM2

```bash
pm2 ls
pm2 restart name/all
pm2 stop name
pm2 delete name
pm2 save
pm2 startup
pm2 logs (name)
```

### 🌐 NGINX

```bash
nginx -t
systemctl reload nginx
```

### 🔐 SSL Renew Test

```bash
certbot renew --dry-run
```