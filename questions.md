# Questions


1	fix http on server01; do not change port

```
dnf install -y setroubleshoot*
systemctl start httpd
journalctl
emanage port -a -t http_port_t -p tcp 82
firewall-cmd --add-port=82/tcp --permanent
firewall-cmd --reload
curl http://localhost:82
```

2	change http port to 80 on server01

```
vim /etc/httpd/conf/httpd.conf
	Listen 80
systemctl restart httpd
systemctl status httpd
firewall-cmd --add-service=http --permanent
firewall-cmd --reload
```

3   break into node02 and reset root password

solution 3
```
e at grub
init=/bin/bash
mount -o remount,rw /
passwd root
touch /.autorelabel
exec /usr/lib/systemd/systemd
```

4. 	setup repo on node02
  
        http://server01/rhel9/AppStream/
        http://server01/rhel9/BaseOS/

solution 4
```
# setup repos
mandb
man -k dnf
man dnf-config-manager
# G and scroll up to EXAMPLES
dnf config-manager --add-repo http://server01/rhel9/AppStream/
dnf config-manager --add-repo http://server01/rhel9/BaseOS/
vim /etc/yum.repos.d/server01_rhel9_AppStream.repo
vim /etc/yum.repos.d/server01_rhel9_BaseOS.repo
  gpgcheck=0

dnf repolist  # should show you local path now
dnf install telnet # configm
```

5      enable ssh on node02, do not change port
    allow root to ssh

solution 5
```
# install troubleshooting
dnf install -y coreutils setroubleshoot*

systemctl start sshd
journalctl
# will show you semanage command
[admin@node02 ~]$ sudo semanage port -l | grep ssh
ssh_port_t                     tcp      2202, 22

semanage port -a -t ssh_port_t -p tcp 2202
systemctl enable --now sshd

# add firewall
firewall-cmd --add-port 2202/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all

sudo vim /etc/ssh/sshd_config
  PermitRootLogin yes

systemctl enable --now sshd
ssh root@node02 -p 2202
```

6	setup ssh as root to node02 without password

```
workstation as root
ssh-keygen
ssh-copy-id -p 2202 root@node02
```

7  http service on non-standard port 82 on appserver3 (10.0.0.13). the system is not able to connect to httpd service at port 82, fix the issue.
	don't change files under /var/www/html directory, should be accessible on port 82 and should start at boot time

 solution 7
 ```
dnf install -y setroubleshoot*
systemctl restart httpd

journalctl
less /var/log/messages
man semanage-port

semanage port -a -t http_port_t -p tcp 82
systemctl restart httpd
systemctl enable httpd

fireall-cmd --add-port=82/tcp --permanent
firewall-cmd --reload

```

8	Create a user devuser1 and build image(Name pdfconvert) from url docker.io/openviewdev/pdfconverter
	run  a container named monitor using the newly created image
 	run on appserver3

solution 8
```
dnf install -y wget
dnf install -y podman
useradd devuser1
passwd devuser1

# appserver3 is prepped with above

# add devuser1 with sudo permission
ssh devuser1@appserver3
sudo podman pull docker.io/openviewdev/pdfconverter
sudo podman images # see image
sudo podman run -dit --name pdfconvert [image ID]
sudo podman ps
sudo podman exec -it pdfconvert /bin/bash # confirm you create container
exit
```
9	As devuser1 create a container using the image monitor which has been created above
	* run container named monitor
  	* attach the volume /opt/input and /opt/processed with container /action/incoming/ and /action/outgoing/ respectively

solution 9
```
# as root
mkdir -p /opt/input /opt/processed
setfacl -m u:devuser1:rwx /opt/input
setfacl -m u:devuser1:rwx /opt/processed

man semanage-fcontext # /EXAMPLE
semanage fcontext -l | grep containerd

semanage fcontext -a -t container_file_t "/opt/input(/.*)?"
semanage fcontext -a -t container_file_t "/opt/processed(/.*)?"
restorecon -R -v /opt/

loginctl enable-linger devuser1

ssh devuser1@ipaddress
podman run -dit --name monitor -v /opt/input/:/action/incoming/ -v /opt/processed/:/action/outgoing/ [image_id]

# login pdfconverter container to validate
pdoman exec -it monitor /bin/bash
cd /action/incomiing
```
10	create a service container-monitor.service
	ensure that container-monitor.service will run automatically at system boot
```
man -k podman-generate
man podman-generate-systemd

mkdir -p ~/.config/systemd/user
cd ~/.config/systemd/user
podman generate systemd --name monitor --files --new
systemctl --user daemon-reload
systemctl --user enable --now container-monitor.service
systemctl --user reload container-monitor.service
```

# NFS & autofs
11	Perform following task on "node02" as "rhcsa9" user 
	Configure autofs to automount the home directories of user "userautofs01". 

Note the following: 
server01.rhcsa9.momer.io(10.0.0.91), NFS-exports /shared to your system, 
userautofs01 home directory is server01.rhcsa9.momer.io:/shared/userautofs01 
userautofs01 home directory should be auto mounted locally at "/shared" as "/shared/userautofs01" 
Home directories must be writable by the users when you login as "userautofs01" on "node02"

solution
# configure NFS server
server01
probably set up already in exam
mkdir -p /shared
useradd -u 1234 -d /shared/userautofs01 userautofs01

dnf install -y nfs-utils
systemctl enable --now nfs-server
vim /etc/exports
	/shared *(rw,no_root_squash)
systemctl restart nfs-server
firewall-cmd --add-service={nfs,mountd,rpc-bind} --permanenet
firewall-cmd --reload
showmount -e localhost

ssh -p 2202 rhcsa9@node02
sudo mkdir -p /shared
sudo useradd -u 1234 -d /shared/userautofs01 userautofs01
sudo dnf install -y autofs
sudo dnf install -y nfs-utils
showmount -e 10.0.0.91
sudo vim /etc/auto.master
	/shared /etc/auto.shared
sudo vim /etc/auto.shared
	* -rw	10.0.0.91:/shared/&
 
	


	







