## 🚀 Azure Enterprise Server: Secure Multi-Service Stack with Zero-Trust Architecture ##

Project Abstract

This project showcases a high-performance, multi-functional server infrastructure deployed on **Microsoft Azure**. The architecture transitions away from traditional, vulnerable hosting methods by implementing a Zero-Port Exposure strategy. Using **AlmaLinux 9**, **CyberPanel**, and **Cloudflare Zero Trust**, this stack handles Web, Mail, and Database services while keeping the origin IP completely hidden from the public internet.

## 🏗️ Infrastructure Design & Provisioning ##


# 🖥️ Server Environment
- **OS: Compute:** Microsoft Azure Standard B2s (2 vCPU, 4GB RAM)
- **OS:** AlmaLinux 9 (Gen 2)
- **Storage:** 30GB Premium SSD
- **Cloud Provider:** Microsoft Azure
- **Networking:** Azure NSG configured
  - All inbound ports blocked by default
  - Port 22 (SSH) restricted to Admin IP only

# Core Stack & Web Hosting
**We utilized CyberPanel for its native OpenLiteSpeed integration, providing superior performance compared to traditional Apache/Nginx setups**.

## 🏗️ Phase 1: Infrastructure & OS Hardening
Before installing any services, the AlmaLinux 9 environment was secured and optimized.

# 1. Initial System Update & Utilities
```bash

# Update all system packages
sudo dnf update -y

# Install essential administration tools
sudo dnf install wget curl nano tar epel-release bzip2 -y

```
## 🔒 CyberPanel Installation with PHP 8.2 & OpenLiteSpeed

To build a complete web hosting environment, CyberPanel was installed with OpenLiteSpeed and PHP 8.2 for better speed, security, and management.

### 💻 Installation Command

```bash
sh <(curl https://cyberpanel.net/install.sh || wget -O - https://cyberpanel.net/install.sh)

```
# ⚙️ Recommended Installation Selections

| Option               | Selected Value    | Purpose                                 |
| -------------------- | ----------------- | --------------------------------------- |
| Web Server           | **OpenLiteSpeed** | High-performance lightweight web server |
| Panel Type           | **CyberPanel**    | Web hosting control panel               |
| Full Service Install | **Yes**           | Includes DNS, Mail, FTP services        |
| Remote MySQL         | **No**            | Uses local database server              |
| Memcached            | **Yes**           | Improves caching performance            |
| Redis                | **Yes**           | Fast object caching / session handling  |
| PHP Version          | **8.2**           | Modern PHP version for latest apps      |

## 🔒: Zero-Trust Security (Cloudflare Tunnel)
This is the core security feature that hides the server's Origin IP.

1. Install Cloudflared Daemon
```bash
# Download and install the latest Cloudflared RPM
curl -L --output cloudflared.rpm https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm
sudo dnf localinstall cloudflared.rpm -y
```
# 2. Tunnel Authentication & Creation

```bash
# Authenticate with Cloudflare
cloudflared tunnel login

```
```bash

# Create the Tunnel
cloudflared tunnel create azure-enterprise-tunnel

# Verify the tunnel list
cloudflared tunnel list

```
# 3. Routing Traffic (Ingress Rules)
**I configured the config.yml to map subdomains to internal ports**:

```bash
YAML
ingress:
  - hostname: panel.yourdomain.com
    service: http://localhost:8090
  - hostname: yourdomain.com
    service: http://localhost:80
  - service: http_status:404
```
# 4. Run as a System Service

```bash
# Install the tunnel as a background service
sudo cloudflared service install [Your-Tunnel-Token]
sudo systemctl start cloudflared
sudo systemctl enable cloudflared

```

## 🛠️ Nginx Service Management ##

 **On AlmaLinux 9, Nginx is managed via systemctl. These are the daily operational commands**:

```bash

# Install Nginx (if not already present)
sudo dnf install nginx -y

# Start and Enable Nginx to run on boot
sudo systemctl enable --now nginx

# Check the current status of the server
sudo systemctl status nginx

# Reload Nginx (Use this after changing configs to avoid downtime)
sudo systemctl reload nginx

# Stop Nginx
sudo systemctl stop nginx

```
# 🔍 Configuration & Syntax Testing
Before restarting Nginx, you must always verify that your code is correct to prevent the server from crashing.

