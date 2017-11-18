---
layout: post
title: How To Rescue Password 
key: 20160204
tags: linux password rhcsa
comment: true
---

### Step one

> press 3 before grub

### Step two

> Then, go to the kernel line (the line starting with linux16) and add the following statements at the end:

```
rd.break enforcing=0
```

### Step three

> remount the sysroot as writable

```
mount -o remount,rw /sysroot
```

### Step four

> change root directory

```
chroot /sysroot
```

### Step five

> reset password of certain user

```
passwd some-user
```

### Step six

>exit

### Step seven

> login with new password and save all changes

```
restorecon /etc/shadow
reboot
```

Don’t need to force a SELinux relabel (# touch /.autorelabel) or load the SELinux policy (# /usr/sbin/load_policy -i).
