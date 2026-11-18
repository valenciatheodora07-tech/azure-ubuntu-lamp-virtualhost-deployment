# azure-ubuntu-lamp-virtualhost-deployment
End-to-end provisioning, multi-user privilege management, LAMP stack deployment, Apache Virtual Host routing, automated daily backup, and ClamAV malware scanning on an Ubuntu 26.04 LTS Microsoft Azure VPS.
# Azure Ubuntu Server: Multi-User LAMP Stack, Virtual Host & Security Automation

A production-style Linux server administration and web hosting infrastructure project deployed on a **Microsoft Azure Virtual Private Server (VPS)** running **Ubuntu 26.04 LTS**[cite: 2, 11]. This project demonstrates end-to-end OS provisioning, role-based privilege separation, hardened LAMP stack deployment, custom Apache Virtual Host alias routing, SFTP publishing workflows, resource monitoring, and automated cron-based backup and antivirus pipelines[cite: 2, 11, 13, 14, 21, 42, 43, 45, 48].

---

## System Architecture & Specifications

* **Cloud Provider:** Microsoft Azure (Linux VPS)[cite: 2]
* **Operating System:** Ubuntu 26.04 LTS (`GNU/Linux x86_64`, minimized server installation with LVM partitioning)[cite: 4, 6, 11]
* **Web Server:** Apache HTTP Server `2.4.66` with custom Virtual Host (`reboot.id`)[cite: 13, 21]
* **Backend & Database:** PHP `8.5.4`, MariaDB `11.8.6`, and phpMyAdmin[cite: 13, 14, 16]
* **Remote Access & File Transfer:** OpenSSH Server (Port `22`) and SFTP (CLI `sftp`, WinSCP, FileZilla)[cite: 8, 18, 19, 40]
* **Security & Maintenance:** MariaDB Secure Hardening, ClamAV `1.5.3` Antivirus Engine, and Bash + `crontab` automation[cite: 14, 15, 43, 44, 46, 48]

---

## Key Features & Implementation

### 1. OS Provisioning & Privilege Separation
* **Minimalist OS Setup:** Installed Ubuntu Server (minimized) with LVM storage layout (`ext4` root volume), local Indonesian archive mirror (`id.archive.ubuntu.com`), and OpenSSH enabled for remote administration[cite: 4, 5, 6, 8].
* **Multi-User Access Control:** Configured a privilege hierarchy separating administrative (`root`/`sudo`) accounts from standard user accounts (`adduser`)[cite: 2, 11, 13]. Standard users are restricted from executing `sudo su` root escalation while retaining full write ownership over their isolated web directories[cite: 13, 20, 26].

### 2. Hardened LAMP Stack & Database Management
* **Web & PHP Runtime:** Deployed `apache2`, `php`, `libapache2-mod-php`, and `php-mysql`[cite: 13].
* **Database Security (`mariadb-secure-installation`):** Enforced root password authentication, removed anonymous users, disabled remote `root` login (restricted to `localhost`), removed the default `test` database, and reloaded privilege tables[cite: 14, 15].
* **phpMyAdmin Integration:** Configured phpMyAdmin with `dbconfig-common` and enabled the `mbstring` PHP extension for browser-based database administration[cite: 16, 17].

### 3. Apache Virtual Host & Multi-Tenant Path Routing
* Configured a dedicated Virtual Host (`/etc/apache2/sites-available/reboot.conf`) serving `reboot.id` and `www.reboot.id` with isolated `Alias` routes for five developer portfolio directories under `/var/www/`[cite: 21]:
  * `http://reboot.id/mahasiswa160425006` $\rightarrow$ `/var/www/160425006`[cite: 21, 29]
  * `http://reboot.id/mahasiswa160425040` $\rightarrow$ `/var/www/160425040`[cite: 21, 39]
  * `http://reboot.id/mahasiswa160425067` $\rightarrow$ `/var/www/160425067`[cite: 21, 36]
  * `http://reboot.id/mahasiswa160425072` $\rightarrow$ `/var/www/160425072`[cite: 21, 41]
  * `http://reboot.id/mahasiswa160425084` $\rightarrow$ `/var/www/160425084`[cite: 21, 34]
* Enforced directory ownership (`chown -R [USER]:www-data`) and permissions (`chmod 755`) so each user can independently deploy HTML/CSS and media assets over SFTP[cite: 18, 20].

### 4. Server Performance Monitoring (`htop`)
* Audited live server metrics under active web serving conditions[cite: 42]:
  * **CPU Load:** `0.0%` – `0.7%` across 2 active vCPU cores[cite: 42]
  * **Memory Usage:** `458 MB / 7.70 GB` (`~5.8%` utilization)[cite: 42]
  * **Process & Load Average:** `46` tasks (`55` user threads, `110` kernel threads) with load averages of `0.24` (1m), `0.14` (5m), and `0.10` (15m)[cite: 42]

