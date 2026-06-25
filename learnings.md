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

## Create a cron job
1. Install package and enable -> sudo yum install -y cronie && sudo systemctl enable --now crond
2. ssh into server
3. Edit the root crontab -> sudo crontab -e
4. Add the cron job code.
5. Verify -> sudo crontab -l
6. cron format -> 
* * * * *  command
│ │ │ │ │
│ │ │ │ └── Day of week (0–7)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)

## Install SELINUX packages
1. SSH into the server
2. Install SELINUX packages -> sudo (yum or dnf) install -y selinux-policy selinux-policy-targeted policycoreutils
3. Permanently disable SELinux -> sudo vi /etc/selinux/config. Set SELINUX=diabled
4. Verify -> grep ^SELINUX= /etc/selinux/config
