---
layout: post
title: Docker study notes
key: 20191116
tags: Docker Linux kubernate
---

### Introduction

This is the first part of kubernetes study note. The part include the basic concepts of docker. How some docker command works 


It is all about how to the **border** of the executable image

1. Linux namespace
2. Linux CGroup
3. Union file group



### Namespace

```
docker run -it busybox ps -a

```

这种技术，就是 Linux 里面的 Namespace 机制。而 Namespace 的使用方式也非常有意思：它其实只是 Linux 创建新进程的一个可选参数。我们知道，在 Linux 系统中创建线程的系统调用是 clone()，比如：

int pid = clone(main_function, stack_size, SIGCHLD, NULL);
这个系统调用就会为我们创建一个新的进程，并且返回它的进程号 pid。

而当我们用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数，比如：

int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL);
这时，新创建的这个进程将会“看到”一个全新的进程空间，在这个进程空间里，它的 PID 是 1。之所以说“看到”，是因为这只是一个“障眼法”，在宿主机真实的进程空间里，这个进程的 PID 还是真实的数值，比如 100。

```

#### Process namespace

分类	系统调用参数	相关内核版本
Mount namespaces	CLONE_NEWNS	Linux 2.4.19
UTS namespaces	CLONE_NEWUTS	Linux 2.6.19
IPC namespaces	CLONE_NEWIPC	Linux 2.6.19
PID namespaces	CLONE_NEWPID	Linux 2.6.24
Network namespaces	CLONE_NEWNET	始于Linux 2.6.24 完成于 Linux 2.6.29
User namespaces	CLONE_NEWUSER	始于 Linux 2.6.23 完成于 Linux 3.8)

#### Network namespace


```
sudo ip netns add test1
sudo ip netns add test2
sudo ip link add veth-test1 type veth peer name veth-test2
sudo ip link set veth-test1 netns test1
sudo ip link set veth-test2 netns test2
sudo ip netns exec test1 ip addr add 192.168.77.1/24 dev veth-test1
sudo ip netns exec test2 ip addr add 192.168.77.2/24 dev veth-test2

sudo ip netns exec test1 ip link set dev veth-test1 up 
sudo ip netns exec test2 ip link set dev veth-test2 up 
sudo ip netns exec test1 ping 192.168.77.2
sudo ip netns exec test2 ping 192.168.77.1
```















