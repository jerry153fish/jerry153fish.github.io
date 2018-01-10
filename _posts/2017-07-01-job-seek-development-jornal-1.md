---
layout: post
title: Job Seek Development Journal - 1
key: 20170701
tags: python scrapy sqlalchemy agile sqlite oop nltk
---

This development journal collects my development traces of job seek. Job seeker started when the project I was working was about to finish. I wanted to keep an eye on job market and the trend of techniques.

### Requirement development 

The purposes of this toy project is very simple, 

1. Find jobs in employment websites based on keywords
2. Automatically generate Cover letters and CVs in terms of the profile and job details

### Requirement Analysis 

According to the requirements,  three are four sub-system is essential here.

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
