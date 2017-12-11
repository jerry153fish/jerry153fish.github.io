---
layout: post
title: Ubuntu Development Set Up
key: 20171120
tags: ubuntu i3 zsh vim bash linux
---

This article show how to setup a ubuntu development with i3, oh-my-zsh and spaceVim.

### Preparation

- Download [virtualbox](https://www.virtualbox.org/) if windows or mac
- Download [Ubuntu](https://www.ubuntu.com/) and install it

### Set up oh my zsh

- install zsh and set as default sh

```bash
sudo apt update && sudo apt install zsh # install zsh
sudo chsh $USER -s `which zsh` # also works on aws ubuntu ec2 instance
```
- install oh-my-zsh and say goodbye to bash configuration

```bash
sudo apt update && sudo apt install curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

- install autojump
```bash
sudo apt update && sudo apt install autojump
```
> add to autojump to .zshrc

```
plugins=(git autujump)
```

### Set up SpaceVim

- install vim and SpaceVim say good bye to vim configuration

```bash
sudo apt update && sudo apt install vim
curl -sLf https://spacevim.org/install.sh | bash
```


### Set up i3 as window manager

- install

```bash
sudo apt update && sudo apt install i3
```
- configuratio 

> do not show desktop-icons for nautilus

```
exec_always --no-startup-id gsettings set org.gnome.desktop.background show-desktop-icons false 
```
for me that is enough, but you can configure it as the way you want.

### To be continued..

![ubuntu](/assets/img/ubuntu.png)
