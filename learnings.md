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
