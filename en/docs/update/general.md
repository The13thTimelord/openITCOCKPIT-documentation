# General update of openITCOCKPIT

Updates of openITCOCKPIT are done via the package manager of the used linux distribution.

!!! danger "Backup"
    Before you start, make sure you have a **working backup** of your system!

!!! warning "E-Mail Configuration"
    If you made changes in e-mail configuration make sure to backup **/opt/openitc/frontend/config/email.php** before you update and restore it afterwards.

Please make sure to run all commands as `root` user.

## Ubuntu / Debian

Update package sources
```
apt-get update
```

Update
```
apt-get dist-upgrade
```


## Red Hat Enterprise Linux

Update package sources
```
dnf --refresh check-update
```

Update
```
dnf update
```
