`dnf install -y setroubles* policycore*`

# reset root
```
mount -o remount,rw /
passwd
touch /.autorelabel
exec /usr/lib/systemd/systemd
```

# Documentation
```
Mandb
Man -k
Info
/usr/share/doc
```
# Logging
```
/var/log/messages
journalctl
```
set up persistent logging

`mkdir /var/log/journal`

`journalctl --flush` to flush the files from `/run/log/journal` to `/var/log/journal`

# SSH
```
ssh-keygen
ssh-copy-id user@server
ssh -p 22 user@server
```
SSH Security

•	Configure it in `/etc/ssh/sshd_conf`

•	Enable root login with the `PermitRootLogin yes` value

# Tuning Profiles
```
dnf install -y tuned
tuned-adm profile
/etc/tuned/tuned-main.conf
```
# Repos
`man dnf.conf`  find config values
```
dnf config-manager --addrepo=http://reposerver.example.com/BaseOS
dnf config-manager --add-repo=file:///repo/BaseOS
```
edit the repository file in /etc/yum.conf.d after adding it, so that it includes the line 
`gpgcheck=0`
verify:
```
dnf repolist all
dnf install telnet
```
# web / http
You will configure a web server running on your system serving content using a non-standard port (82)

- check if any service is running on that port 
```
curl http://localhost:82 
sudo ss -tlnp | grep ":82" 
sudo semanage port -l | grep http_port
journalctl will show semanage error and point you in right direction
```

# Grub
```
find / -type f -name grub
/etc/default/grub
```
Info grub2-mkconfig
`grub-mkconfig -o /boot/grub2/grub.cfg`

# Password Policy
•	Password age is configured in `/etc/login.defs`
•	Password quality is configured in `/etc/security/pwquality.conf`
•	Easy stuff is under `/etc/login.defs` 
```
PASS_MAX_DAYS 90
PASS_MIN_DAYS 15
PASS_WARN_AGE 7
```
# SELinux
•	Install everything under keywords policycoreutils and setroubleshoot 
`dnf install -y policycore* setrouble*`
•	Make enforcement settings persistent in `/etc/selinux/config`
•	If you're troubleshooting httpd, for instance, you can find info at the following: 
```
grep httpd /var/log/messages
grep httpd /var/log/audit/audit.log
journalctl -u httpd
ausearch -c 'httpd' --raw
ausearch -p 1757 (assuming the broken PID from the other logs is 1757)
```
•	After installing setroubleshoot-server, you'll have the semanage command
•	SELinux port label commands are the following 
```
semanage port -l to list the ports
semanage port -l | grep http to narrow it down
semanage port --add -t http_port_t -p tcp 82 to add tcp 82 to the httpd_port_t label
/etc/selinux/targetd/contexts/files - This is where the semanage fcontext supposedly sends files
```

# Time, timezone, NTP, hostname
```
man -k chron
timedatectl
/etc/chrony.conf
hostnamectl
nmcli general hostname <hostname>
```
# Storage Management
```
fdisk
man mkfs
mkswap
swapon
swapoff
```
Persistent Mounts
```
/etc/fstab
lsblk
blkid
swapon -a
wipefs -a  # equivalent to diskpart clean
```
Logical Volume Mangement
```
pvs
pvcreate
vgs
vgcreate
lvs
lvcreate
```
# File Systems
NFS
```
/etc/exports
/etc/fstab
firewall-cmd --add-service {nfs,rpc-bind,mountd} --permanent
firewall-cmd --reload
firewall-cmd --list-all
```
autofs
```
/etc/auto.master
  /mnt/nfs_home  /etc/auto.home
/etc/auto.home
  *  192.168.55.71:/export/home/&
```
# File Permissions
`umask`  sets default permission. 0002 says to set 755 on directories and 664 on files. 0022 says to set 755 on directories and 644 on files
`/etc/profile.d/`  change globally, set it on per-user basis by adding it to `~/.bashrc`. Set it for newly created users only under `/etc/skel/.bashrc`. you can also set it under `/etc/login.defs`
`chmod g+s`  set-GID bit
`chmod u+s`  set-UID bit
`chmod +t`   sticky bit

# Stratis
filesystem is always `xfs`

`defaults` is appended with the startis value `defaults,x-systemd.requires=stratisd.service`

How to find the mount options, since it's not in man stratis
`man -k mount`
`systemd.mount (5)`
`man 5 systemd.mount`  search by /fstab
`x-systemd.requires=`  first part
`systemctl status stratisd`  will give you second part: `stratisd.service`
```
dnf install -y stratis*
systemctl enable --now stratisd
stratis pool create mypool /dev/sdc
stratis pool list
stratis fs create mypool stratis1
stratis fs
vim /etc/fstab
UUID=<UUID>  /stratis1  xfs  defaults,x-systemd.requires=stratisd.service  0 0
```

# VDO
```
dnf install -y vdo
man lvmvdo
```
file system can be `xfs` or `ext4`
```
vgcreate vdo-vg /dev/sdb
lvcreate --type vdo -n web_storage -l 100%FREE -V 30G vdo-vg
```
* Heirarchy: Physical Volume > Volume Group > Pool > VDO Logical Volume
* `lvcreate` will also create the pool if you don't specify it, much like `vgcreate`  will create the physical volume
* when creating the filesystem on a VDO volume, add the `-K` option
  * `mkfs.ext4 -K /dev/vdo-vg/web_storage`
* `vdostats` will show you what's going on


# Containers
•	podman and skopeo
```
podman search redis --filter=is-official
dnf install -y container-tools
podman images # List images
podman run -it registry.access.redhat.com/ubi9/ubi:latest
podman ps --all # List containers
podman rm -a # Remove all containers
podman images
podman rmi registry.access.redhat.com/ubi9/ubi:latest # Remove specific image
```
Recap

•	Basic commands are podman info, podman login, podman search, podman pull, podman images, skopeo inspect, podman inspect, podman run, podman ps, podman rm, podman rmi...

•	podman run creates and starts a container

•	podman stop stops a running container

•	podman start starts an existing container
# Containers as Services
```
loginctl enable-linger <username>
loginctl show-user <username>
systemctl --user daemon-reload
systemctl --user start|stop|enable UNIT
mkdir ~/.config/systemd/user/
cd mkdir ~/.config/systemd/user/
podman generate systemd
```
# NFS
## Configure NFS server and share directory
```
dnf install -y nfs* rpc*
systemctl enable --now nfs-server
systemctl status nfs-server
firwall-cmd --add-service={nfs,mountd,rpc-bind} --permanent
firewall-cmd --reload
firewall-cmd --list-all
```
## Configure NFS share
```
mkdir /shared
chmod 777 /shared
vi /etc/exports
  /shared  *(fw,no_root_squash)
```
# autofs
user must be created on server with same UID on NFS server
```
dnf install -y autofs nfs-utils
showmount -e server01
vim /etc/auto.master
  /shared  /etc/auto.shared
vim /etc/auto.shared
  * -rw server01:/shared/&
systemctl restart autofs
```