```bash

# Test Nginx configuration for syntax errors
sudo nginx -t

# Check the Nginx version and compiled modules
nginx -V

```

## 📂 File Paths & Directory Structure

A professional Nginx deployment should clearly document where critical files and folders are located:

| Component | Path | Purpose |
|-----------|------|---------|
| Main Configuration | `/etc/nginx/nginx.conf` | Core Nginx settings and global directives |
| Server Blocks (vHosts) | `/etc/nginx/conf.d/` | Store individual `.conf` files for each domain / virtual host |
| Default Web Root | `/usr/share/nginx/html` | Default location for website files |
| Access Log | `/var/log/nginx/access.log` | Records incoming client requests |
| Error Log | `/var/log/nginx/error.log` | Stores warnings, errors, and troubleshooting logs |

> 💡 Keeping these paths organized helps with maintenance, troubleshooting, and multi-site hosting.


## 🛡️ Security & Hardening Commands

For Security , you should demonstrate "Security by Obscurity."

```bash
Hide Nginx Version:
Open the config: sudo nano /etc/nginx/nginx.conf
Add this inside the http {} block:

Nginx
server_tokens off;

```
```bash
#Allow Nginx through the local firewall:

sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

```
## 🔄 Reverse Proxy Snippet 

**If you were to use Nginx as a proxy for your CyberPanel or a Node.js app, the command to create the config would be**:

```bash

sudo nano /etc/nginx/conf.d/proxy.conf

#Then paste this professional block:

Nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8090; # Forwarding to CyberPanel
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

```
## PHP 8.2 Installation & Optimization ##

**In an enterprise environment, PHP must be tuned for security and speed. We utilized lsphp (LiteSpeed PHP) to ensure seamless integration with the OpenLiteSpeed engine**.

# 1. Installation via CyberPanel CLI
CyberPanel usually handles the base installation, but you can manually install or add extensions using the following commands on AlmaLinux:

```bash

# Install PHP 8.2 and common extensions

sudo dnf install lsphp82 lsphp82-common lsphp82-mysqlnd lsphp82-gd lsphp82-process lsphp82-mbstring lsphp82-xml lsphp82-opcache -y

```

## ⚙️ PHP Configuration (`php.ini`)

To support professional web applications and larger file uploads (such as portfolio assets, images, and project files), the PHP configuration was optimized.

### 📄 File Path

`/usr/local/lsws/lsphp82/etc/php.ini`

### ✏️ Edit the PHP Configuration

```bash
sudo nano /usr/local/lsws/lsphp82/etc/php.ini

```
# 🔧 Key Optimizations Applied

| Setting               | Value        | Purpose                                       |
| --------------------- | ------------ | --------------------------------------------- |
| `memory_limit`        | `256M`       | Ensures enough RAM for heavy scripts          |
| `upload_max_filesize` | `64M`        | Allows high-quality image uploads             |
| `post_max_size`       | `64M`        | Supports larger form submissions              |
| `max_execution_time`  | `300`        | Prevents timeout during long processes        |
| `date.timezone`       | `Asia/Dhaka` | Sets local timezone for logs and applications |


## 3. Enabling OPcache for High Speed
OPcache improves PHP performance by storing precompiled script bytecode in shared memory.

```bash
# Ensure OPcache is enabled in the config
zend_extension=opcache.so
opcache.enable=1
opcache.memory_consumption=128

```

# 4. Verifying PHP Status
**You can verify which version is active and check for installed modules using these commands**:

```bash

# Check PHP version
/usr/local/lsws/lsphp82/bin/php -v

# List installed PHP modules
/usr/local/lsws/lsphp82/bin/php -m

```
# 5. Restarting PHP Services
Whenever you change php.ini, you must kill the existing PHP processes so the changes take effect:

```bash
# Kill old PHP processes to apply new settings

killall lsphp

```
## 🗄️ Database Management (MariaDB / MySQL)

In this project, **MariaDB** — a community-developed fork of MySQL — was used as the primary database engine due to its strong performance, stability, and full compatibility with AlmaLinux 9.

### 💡 Why MariaDB?

- High performance for web applications  
- Fully compatible with MySQL syntax and tools  
- Reliable for production environments  
- Open-source and actively maintained  
- Optimized for Linux hosting servers  

---

## ⚙️ Service Administration

Database services were managed through `systemctl` to monitor status, start, stop, and restart operations when required.

