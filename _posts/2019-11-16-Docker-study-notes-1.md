---
layout: post
title: Docker study notes
key: 20191116
tags: Docker Linux kubernate
---

### Introduction

This is the first part of kubernetes study note. The part include the basic concepts of docker. How some docker command works 


1. Linux namespace
2. Linux CGroup
3. Union file group



### Namespace

#### Process namespace


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















