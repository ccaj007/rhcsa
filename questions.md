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

