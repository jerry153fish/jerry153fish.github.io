---
layout: post
title: Job Seek Development Journal - 1
key: 20180121
tags: c# enity rabbitmq redis sharpnltk agile DDD
---

Job seeker started when wherewot projects were about to finish. The first version of it was wrote in python. Now I want to refactor it with .net core stack due to ~~the job market trends in hobart~~ its simplicity and powerful features.

The purposes of this toy project is very simple,

* Find jobs in employment websites based on search query
* Automatically generate Cover letters and CVs in terms of the profile and job details
* Analyse the user inputs and job market and try to give useful statistics


The python version I wrote before is only for personal usage, I was always told 'dream bigger'. So this version, I am going to extends its scalability by turning into a C / S cloud application.

### Feasibility and possible technique sets 

The figure below displays the simple DDD four layer structure and the main technique stacks required within them. Despite many need-to-learn knowledge especially the .net core stack and rabbitmq, most of concepts and design pattern in the project are not new for me.

![job seek main](/assets/img/jobseek/job-seek.png) 


### Main architecture

I will use CQRS for the main pattern[^1], because:

* To send job one by one back to user in websocket if use start to search job by keywords
* There are many time consuming tasks in the project: web crawling / email / pdf and docs generating
* The analyse tasks will use lots of write IO
* Only need eventually consistency


![CQRS](/assets/img/jobseek/CQRS.jpg)


The main process of this job seek would be:

1. When user open any ends, a websocket connection will be established.
2. The server will initialize jwt, message broker and other server resource
3. Then depends on the actions user started, the controller will decide to go through command bus or directly query database.

### Domain define

Despite not been an expert in job seeking area, after finishing this project, I hope I will be. 

#### UL 

* job: which refers to the posted job in the job market website -- taggable
* requirements: the requirement in a job -- taggable
* employer: just employer -- taggable
* category: job category


* location: location contains address and latitude / longitude
* locality: locality with postcode
* state: state
* country: country


* profile: a collection of user information, skills, education, experience, projects and so on
* skills: skills -- taggable
* experience: working experience -- taggable
* projects: user projects -- taggable
* activities: user activities -- taggable


* tags: identifier and corner store in the project
* keywords: just keywords

* user: login user and non login user
* role: role
* assets: cvs, cover letters, applied jobs
* token: json web token for user

#### BC

* user search
* user query
* user register / login
* user manage
* tag system


#### Entity

* job
* user
* profile
* experience
* projects
* educations
* activities
* otherItems

#### Value Object

* tag
* keyword
* category
* skills
* requirements
* location
* locality
* state
* country
* employer

#### db design

As can been seen from the db table below, job seek is highly replied on tag system. All every value objects and some profile related entities are tagged with a weight.

![db design](/assets/img/jobseek/db-design.jpg)

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





### Reference

[^1]: Microsoft 2018, **CQRS Journey and guide**, https://msdn.microsoft.com/en-us/library/jj554200.aspx

