# Creating a non-interactive shell
1. A non-interactive shell is a shell that does not allow a user to log in and start an interactive command session. It's commonly assigned to system accounts, service accounts, or users that need to exist on the system but should not be able to log in via SSH or a terminal.
2. cat /etc/hosts -> to list all hosts
3. sudo useradd -s /sbin/nologin required-name
4. grep '^required-name:' /etc/passwd -> to view the access of the user

# Creating an expiring date for a user
1. sudo useradd -e <date> <user-name>
2. verify -> sudo chage -l <user-name>

# Disable direct SSH root login
1. ssh username@servername
2. sudo vi /etc/ssh/sshd_config
3. Make the change -> PermitRootLogin no
4. Restart SSH -> sudo systemctl restart sshd
5. Verify -> grep "^PermitRootLogin" /etc/ssh/sshd_config
