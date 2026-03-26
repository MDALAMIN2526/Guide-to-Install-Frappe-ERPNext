# [Guide] How to install ERPNext v16 on Linux Ubuntu 24.04 (step-by-step instructions)

## 📋 Table of Contents
1. [Server Setup](#step-1-server-setup)
2. [Install Required Packages](#step-2-install-required-packages)
3. [Configure MySQL Server](#step-3-configure-mysql-server)
4. [Install Node, NPM, Yarn](#step-4-install-node-npm-yarn)
5. [Initialize Frappe Bench](#step-5-initialize-frappe-bench)
6. [Create New Site & Install Frappe Framework](#step-6-create-new-site--install-frappe-framework)
7. [Setup Production Environment](#step-7-setup-production-environment)
8. [Install ERPNext and other Apps](#step-8-install-erpnext-and-other-apps)
9. [Custom Domain & SSL Setup](#step-9-custom-domain--ssl-setup)

## 🛠 Pre-Requisites
* **Operating System:** Linux Ubuntu 24.04 LTS
* **Access:** SSH access to the server
* **Software requirements:**
    * Python 3.14
    * Node 24
    * MariaDB 11.8
    * Redis 6+
    * Yarn 1.22+
    * Pip 25.3+

## STEP 1: SERVER SETUP

### 1.1 Login to the server using SSH

### 1.2 Setup correct date and timezone
Check the server’s default timezone:
```bash
date
```
To set a different timezone (e.g., America/Los_Angeles):
```bash
sudo timedatectl set-timezone "America/Los_Angeles"
```

### 1.3 Update & upgrade system packages
```bash
sudo apt-get update -y
sudo apt-get upgrade -y
```

### 1.4 Create a new user
Create a user with sudo privileges to avoid using root. Replace `[frappe-user]` with your username.
```bash
sudo adduser [frappe-user]
sudo usermod -aG sudo [frappe-user]
su [frappe-user]
cd /home/[frappe-user]/
```

## STEP 2: INSTALL REQUIRED PACKAGES

### 2.1 Install Git
```bash
sudo apt-get install git -y
git --version
```

### 2.2 Install cURL
```bash
sudo apt-get install curl -y
curl --version
```

### 2.3 Install Python
Frappe v16 recommends using the **uv** package manager.
```bash
sudo apt-get install python3-dev python3-pip python3-setuptools python3-venv -y

cd /home/[frappe-user]/
curl -LsSf [https://astral.sh/uv/install.sh](https://astral.sh/uv/install.sh) | sh
source ~/.bashrc

uv python install 3.14 --default
```

### 2.4 Other required packages
```bash
sudo apt-get install software-properties-common xvfb libfontconfig libmysqlclient-dev pkg-config -y
```

### 2.5 Install Redis Server
```bash
sudo apt-get install redis-server -y
redis-server --version
```

### 2.6 Install wkhtmltopdf
```bash
sudo wget [https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_arm64.deb](https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_arm64.deb)
# Note: Use amd64.deb if on x86 architecture
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_arm64.deb
sudo apt-get -f install -y
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_arm64.deb
```

---

## STEP 3: CONFIGURE MySQL SERVER

### 3.1 Install MariaDB 11.8+
```bash
sudo apt install mariadb-server mariadb-client -y
mysql --version
```

### 3.2 Secure MariaDB
```bash
sudo mysql_secure_installation
```
* **Switch to unix_socket authentication:** Y
* **Change the root password:** Y
* **Remove anonymous users:** Y
* **Disallow root login remotely:** N
* **Remove test database:** Y
* **Reload privilege tables:** Y

### 3.3 Update MariaDB config
```bash
sudo nano /etc/mysql/my.cnf
```
Add the following to the end of the file:
```ini
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

### 3.4 Restart MariaDB
```bash
sudo service mysql restart
```

---

## STEP 4: INSTALL Node, NPM, Yarn

### 4.1 Install Node 24 via NVM
```bash
cd /home/[frappe-user]/
curl [https://raw.githubusercontent.com/creationix/nvm/master/install.sh](https://raw.githubusercontent.com/creationix/nvm/master/install.sh) | bash
source ~/.profile
nvm install 24
nvm use 24
```

### 4.2 Install NPM and Yarn
```bash
sudo apt-get install npm -y
sudo npm install -g yarn
```

---

## STEP 5: INITIALIZE FRAPPE BENCH

### 5.1 Install Frappe Bench via uv
```bash
uv tool install frappe-bench
bench --version
```

### 5.2 Initialize Bench
```bash
cd /home/[frappe-user]/
bench init --frappe-branch version-16 frappe-bench
```

### 5.3 Set permissions
```bash
cd /home/[frappe-user]/frappe-bench/
sudo chmod -R o+rx /home/[frappe-user]/
```

---

## STEP 6: CREATE NEW SITE & INSTALL FRAPPE
```bash
bench new-site site1.local
```
*(You will be prompted to set the Administrator password)*

---

## STEP 7: SETUP PRODUCTION ENVIRONMENT

### 7.1 Enable Scheduler & Disable Maintenance
```bash
bench --site site1.local enable-scheduler
bench --site site1.local set-maintenance-mode off
```

### 7.2 Production Setup via Ansible
```bash
sudo apt install -y ansible
sudo env "PATH=$PATH" bench setup production [frappe-user]
```

### 7.3 Setup NGINX & Supervisor
```bash
bench setup nginx
sudo supervisorctl restart all
sudo supervisorctl status
```

---

## STEP 8: INSTALL ERPNEXT AND OTHER APPS

### 8.1 Fetch apps
```bash
bench get-app --branch version-16 erpnext
bench get-app --branch version-16 hrms
```

### 8.2 Install apps on site
```bash
bench --site site1.local install-app erpnext
bench --site site1.local install-app hrms
```

---

## STEP 9: CUSTOM DOMAIN & SSL SETUP

### 9.1 Enable DNS multi-tenancy
```bash
bench config dns_multitenant on
```

### 9.2 Add Domain
Ensure your "A" record points to the server IP.
```bash
bench setup add-domain [subdomain.yourdomain.com] --site site1.local
bench setup nginx 
sudo service nginx reload
```

### 9.3 Install SSL (Certbot)
```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

---

### 🛡️ Firewall Settings
```bash
sudo ufw allow 22,25,143,80,443,3306,3022,8000/tcp
sudo ufw enable
```

**Installation Complete!** Access your site at `http://[server-ip]` or your domain.
```

Would you like me to help you generate any specific configuration files or scripts based on this guide?
