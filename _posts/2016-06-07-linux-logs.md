---
layout: post
title: Linux Log Files 
key: 20160607
tags: Linux Logs Note
---

### logs in linux

- /var/log/boot.log：

The process of kernel detecting devices during booting and initing core functions will be record here. Only record for each booting.

- /var/log/cron：

cron jobs go here

- /var/log/lastlog：

last login

- /var/log/maillog 或 /var/log/mail/\*：

for postfix (SMTP) and dovecot (POP3)

- /var/log/messages：

*important* Every major infomation will go here

- /var/log/secure：

Anything related to password will go here, including login, gdm, su, sudo, ssh, telnet and so on

- /var/log/wtmp, /var/log/faillog：

wtmp and faillog