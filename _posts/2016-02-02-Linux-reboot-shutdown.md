---
layout: post
title: Linux reboot and shutdown
key: 20160202
tags: linux rhcsa reboot shutdown
comment: true
---
####  reboot the system
```
reboot
systemctl reboot
shutdown -r now
init 6
telinit 6
```

#### shutdown
```
halt
sytemctl halt
shudown -h now
init 0
telinit 0
```

#### poweroff
```
poweroff
systemctl poweroff
```

#### suspend

```
systemctl suspend
```

#### hibernation and suspend

```
systemctl supend
```

#### boot system manually
The default run level was set in the /etc/inittab file.

1. single: maintenance level,
2. level without network resources (NFS, etc),
3. multi-user level without graphical interface,
5. multi-user level with graphical interface.


To get the current run level with the old way, type:
```
runlevel
```

new *Systemd*, new commands are available:

1. systemctl rescue: to move to single user  mode/maintenance level with mounted local file systems,
2. systemctl emergency: to move to single user mode/maintenance with only /root mounted file system,
3. systemctl isolate multi-user.target: to move to multi-user level without graphical interface (equivalent to previous run level 3),
4. systemctl isolate graphical.target: to move to multi-user level with graphical interface (equivalent to previous run level 5),
5. systemctl set-default graphical.target: to set the default run level to multi-user graphical mode,
6. systemctl get-default: to get the default run level.
