## Creating a non-interactive shell
1. A non-interactive shell is a shell that does not allow a user to log in and start an interactive command session. It's commonly assigned to system accounts, service accounts, or users that need to exist on the system but should not be able to log in via SSH or a terminal.
2. cat /etc/hosts -> to list all hosts
3. sudo useradd -s /sbin/nologin required-name
4. grep '^required-name:' /etc/passwd -> to view the access of the user

## Creating an expiring date for a user
1. sudo useradd -e <date> <user-name>
2. verify -> sudo chage -l <user-name>

## Disable direct SSH root login
1. ssh username@servername
2. sudo vi /etc/ssh/sshd_config
3. Make the change -> PermitRootLogin no
4. Restart SSH -> sudo systemctl restart sshd
5. Verify -> grep "^PermitRootLogin" /etc/ssh/sshd_config

## Grant executable permissions
1. SSH into the server
2. change executable permissions -> sudo chmod a+x <file-name (or) file-path>
3. Verify -> ls -l <file-name (or) file-path>

## Install SELINUX packages
1. SSH into the server
2. Install SELINUX packages -> sudo (yum or dnf) install -y selinux-policy selinux-policy-targeted policycoreutils
3. Permanently disable SELinux -> sudo vi /etc/selinux/config. Set SELINUX=diabled
4. Verify -> grep ^SELINUX= /etc/selinux/config

## Create a cron job
1. Install package and enable -> sudo yum install -y cronie && sudo systemctl enable --now crond
2. SSH into the server
3. Edit the root crontab -> sudo crontab -e
4. Add the cron job code.
5. Verify -> sudo crontab -l
6. cron format -> 
```text
* * * * *  command
│ │ │ │ │
│ │ │ │ └── Day of week (0–7)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

## Linux SSH key-based authentication to avoid a password for login
1. ssh-keygen -t rsa -> on the jump host for RSA key creation
2. ssh-copy-id username@servername

## Install Ansible using pip3 
1. sudo pip3 install ansible==4.9.0
2. ansible --version

## Troubleshooting MARIADB
1. SSH into the MariaDB server
2. See the error -> sudo journalctl -u mariadb -n 20 --no-pager
3. Create directory -> sudo mkdir -p /var/lib/mysql
4. Change the owner and group of the directory to the mysql user, so MariaDB has permission to access it -> sudo chown mysql:mysql /var/lib/mysql
5. Set the directory permissions: the owner (mysql) gets full access (rwx), while everyone else gets read and execute access (r-x) -> sudo chmod 755 /var/lib/mysql
6. Initialize MariaDB -> sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
7. Start MariaDB -> sudo systemctl start mariadb
8. Verify if it's active -> sudo systemctl is-active mariadb

## Day 10 DevOps Task
1. SSH into the server
2. Install zip -> sudo yum install -y zip
3. Make directory -> sudo mkdir -p /scripts
4. Permissions -> sudo chown user:user /scripts
5. Create the script -> vi /scripts/beta_archive.sh
6. Add the code ->
```
#!/bin/bash

