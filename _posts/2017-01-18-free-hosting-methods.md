---
layout: post
title: Free Hosting Methods
key: 20170118
tags: hosting website free
---
---
layout: post
title: Free Hosting Methods
key: 20170118
tags: hosting website free
---

In this article, I am going to introduce three way to free hosting your simple website.

### Google Firebase

> Firebase Hosting is a static and dynamic web hosting service that launched on May 13, 2014. It supports hosting static files such as CSS, HTML, JavaScript and other files, as well as dynamic Node.js support through Cloud Functions. The service delivers files over a content delivery network (CDN) through HTTP Secure (HTTPS) and Secure Sockets Layer encryption (SSL). Firebase partners with Fastly, a CDN, to provide the CDN backing Firebase Hosting. The company states that Firebase Hosting grew out of customer requests; developers were using Firebase for its real-time database but needed a place to host their content.[^1]

Firebase provide 1G space and 10G per month transfer for free, which is pretty enough for normal website.

#### Setup

* register a google account
* create a firebase project in firebase console
* install nodejs
* setup

```js

# install firebase
npm install -g firebase-tools

# login in google
firebase login

# go to website root
cd path-to-your-website-root

# init firebase
firebase init

# before deploy remember to configure firebase.json
firebase deploy

```

Firebase supports connecting to a custom domain after verifying ownership of domain. eg [www.mpcf.com.au](https://mpcf.com.au)

### Github pages

Github pages is another static hosting service provided by github. Moreover, it integrated with Jekell for static and blog generation. 

Github pages provide 1G space and 100G transfer per month for free.

#### set up

* create a repository in github 
    
    * your-github-name.github.io
        * a 
    * other name 

### Your house or office

You can host any website at home or in your office. 

### Reference

[^1]: Wiki 2017, *Fire base*, https://en.wikipedia.org/wiki/Firebase