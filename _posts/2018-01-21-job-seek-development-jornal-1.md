---
layout: post
title: Job Seek Development Journal - 1
key: 20180121
tags: c# enity rabbitmq redis sharpnltk agile
---

Job seeker started when wherewot projects were about to finish. The first version of it was wrote in python. Now I want to refactor it with .net core stack due to the job market needs in hobart.

The purposes of this toy project is very simple,

* Find jobs in employment websites based on search query
* Automatically generate Cover letters and CVs in terms of the profile and job details
* Analyse the user inputs and job market and try to give useful statistics

### Requirement Analysis 

The figure below demonstrate the subsystems needed in job seek and the main technique stacks required. This is just a mind map, which could be updated during development. However, the main stack will be .net core based and we will adopt C / S model to develop this projetc.

![job seek main](/assets/img/jobseek/job-seek.png) 


* Server layer
    * .net core API + Websocket
    * will separate the model layer
    * a cache layer before it

* Ends
    * Simple reactjs / redux / websocket
    * Use indexdb as cache -- hash query key based
    * Might demonstrate statistics 

* Analysis
    * bag of keys + ntlk for text analysis and tagging
    * might use AI to for tagging
    * for both job and profile 

* Scrapy
    * performance bottlenecks must work with message broker in RPC model ? 
    * simple multiple threading

* Message broker
    * rabbitmq + RPC model ?

* Utility
    * PDF generator server / end ?
    * docs generator 
    * email sender

* Models
    * Entity 
    * Job groups
    * Profile group
    * Tag based


### Simple Sequence diagram

The main sequence of this job seek would be:

1. When user open end, a websocket connection will be established.
2. The server will initialize jwt, message queue and other server resource
3. The user start to search
4. check cache in both client side and server side see if there was a hit
5. If new search, server start to scarpy | tag the result | save to database | generate cv or letters
    *   scrapy will use 


