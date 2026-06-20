# Creating a non-interactive shell
1. A non-interactive shell is a shell that does not allow a user to log in and start an interactive command session. It's commonly assigned to system accounts, service accounts, or users that need to exist on the system but should not be able to log in via SSH or a terminal.
2. cat /etc/hosts -> to list all hosts
3. sudo useradd -s /sbin/nologin required-name
4. grep '^required-name:' /etc/passwd -> to view the access of the user
