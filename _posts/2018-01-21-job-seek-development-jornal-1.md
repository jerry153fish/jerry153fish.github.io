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

The figure below demonstrate the subsystems needed in job seek and the main technique stacks required,

![job seek main](/assets/img/jobseek/job-seek.png) 

* User interface
    * command line
    * web
* Model
    * job related
    * user related
* Controllers  
    * Scrapy 
    * classifier for job and profile
    * generator for resume 
    * utility

### Major class

* Scrapy Classes
* Classifier Classes
* Job Class
* Profile Class
* GUI Class
* Model Classes

### Data design

### Job
1. name: string
2. category: string
3. description: text
4. requirements: [{
    name: string,
    keywords: [strings],
    priority: number
   }]
5. email: string
6. contact name: string
7. phone: string
8. notes: text

### profile

1. skills: [{
    name: string,
    keywords: [strings]
   }]

2. experiences: [{
    description: text,
    start: datetime,
    end: datatime,
    position: String,
    keywords: [String] # for search,
    achievement: text,
  }]
3. education: [{
    description: text,
    start: datetime,
    end: datetime,
    major: string,
    score: string
  }]
4. project: [{
    description: text,
    start: datetime,
    end: datatime,
    position: String,
    keywords: [String] # for search,
    achievement: text
  }]
