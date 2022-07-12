# linux-server-hardening-checklist
Checklist for hardening a Linux server that includes network and kernel hardening, BIOS protection, disk encryption, disk protection, boot directory permissions, USB usage, system updates, removing unnecessary packages, closing unnecessary ports, securing SSH protocol, enabling SELinux, password policies, permissions and verifications, and additional process hardening.

## Linux Server Hardening

### 15-step Checklist for a secure Linux Server

#### Information
- Machine Name:
	- <hostname>

- IP Address:
	- Local: <local ip>
	- Public: <public ip>

- MAC Address:
	- <MAC address>

- Servicing Technician:
	- <full name>

- Date:
	- <date>

- Asset Number:
	- <asset number>

#### Hardening Checklist

1. [] BIOS Protection
	- [] Set Admin Password
	- [] Disable booting from external storage device
	- [] **Check for MOBO internal web server and change default password**

2. [] Hard Disk Encryption
	- [] Set up encrypted LVM
	- **If distribution doesn't support encryption, software like TrueCrypt is available**

3. [] Disk Protection
	- [] Create Managed Backups
		- [] Important Servers need the backups to be transferred offsite
	- [] Critical systems should be separated into different partitions
		- [] /
		- [] /boot
		- [] /usr
		- [] /home
		- [] /tmp
		- [] /var
		- [] /opt

4. [] Lock the boot directory to read-only permissions
	- [] Edit fstab /boot entry with mount options "defaults,ro"
	- [] Check that fstab ownership is "root:root"
	- [] Set grub.conf ownership is "root:root"
		- Run `# chown root:root /etc/grub.conf`
	- [] Set permissions for grub.conf to read/write for root only
		- Run `# chmod og-rwx /etc/grub.conf`
	- [] Require authentication for single-user mode
		- Run `# sed -i "/SINGLE/s/sushell/sulogin/" /etc/sysconfig/init`
		- Run `# sed -i "/PROMPT/s/yes/no/" /etc/sysconfig/init`

5. [] (Optional) Disable USB usage
	- [] Edit "/etc/modprobe.d/blacklist.conf" file
		- [] Add the following line: `blacklist usb_storage`
	- [] Edit "/etc/rc.local"
		- [] Add the following lines:
			- ```\n
			  modprobe -r usb_storage\n
			  exit 0\n
			  ```\n

6. [] System Update
	- [] `#apt update && apt upgrade -y && apt autoremove -y`

7. [] Check the installed packages
	- [] List all of the installed packages and remove anything unnecessary
		- Run `# apt-cache pkgnames`
	- [] List of important packages to remove:
		- Telnet server
		- RSH server
		- NIS server
		- TFTP server
		- TALK server

8. [] Check for open ports
	- [] Run `# netstat -antp`

9. [] Secure SSH
	- [] Edit "/etc/ssh/sshd_config"
		- [] Change default port
		- [] Disable root login
			- Add the following line `PermitRootLogin no`
		- [] Allow specific users
			- Add the following line `AllowUsers [username]` (comma separated list of users)
		- [] Additional "sshd_config" options
			- `Protocol2`
			- `IgnoreRHosts yes`
			- `HostbasedAuthentication no`
			- `PermitEmptyPasswords no`
			- `X11Forwarding no`
			- `MaxAuthTries 5`
			- `Ciphers aes128-ctr.aes192-ctr,aes256-ctr`
			- `ClientAliveInterval 900`
			- `ClientAliveCountMax 0`
			- `UsePAM yes`
		- [] Set ownership and permissions on "sshd_config" so that only root users can change contents
			- Run `# chown root:root /etc/ssh/sshd_config`
			- Run `# chmod 600 /etc/ssh/sshd_config`

10. [] Enable SELinux
	- [] Edit file "/etc/selinux/config"
		- Add the following line `SELINUX=enforcing`

11. [] Network parameters
	- [] Disable IP Forwarding
		- Add the following line `net.ipv4_forward=0` to "/etc/sysctl.conf"
	- [] Disable the Send Packet Redirects
		- Add the following lines:
			```\n
			net.ipv4.conf.all.send_redirects=0\n
			net.ipv4.conf.default.send_redirects=0\n
			```\n
		  to "/etc/sysctl.conf
	- [] Disable ICMP Redirects Acceptance
		- Add the following lines:
			```\n
			net.ipv4.conf.all.accept_redirects=0\n
			net.ipv4.conf.default.accept_redirects=0\n
			```\n
		  to "/etc/sysctl.conf
	- [] Enable Bad Error Message Protection
		- Add the following line `net.ipv4.icmp_ignore_bogus_error_responses=1` to "/etc/sysctl.conf"
	- [] (Optional, but strongly recommended) Configure Linux Firewall using iptables 
		- [] Applying iptable rules and filtering all the incoming, outgoing and forwarded requests

