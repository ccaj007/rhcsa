`dnf install -y setroubles* policycore*`

# Documentation
```
Mandb
Man -k
Info
/usr/share/doc
```
# SSH
```
ssh-keygen
ssh-copy-id user@server
ssh -p 22 user@server
```

# Repos
```
dnf config-manager --addrepo=http://reposerver.example.com/BaseOS
dnf config-manager --add-repo=file:///repo/BaseOS
```
edit the repository file in /etc/yum.conf.d after adding it, so that it includes the line gpgcheck=0
verify:
```
dnf repolist all
dnf install telnet
web / http
```
You will configure a web server running on your system serving content using a non-standard port (82)

- check if any service is running on that port 
curl http://localhost:82 
sudo ss -tlnp | grep ":82" 
sudo semanage port -l | grep http_port
journalctl will show semanage error and point you in right direction
Grub
# find / -type f -name grub
/etc/default/grub

Info grub2-mkconfig
grub-mkconfig -o /boot/grub2/grub.cfg
Password Policy
•	Password age is configured in /etc/login.defs
•	Password quality is configured in /etc/security/pwquality.conf
•	Easy stuff is under /etc/login.defs 
o	PASS_MAX_DAYS 90
o	PASS_MIN_DAYS 15
o	PASS_WARN_AGE 7
SSH Security
•	Configure it in /etc/ssh/sshd_conf
•	Enable root login with the PermitRootLogin yes value

SELinux
•	Install everything under keywords policycoreutils and setroubleshoot 
o	dnf install -y policycore* setrouble*
•	Make enforcement settings persistent in /etc/selinux/config
•	If you're troubleshooting httpd, for instance, you can find info at the following: 
o	grep httpd /var/log/messages
o	grep httpd /var/log/audit/audit.log
o	journalctl -u httpd
o	ausearch -c 'httpd' --raw
o	ausearch -p 1757 (assuming the broken PID from the other logs is 1757)
•	After installing setroubleshoot-server, you'll have the semanage command
•	SELinux port label commands are the following 
o	semanage port -l to list the ports
o	semanage port -l | grep http to narrow it down
o	semanage port --add -t http_port_t -p tcp 82 to add tcp 82 to the httpd_port_t label
o	/etc/selinux/targetd/contexts/files - This is where the semanage fcontext supposedly sends files
Time, timezone, NTP, hostname
•	man -k chron
•	timedatectl
•	/etc/chrony.conf
•	hostnamectl
•	nmcli general hostname <hostname>
Containers
•	podman and skopeo
•	podman search redis --filter=is-official
o	Find that in man podman-search
•	Rootless config https://www.redhat.com/sysadmin/rootless-podman-user-namespace-modes
•	dnf install -y container-tools
•	less /etc/containers/registries.conf
•	podman --help | less
•	podman info | less
•	podman login registry.access.redhat.com
•	podman search registry.access.redhat.com/ubi9 | less
•	podman pull registry.access.redhat.com/ubi9/ubi:latest
•	podman images - List images
•	skopeo inspect docker://registry.access.redhat.com/ubi9/ubi:latest | less
•	podman run -it registry.access.redhat.com/ubi9/ubi:latest
•	podman ps --all - List containers
•	podman rm -a - Remove all containers
•	podman images
•	podman rmi registry.access.redhat.com/ubi9/ubi:latest - Remove specific image
Recap
•	Basic commands are podman info, podman login, podman search, podman pull, podman images, skopeo inspect, podman inspect, podman run, podman ps, podman rm, podman rmi...
•	podman run creates and starts a container
•	podman stop stops a running container
•	podman start starts an existing container

