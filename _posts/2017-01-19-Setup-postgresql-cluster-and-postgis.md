--- 
layout: post
title: Setup postgesql cluster and install Postgis 
key: 20170118
tags: Postgresql Database Postgis 
---

Recently, I had upgrade ubuntu from 14.04 to 16.04. When I upgrade postgresql and try to set up postgis, I encountered a bug. This article record the process of solving it.

### Background

#### Upgrading ubuntu from 14.04 to 16.04

```sh
sudo do-release-upgrade # will take hours depends
```

#### Then change data directory for postgresql 

```sh
# make new directory and set ownership
mkdir /data/pg
sudo chown postgres /data/pg

# copy data
sudo systemctl stop postgresql

sudo rsync -av /var/lib/postgresql /data/pg
sudo mv /var/lib/postgresql/9.5/main /var/lib/postgresql/9.5/main.bak ## backup in case failed

# change data directory
sudo vim /etc/postgresql/9.5/main/postgresql.conf

    data_directory = '/data/pg/postgresql/9.5/main' 

sudo systemctl start postgresql
```

#### set up postgis

```
sudo apt install postgis postgresql-9.5-postgis-2.2
```

### Bug appears

> postgis faild to create postgis extension ppa

```
# CREATE EXTENSION postgis; ERROR: could not load library "/usr/lib/postgresql/9.5/lib/postgis-2.2.so":  /usr/lib/x86_64-linux-gnu/libSFCGAL.so.1: undefined symbol: _ZN5osgDB13writeNodeFileERKN3osg4NodeERKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEEPKNS_7OptionsE
```

#### google  and find [the reason]( https://gis.stackexchange.com/questions/204575/libsfcgal-so-1-undefined-symbol-upgrading-to-postgis-2-2-2-and-2-2-1/206155#206155)

*miss libopenscenegraph-dev packages*

try to install libopenscenegraph-dev

```sh 
 sudo apt install libopenscenegraph-dev find umet packages
 ```

#### another bug appears

```sh
libopenscenegraph-dev : Depends: libopenthreads-dev but it is not going to be installed
                        Depends: libopenscenegraph100v5 (= 3.2.1-7ubuntu4) but 3.2.3+dfsg1-1~trusty3 is to be installed
```

Then check what has been installed

```sh
# list what installed
dpkg -l | egrep "open(scenegraph|threads)"
# results
ii  libopenscenegraph100v5:amd64               3.2.3+dfsg1-1~trusty3                         amd64        3D scene graph, shared libs
ii  libopenthreads20:amd64                     3.2.3+dfsg1-1~trusty3                         amd64        Object-Oriented (OO) thread interface for C++, shared libs
```

google again find:

*cause: use unstable ubuntugis ppa*

### Solution

Install newest ubuntugis

```sh
sudo add-apt-repository --remove ppa:ubuntugis/ubuntugis-unstable
sudo add-apt-repository ppa:ubuntugis/ppa

sudo apt update && sudo apt upgrade

sudo apt autoremove

sudo install libopenscenegraph-dev
```
