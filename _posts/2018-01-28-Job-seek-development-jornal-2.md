---
layout: post
title: Job Seek Development Journal - 2
key: 20180128
tags: c# enity rabbitmq redis sharpnltk agile DDD
---

Last week, I tried to fit job seek in ddd concet


### Preparing

Docker is essential for development in job seek development for three purpose

* rabbitmq

```sh
 docker run -d --hostname my-rabbit -p 5672:5672 --name first-rabbit rabbitmq:3
 ```

* redis / postgres

```sh
docker run -p 6379:6379 --name first-redis -d redis # redis
docker run -p 5432:5432 --name first-postgres -e POSTGRES_PASSWORD=hellopassword -d postgres # posgresql

```
* selenium with chrome

```sh
docker run -p 4444:4444 --name first-selenium -d selenium/standalone-chrome
```


