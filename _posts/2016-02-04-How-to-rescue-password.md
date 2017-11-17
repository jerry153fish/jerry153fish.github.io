---
layout: post
title: How to rescue password 
key: 20160204
tags: linux password rhcsa
comment: true
---

#### 1. press 3 before grub

#### 2. Then, go to the kernel line (the line starting with linux16) and add the following statements at the end:

```
rd.break enforcing=0
```

#### 3. remount the sysroot as writable

```
mount -o remount,rw /sysroot
```

#### 4. change root directory

```
chroot /sysroot
```

#### 5. reset password of certain user

```
passwd some-user
```

#### 6. exit

#### 7. login with new password and save all changes

```
restorecon /etc/shadow
reboot
```

Don’t need to force a SELinux relabel (# touch /.autorelabel) or load the SELinux policy (# /usr/sbin/load_policy -i).