12. [] Password policies
	- Add the following lines:
		```\n
		auth	sufficient	pam_unix.so likeauth nullok\n
		password	sufficient	pam_unix.so remember=4\n
		```\n
	  to "/etc/pam.d/common-password"
	- [] To protect against dictionary and brute-force attacks
		- [] Add the following line `/lib/security/$ISA/pam_cracklib.so retry=3 minlen=8 lcredit=-1 ucredit=-2 dcredit=-2 ocredit=-1` to "/etc/pam.d/system-auth"
	- [] Lock account after 5 failed attempts
		- [] Add the following lines:
			```\n
			auth required pam_env.so\n
			auth required pam_faillock.so preauth audit silent deny=5 unlock_timer=604800\n
			auth [success=1 default=bad] pam_unix.so\n
			auth [default=die] pam_faillock.so authfail audit deny=5 unlock_timer=604800\n
			auth sufficient pam_faillock.so authsucc audit deny=5 unlock_timer=604800\n
			auth required pam_deny.so\n
			```\n
		  to "/etc/pam.d/password-auth"
		- **After 5 failed attempts only an administrator can unlock account with the following command:**
			- Run `# /usr/sbin/faillock --user <userlocked> --reset`
	- [] Set user password expiry
		- [] Add the following line: `PASS_MAX_DAYS=90` to "/etc/login.defs"
		- [] Change active users with following command: Run `# chage -maxdays 90 <user>`
	- [] Restrict access to the **`su`** command
		- [] Add the following line: `auth required pam_wheel.so use_uid` to "/etc/pam.d/su"
	- [] Disable the system accounts for non-root users with the following script:
		- ```\n
		  #!/bin/bash\n
		  for user in `awk -F: '($3 < 500) { print $1 }' /etc/passwd`; do\n
		  if [ $user != "root" ]\n
		  then\n
			/usr/sbin/usermod -L $user\n
		  	if [ $user != "sync" ] && [ $user != "shutdown" ] && [ $user != "halt" ]\n
		  	then\n
				/usr/sbin/usermod -s /sbin/nologin $user\n
		  	fi\n
		  fi\n
		  done\n
		  ```\n

13. Permissions and Verifications
	- [] Set User/Group Owner and Permissions on "/etc/anacrontab", "/etc/crontab" and "/etc/cron.*"
		- Run the following commands:
			```
			# chown root:root /etc/anacrontab
			# chmod og-rwx /etc/anacrontab
			# chown root:root /etc/crontab
			# chmod og-rwx /etc/crontab
			# chown root:root /etc/cron.hourly
			# chmod og-rwx /etc/cron.hourly
			# chown root:root /etc/cron.daily
			# chmod og-rwx /etc/cron.daily
			# chown root:root /etc/cron.weekly
			# chmod og-rwx /etc/cron.weekly
			# chown root:root /etc/cron.d
			# chmod og-rwx /etc/cron.d
			```
	- [] Set the right and permissions on "/var/spool/cron" for "root crontab"
		- Run the following commands:
			```
			# chown root:root <crontabfile>
			# chmod og-rwx <crontabfile>
			```
	- [] Set User/Group Owner and Permission on "passwd" file
		- Run the following commands:
			```
			# chmod 644 /etc/passwd
			# chown root:root /etc/passwd
			```
	- [] Set User/Group Owner and Permission on the "group" file
		- Run the following commands:
			```
			# chmod 644 /etc/group
			# chown root:root /etc/group
			```
	- [] Set User/Group Owner and Permission on the "shadow" file
		- Run the following commands:
			```
			# chmod 600 /etc/shadow
			# chown root:root /etc/shadow
			```
	- [] Set User/Group Owner and Permission on the "gshadow" file
		- Run the following commands:
			```
			# chmod 600 /etc/gshadow
			# chown root:root /etc/gshadow
			```

14. - [] Additional process hardening
	- [] Restrict Core Dumps
		- [] Adding `hard core 0` to the "/etc/security/limits.conf" file
		- [] Adding `fs.suid_dumpable = 0` to the "/etc/sysctl.conf" file
	- [] Configure Exec Shield
		- [] Adding `kernel.exec-shield = 1` to the "/etc/sysctl.conf" file
	- [] Enable randomized Virtual Memory Region Placement
		- [] Adding `kernel.randomize_va_space = 2` to the "/etc/sysctl.conf" file
