## 🚀 Azure Enterprise Server: Secure Multi-Service Stack with Zero-Trust Architecture ##

Project Abstract
This project showcases a high-performance, multi-functional server infrastructure deployed on Microsoft Azure. The architecture transitions away from traditional, vulnerable hosting methods by implementing a Zero-Port Exposure strategy. Using AlmaLinux 9, CyberPanel, and Cloudflare Zero Trust, this stack handles Web, Mail, and Database services while keeping the origin IP completely hidden from the public internet.

🏗️ Infrastructure Design & Provisioning
Compute: Microsoft Azure Standard B2s (2 vCPU, 4GB RAM)

OS: AlmaLinux 9 (Gen 2)

Storage: 30GB Premium SSD

Networking: Azure NSG (All inbound ports closed by default; Port 22 restricted to Admin IP)

# Part 2: Core Stack & Web Hosting
We utilized CyberPanel for its native OpenLiteSpeed integration, providing superior performance compared to traditional Apache/Nginx setups.

## 🏗️ Phase 1: Infrastructure & OS Hardening
Before installing any services, the AlmaLinux 9 environment was secured and optimized.

# 1. Initial System Update & Utilities
```bash

# Update all system packages
sudo dnf update -y

# Install essential administration tools
sudo dnf install wget curl nano tar epel-release bzip2 -y

```
## 🔒 Phase 2: CyberPanel Installation with PHP 8.2 & OpenLiteSpeed

```bash

sh <(curl https://cyberpanel.net/install.sh || wget -O - https://cyberpanel.net/install.sh)

```
Selection during install:

Install CyberPanel with OpenLiteSpeed.

Install Full Service (DNS, Mail, FTP).

Remote MySQL: No.

Memcached/Redis: Yes (for performance).

## 🔒 Phase 3: Zero-Trust Security (Cloudflare Tunnel)
This is the core security feature that hides the server's Origin IP.

1. Install Cloudflared Daemon
```bash
# Download and install the latest Cloudflared RPM
curl -L --output cloudflared.rpm https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm
sudo dnf localinstall cloudflared.rpm -y
```
2. Tunnel Authentication & Creation
3. 
```bash
# Authenticate with Cloudflare
cloudflared tunnel login

```

# Create the Tunnel
```bash

cloudflared tunnel create azure-enterprise-tunnel

# Verify the tunnel list
cloudflared tunnel list

```
# 3. Routing Traffic (Ingress Rules)
I configured the config.yml to map subdomains to internal ports:

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

# On AlmaLinux 9, Nginx is managed via systemctl. These are the daily operational commands:

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

In a professional setup, you need to know where the important files live:

Main Configuration: /etc/nginx/nginx.conf

Server Blocks (vHosts): /etc/nginx/conf.d/ (This is where you create .conf files for your domains)

Default Web Root: /usr/share/nginx/html

Log Files: /var/log/nginx/access.log and /var/log/nginx/error.log


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

If you were to use Nginx as a proxy for your CyberPanel or a Node.js app, the command to create the config would be:

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

# In an enterprise environment, PHP must be tuned for security and speed. We utilized lsphp (LiteSpeed PHP) to ensure seamless integration with the OpenLiteSpeed engine.

# 1. Installation via CyberPanel CLI
CyberPanel usually handles the base installation, but you can manually install or add extensions using the following commands on AlmaLinux:

```bash

# Install PHP 8.2 and common extensions

sudo dnf install lsphp82 lsphp82-common lsphp82-mysqlnd lsphp82-gd lsphp82-process lsphp82-mbstring lsphp82-xml lsphp82-opcache -y

```

# 2. PHP Configuration (php.ini)
To handle professional web applications and large file uploads (like portfolio assets), we modified the php.ini settings.

File Path: /usr/local/lsws/lsphp82/etc/php.ini

# Edit the PHP configuration

sudo nano /usr/local/lsws/lsphp82/etc/php.ini

Key Optimizations Applied:

memory_limit = 256M (Ensures enough RAM for heavy scripts)

upload_max_filesize = 64M (Allows high-quality image uploads)

post_max_size = 64M

max_execution_time = 300

date.timezone = Asia/Dhaka (Setting local time for logs)


## 3. Enabling OPcache for High Speed
OPcache improves PHP performance by storing precompiled script bytecode in shared memory.

```bash
# Ensure OPcache is enabled in the config
zend_extension=opcache.so
opcache.enable=1
opcache.memory_consumption=128

```

# 4. Verifying PHP Status
You can verify which version is active and check for installed modules using these commands:

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
## Database Management (MariaDB/MySQL) ##

# In this project, MariaDB (a community-developed fork of MySQL) was used as the primary database engine due to its performance and compatibility with AlmaLinux 9.

# 1. Service Administration
   
```bash

# Check if MariaDB service is active
sudo systemctl status mariadb

# Start and Enable MariaDB on boot
sudo systemctl enable --now mariadb

# Check MariaDB version
mysql -V

```
# 2. Security Hardening

After installation, I ran the security script to remove anonymous users and secure the root access:

```bash

# Secure the installation
sudo mysql_secure_installation

#Actions taken: Set root password, removed anonymous users, disallowed root login remotely, and removed the test database.


## 📧 Enterprise Mail Service Architecture ## 
This phase covers the installation of the Mail Transfer Agent (MTA), security authentication, and bypassing Azure's outbound restrictions.

# 1. Core Mail Components
Postfix: Handles sending and receiving emails (MTA).

Dovecot: Manages mail storage and allows users to access mail via IMAP/POP3 (MDA).

SnappyMail: The lightweight, modern webmail interface.

# 2. Service Management Commands

To ensure the mail services are running correctly on your server:

```bash

# Check status of Postfix and Dovecot

sudo systemctl status postfix
sudo systemctl status dovecot

# Restart services after configuration changes
sudo systemctl restart postfix dovecot

```
# 3. DNS Engineering for Deliverability
Setting up the software is only 50% of the job. The other 50% is configuring DNS to ensure emails reach the Inbox instead of the Spam folder.

SPF (Sender Policy Framework): In your DNS provider (Cloudflare), add a TXT record:
v=spf1 ip4:[Your_Azure_Public_IP] +a +mx ~all

DKIM (DomainKeys Identified Mail): Generated via CyberPanel (Mail > DKIM Manager). This adds a digital signature to every email sent.

DMARC: A policy to tell receiving servers what to do if SPF/DKIM fails:
v=DMARC1; p=quarantine; adkim=r; aspf=r

# 4. SSL/TLS Encryption for Mail
Emails must be encrypted during transit. I used Let's Encrypt via CyberPanel to secure the mail server hostname.

```bash

# Command to check if the mail certificate is valid
openssl s_client -connect mail.yourdomain.com:465

```
# 5. Handling Azure Port 25 Restrictions
By default, Microsoft Azure blocks outbound traffic on Port 25 to prevent spam. To solve this, we practiced two methods:

Requesting Unblock: Filing a request with Azure Support to open Port 25.

Using Submission Ports: Configuring the mail client to use Port 587 (STARTTLS) or Port 465 (SSL/TLS) for authenticated sending.

# 6. Mail Server Testing (CLI Tools)
As a professional, you should test your server using the command line:

```bash

# Testing SMTP connectivity manually
telnet localhost 25

# Checking the mail logs in real-time for errors
sudo tail -f /var/log/maillog

```

