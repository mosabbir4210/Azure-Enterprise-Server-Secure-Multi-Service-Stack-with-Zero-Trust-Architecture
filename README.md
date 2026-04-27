📌 Project Abstract
This project showcases a high-performance, multi-functional server infrastructure deployed on Microsoft Azure. The architecture transitions away from traditional, vulnerable hosting methods by implementing a Zero-Port Exposure strategy. Using AlmaLinux 9, CyberPanel, and Cloudflare Zero Trust, this stack handles Web, Mail, and Database services while keeping the origin IP completely hidden from the public internet.

🏗️ Part 1: Infrastructure Design & Provisioning
The foundation is built on an enterprise-grade Linux distribution to ensure long-term stability and security.

Compute: Microsoft Azure Standard B2s (2 vCPU, 4GB RAM).

Operating System: AlmaLinux 9 (Gen 2).

Storage: 30GB Premium SSD.

Network Security Group (NSG): * Inbound: All ports (80, 443, 8090) are CLOSED by default.

Management: Port 22 (SSH) is restricted to a specific Administrative IP.

Part 2: Core Stack & Web Hosting
We utilized CyberPanel for its native OpenLiteSpeed integration, providing superior performance compared to traditional Apache/Nginx setups.

1. Server Environment Setup
Bash
# Updating system packages
sudo dnf update -y
sudo dnf install wget curl nano tar epel-release -y

# CyberPanel Installation with PHP 8.2 & OpenLiteSpeed
sh <(curl https://cyberpanel.net/install.sh || wget -O - https://cyberpanel.net/install.sh)
2. Web Hosting Features
Runtime: PHP 8.2 (optimized for modern frameworks).

Database: MariaDB (MySQL) with optimized query caching.

LSCache: Implemented server-level caching for sub-second page load times on the hosted Portfolio Website.
