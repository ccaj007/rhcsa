```
mandb
man -k dnf-config
dnf config-manager --add-repo http://server/repo
/etc/yum.repos.d/*.repo
vim *.repo - add gpgcheck=0
dnf clean dbcache
dnf install -y policycore*
dnf install -y setroubleshoot*
dnf whatprovides *semanage
```