zip -r /archives/xfusioncorp_beta.zip /var/www/html/beta
scp /archives/xfusioncorp_beta.zip natasha@ststor01:/archives/
```
7. Make it executable -> chmod +x /scripts/beta_archive.sh
8. Create the RSA key and add it to the storage server for passwordless access
```
ssh-keygen -t rsa
ssh-copy-id natasha@ststor01
```
9. Run the script -> /scripts/beta_archive.sh
10. Verify ->
```
ls -l /archives/xfusioncorp_beta.zip
ssh natasha@ststor01 "ls -l /archives/xfusioncorp_beta.zip"
```
## Install Tomcat server and configure it
1. Copy the .war file into the server -> scp /tmp/ROOT.war banner@stapp03:/tmp/
2. SSH into the server
3. Install the Tomcat server -> sudo yum install -y tomcat tomcat-webapps tomcat-admin-webapps
4. Change the port as needed -> sudo sed -i 's/port="8080"/port="own-port-number"/' /etc/tomcat/server.xml
5. Check if a ROOT directory already exists -> ls -l /var/lib/tomcat/webapps/
6. If it exists -> sudo rm -rf /var/lib/tomcat/webapps/ROOT
7. Copy into the folder path -> sudo cp /tmp/ROOT.war /var/lib/tomcat/webapps/
8. Access -> sudo chown tomcat:tomcat /var/lib/tomcat/webapps/ROOT.war
9. Start Tomcat server -> sudo systemctl start tomcat
10. curl URL -> check if the output gives the needed webpage.

## Fixing Apache issue with the port
1. SSH into the server
2. Check status -> sudo systemctl status httpd
3. Check if it is listening on the port needed -> sudo ss -tlnp | grep <port-number>
4. If the port is busy -> 127.0.0.1:8083 users:(("sendmail"...)) in something else, then follow the steps below.
5. ```
   sudo systemctl stop sendmail
   sudo systemctl disable sendmail
   ```
6. Start Apache
   ```
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```
7. Check the firewall -> sudo systemctl status iptables
8. Allow the port -> sudo iptables -I INPUT -p tcp --dport <port-number> -j ACCEPT

| Option | Meaning |
|--------|---------|
| `sudo` | Run the command with administrator (root) privileges. |
| `iptables` | Linux firewall management utility. |
| `-I INPUT` | **Insert** the rule at the beginning of the **INPUT** chain (incoming traffic). |
| `-p tcp` | Apply the rule only to the **TCP** protocol. |
| `--dport 8083` | Match traffic whose **destination port** is **8083**. |
| `-j ACCEPT` | **Accept** (allow) the matching traffic. |

9. Test from jump-host -> curl http://>server-name>:<port-number>

## Task 13 answer
1. SSH into any server and find the IP address of the Load Balancer -> getent hosts stlb01
2. Follow the steps below for all the servers:
   ```
   ssh tony@stapp01

   sudo yum install -y iptables iptables-services
   
   sudo iptables -I INPUT -p tcp -s <LBR_IP> --dport 6000 -j ACCEPT
   sudo iptables -A INPUT -p tcp --dport 6000 -j DROP
   
   sudo iptables-save | sudo tee /etc/sysconfig/iptables
   
   sudo systemctl enable iptables
   sudo systemctl restart iptables
   
   sudo iptables -L -n -> to verify
   ```
## Task 14 answer
1. Follow the steps
   ```
   ssh user@server-name

   sudo systemctl status httpd    #check the Apache service

   sudo ss -tlnp | grep 8082    #check the port and see if another service is running in its place

   #stop and disable the other service
   sudo systemctl stop sendmail
   sudo systemctl disable sendmail

   #start and enable apache
   sudo systemctl start httpd
   sudo systemctl enable httpd

   #recheck the Apache service
   sudo systemctl status httpd
   ```
## Task 15 Nginx setup
Prepare **App Server 2** for a new application by:

1. Installing **Nginx**.
2. Configuring **HTTPS (SSL)** using the provided certificate and key.
3. Creating a simple web page.
4. Verifying HTTPS access from the jump host.

---

1. SSH into App Server 2
2. Install Nginx

```bash
sudo yum install -y nginx
```
3. Create a location for SSL certificates

```bash
sudo mkdir -p /etc/pki/nginx
```

### Explanation

- `mkdir` → create directory.
- `-p` → create parent directories if they don't exist.
- `/etc/pki/nginx` → a standard location for storing SSL certificates on RHEL/CentOS systems.
4. Move the certificate and key

```bash
sudo mv /tmp/nautilus.crt /etc/pki/nginx/
sudo mv /tmp/nautilus.key /etc/pki/nginx/
```
5. Secure the private key. 600 -> only owner read and write access. 

```bash
sudo chmod 600 /etc/pki/nginx/nautilus.key
```
6. Configure SSL

The main `nginx.conf` file already contains:

```nginx
include /etc/nginx/conf.d/*.conf;
```

This means **every `.conf` file inside `/etc/nginx/conf.d/` is automatically loaded.**

Instead of editing the main configuration, create a new configuration file.

```bash
sudo vi /etc/nginx/conf.d/ssl.conf
```

Add:

```nginx
server {
    listen 443 ssl;
    server_name stapp02;

    ssl_certificate /etc/pki/nginx/nautilus.crt;
    ssl_certificate_key /etc/pki/nginx/nautilus.key;

    root /usr/share/nginx/html;
    index index.html;
}
```
7. Create the webpage

```bash
echo "Welcome!" | sudo tee /usr/share/nginx/html/index.html
```
8. Test the configuration

```bash
sudo nginx -t
```

Expected output:

```
syntax is ok
test is successful
```

9. Start Nginx

```bash
sudo systemctl enable --now nginx
```
Equivalent to:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

10. Verify locally

```bash
curl -Ik https://localhost
```

Expected:

```
HTTP/1.1 200 OK
```
11. Verify from Jump Host

```bash
curl -Ik https://stapp02/
```

Expected:

```
HTTP/1.1 200 OK
```

## Task 16 Nginx Load Balancer

1. Verify Apache on all App Servers

```bash
sudo systemctl status httpd
grep "^Listen" /etc/httpd/conf/httpd.conf
```

Apache must be **running** and listening on **port 5001**.

---

2. Install Nginx on LBR

```bash
sudo yum install -y nginx
```

---

3. Edit `/etc/nginx/nginx.conf`

Inside the `http {}` block, add:

```nginx
upstream app_servers {
    server stapp01:5001;
    server stapp02:5001;
    server stapp03:5001;
}
```

Replace the existing `server { ... }` block with:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name _;

    location / {
        proxy_pass http://app_servers;
    }
}
```

Remove:

- `root`
- `include /etc/nginx/default.d/*.conf`
- `error_page`
- `404/50x location` blocks

---
4. Test Configuration

```bash
sudo nginx -t
```

---

5. Start Nginx

```bash
sudo systemctl enable --now nginx
sudo systemctl restart nginx
```

---

6. Verify

From Jump Host:

```bash
curl http://stlb01:80
```

If the application page loads, the load balancer is working.

## Install and Configure PostgreSQL
# PostgreSQL - Create User and Database (Stratos Lab)

1. SSH to Database Server

2. Switch to the PostgreSQL user

```bash
sudo su - postgres
```

3. Open PostgreSQL bash

```bash
psql
```
4. Create the database user

```sql
CREATE USER <user-name> WITH PASSWORD '<password>';
```
5. Create the database

```sql
CREATE DATABASE <database-name>;
```
6. Grant all privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE <database-name> TO <user-name>;
```
7. Verify

List users:

```sql
\du
```

List databases:

```sql
\l
```

8. Exit

```sql
\q
```