### 5. Automated Backup & Antivirus Pipelines
* **Daily Website Backup with 7-Day Retention (`/var/backups/scripts/backup.sh`):** Automatically compresses every directory in `/var/www/` into dated `.tar.gz` archives and purges backups older than 7 days[cite: 42, 43].
* **Daily Malware Definition Update & Scan (`/var/scripts/scan.sh`):** Updates ClamAV virus signatures (`3.62M+` signatures) via `freshclam` and recursively scans `/var/*` for infected files[cite: 46, 47].

---

## Configuration & Automation Scripts

### Apache Virtual Host (`reboot.conf`)
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName reboot.id
    ServerAlias www.reboot.id
    DocumentRoot /var/www/

    Alias /mahasiswa160425067 /var/www/160425067
    Alias /mahasiswa160425006 /var/www/160425006
    Alias /mahasiswa160425040 /var/www/160425040
    Alias /mahasiswa160425072 /var/www/160425072
    Alias /mahasiswa160425084 /var/www/160425084

    <Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/reboot_error.log
    CustomLog ${APACHE_LOG_DIR}/reboot_access.log combined
</VirtualHost>
```[cite: 21, 22]

### Automated Website Backup (`backup.sh`)
```bash
#!/bin/bash

BACKUP_DIR="/var/backups"

mkdir -p "$BACKUP_DIR"

for site in /var/www/*; do
    name=$(basename "$site")
    mkdir -p "$BACKUP_DIR/$name"
    tar -czf "$BACKUP_DIR/$name/backup_$(date +%F).tar.gz" "$site"
done

find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete
```[cite: 43]

### Automated ClamAV Update & Scan (`scan.sh`)
```bash
#!/bin/bash
sudo systemctl stop clamav-freshclam
sudo freshclam
sudo systemctl start clamav-freshclam
clamscan -r /var/*
```[cite: 47]

### Crontab Schedule (`crontab -e`)
```cron
# Run daily website backup at 01:00 AM
0 1 * * * /var/backups/scripts/backup.sh

# Run daily ClamAV signature update & filesystem scan at 02:00 AM
0 2 * * * /var/scripts/scan.sh
```[cite: 44, 48]

---

## Quick Deployment Guide

1. **Install LAMP Stack & Security Packages:**
   ```bash
   sudo apt update
   sudo apt install apache2 php libapache2-mod-php php-mysql mariadb-server phpmyadmin clamav clamav-daemon -y
   sudo mariadb-secure-installation
   sudo phpenmod mbstring && sudo systemctl restart apache2
   ```[cite: 11, 13, 14, 16, 17, 45]
2. **Enable Custom Virtual Host:**
   ```bash
   sudo a2ensite reboot.conf
   sudo systemctl reload apache2
   ```[cite: 23]
3. **Configure Local Host Resolution (Client Machine):**
   Append the server's public IP mapping to `/etc/hosts` (Linux/macOS) or `C:\Windows\System32\drivers\etc\hosts` (Windows)[cite: 28, 38]:
   ```text
   <AZURE_VPS_PUBLIC_IP> reboot.id
   ```[cite: 28, 38]
4. **Activate Scheduled Maintenance:**
   ```bash
   sudo chmod +x /var/backups/scripts/backup.sh /var/scripts/scan.sh
   sudo crontab -e
   ```[cite: 43, 48]

---

## Team & Contributors (REBOOT — Group 6)

*Informatics Engineering, University of Surabaya (UBAYA) — 2026*[cite: 1]

| Name | Student ID | System Role | Key Responsibilities |
| :--- | :---: | :---: | :--- |
| **Keffka Mayloa** | `160425072`[cite: 1] | `root`[cite: 48] | OS installation, user provisioning, Apache Virtual Host setup, ClamAV activation, portfolio deployment[cite: 48] |
| **Livio Hardi** | `160425040`[cite: 1] | `root`[cite: 48] | Apache2, MariaDB, phpMyAdmin, and ClamAV installation & configuration, portfolio deployment[cite: 48] |
| **Margaret Christabel Robert** | `160425067`[cite: 1] | `root` / VM Owner[cite: 48] | Virtual Host configuration, `htop` performance monitoring, automated backup script & cron setup, CLI SFTP deployment[cite: 48] |
| **Valencia Theodora** | `160425006`[cite: 1] | `user`[cite: 48] | Portfolio web deployment, technical documentation standardization and editorial review[cite: 48] |
| **Annabelle Milka Seraphine** | `160425084`[cite: 1] | `user`[cite: 48] | Portfolio web deployment, GUI SFTP deployment (WinSCP), technical report formatting[cite: 48] |
