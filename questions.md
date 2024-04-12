# Questions

1   break into node02
    
    setup repo on node02
   
        http://server01/rhel9/AppStream/
        http://server01/rhel9/BaseOS/

solution 1
```
# setup repos
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

2      enable ssh, do not change port
    allow root to ssh

solution 2
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
```
3  http service on non-standard port 82. the system is not able to connect to httpd service at port 82, fix the issue.
	don't change files under /var/www/html directory, should be accessible on port 82 and should start at boot time

 solution 3
 ```
dnf install -y setroubleshoot*
systemctl restart httpd

journalctl
man semanage-port

semanage port -a -t http_port_t -p tcp 82
systemctl restart httpd
systemctl enable httpd

```

4	Create a user devuser1 and build image(Name pdfconvert) from url docker.io/openviewdev/pdfconverter
	run  a container named monitor using the newly created image

 solution 4
```
dnf install -y wget
dnf install -y podman
useradd devuser1
passwd devuser1

# appserver3 is prepped with above

ssh devuser1@appserver3
podman pull docker.io/openviewdev/pdfconverter
podman images # see image
podman run -dit --name pdfconvert [image ID]
podman ps
podman exec -it pdfconvert /bin/bash # confirm you create container
exit
```
5	create a container using the image monitor which has been created above

	* run container named monitor
 
 	* attach the volume /opt/input and /opt/processed with container /action/incoming/ and /action/outgoing/ respectively

solution 5
```
# as root
mkdir -p /opt/input /opt/processed
setfacl -m u:devuser1:rwx /opt/input
setfacl -m u:devuser1:rwx /opt/processed

man semanage fcontext | grep web

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
6	create a service container-monitor.service
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