```bash
# Check database service status
sudo systemctl status mariadb

# Start service
sudo systemctl start mariadb

# Restart service
sudo systemctl restart mariadb

# Enable on boot
sudo systemctl enable mariadb
# Start and Enable MariaDB on boot
sudo systemctl enable --now mariadb

# Check MariaDB version
mysql -V

```
# 2. Security Hardening

**After installation, I ran the security script to remove anonymous users and secure the root access**:

```bash

# Secure the installation
sudo mysql_secure_installation

#Actions taken: Set root password, removed anonymous users, disallowed root login remotely, and removed the test database.


## 📧 Enterprise Mail Service Architecture

This phase covers the deployment of core mail services, authentication readiness, and preparation for reliable outbound email delivery in a cloud environment.

---

## 🏗️ Core Mail Components

| Service | Role | Purpose |
|--------|------|---------|
| **Postfix** | MTA | Handles sending and receiving emails |
| **Dovecot** | MDA / IMAP / POP3 | Manages mailbox storage and user access |
| **SnappyMail** | Webmail Interface | Lightweight modern browser-based email client |

### 💡 Why This Stack?

- Reliable email delivery  
- Secure mailbox access  
- Webmail from any browser  
- Lightweight and production ready  

---

## ⚙️ Service Management Commands

To verify services are running correctly and restart after configuration changes:

```bash
# Check service status
sudo systemctl status postfix
sudo systemctl status dovecot

# Restart mail services
sudo systemctl restart postfix dovecot

```
# ✅ Operational Benefit

**This architecture provides a complete self-hosted business email platform with SMTP, IMAP, POP3, and webmail access**

## 🌐 DNS Engineering for Email Deliverability

Setting up the mail server software is only 50% of the job — the other 50% is proper DNS configuration to ensure emails land in the Inbox instead of Spam.

### 📌 Core DNS Records Configured

| Record Type | Purpose | Example Value |
|------------|---------|---------------|
| **SPF** | Authorizes your server to send emails for the domain | `v=spf1 ip4:[Your_Azure_Public_IP] +a +mx ~all` |
| **DKIM** | Adds a digital signature to outgoing emails for trust verification | Generated via CyberPanel → Mail → DKIM Manager |
| **DMARC** | Tells receiving servers how to handle SPF/DKIM failures | `v=DMARC1; p=quarantine; adkim=r; aspf=r` |

### 💡 Why It Matters

These records improve email reputation, reduce spoofing risk, and significantly increase inbox delivery rates.

---

## 🔒 SSL/TLS Encryption for Mail

Emails should be encrypted during transmission for privacy and trust.

I used **Let's Encrypt SSL** via CyberPanel to secure the mail server hostname.

### ✅ Benefits

- Encrypts SMTP / IMAP / POP3 traffic  
- Prevents interception during transit  
- Improves trust with receiving mail providers  
- Required for modern secure email delivery

```bash

# Command to check if the mail certificate is valid
openssl s_client -connect mail.yourdomain.com:465

```
## ☁️ Handling Azure Port 25 Restrictions

By default, Microsoft Azure restricts outbound traffic on **Port 25** to reduce spam and abuse risks.  
To ensure reliable email sending, two professional methods were practiced.

### 📌 Solutions Implemented

| Method | Description | Benefit |
|-------|-------------|---------|
| **Request Port 25 Unblock** | Submit a request to Azure Support for outbound SMTP access | Enables direct mail relay from the server |
| **Use Submission Ports** | Configure mail clients to use authenticated SMTP ports | More secure and commonly recommended |

### 🔧 Recommended SMTP Ports

| Port | Encryption | Usage |
|------|------------|------|
| `587` | STARTTLS | Standard authenticated mail submission |
| `465` | SSL/TLS | Secure SMTP connection |
| `25` | Plain / Relay | Server-to-server mail transfer |

### 💡 Best Practice

For modern hosting environments, using **Port 587** or **Port 465** is preferred for secure authenticated email delivery.

### ✅ Result

**This approach helps bypass cloud provider restrictions while maintaining secure and professional email operations**.

# 6. Mail Server Testing (CLI Tools)
**As a professional, you should test your server using the command line**:

```bash

# Testing SMTP connectivity manually
telnet localhost 25

# Checking the mail logs in real-time for errors
sudo tail -f /var/log/maillog

```

