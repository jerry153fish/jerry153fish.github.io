---
layout: post
title: How to build a church website 
key: 20171111
tags: ffmpeg sh youtube church website
---

Recently, the elders in the church ask me to design a church website which need to fulfill requirements below:

* Brief introduction
* Location and navigation
* Sermons
* Calender
* Groups
* Easy-to-use admin board

### Analysis

This is a simple website, there are two main technique stack to select here:

* Wordpress
* static html/css + google sheet

Wordpress could be the choice as it is very simple with various free templates and plugins. However, the church website does not have many functions and data. Most importantly, it could be too hard for the elders to login and manage all resources despite of its simple dashboard.

Thus, using google sheet as back-end become my the silver bullet for the management issue. Moreover, it has collaboration function which could be extremely useful for the church management which has different groups with different activities. This is because, anyone who are shared with 'dashboard google sheet' can edit the content.

### Implementation

The technique stack includes

* [firebase](https://firebase.google.com/) for hosting and free ssl certificates
* [google sheet](https://developers.google.com/sheets/api/) for resource managements
* [google calendar](https://developers.google.com/google-apps/calendar/) for calender 
* [google map](https://developers.google.com/maps/) for location and navigation
* [youtube](https://www.youtube.com/channel/UCLfms9DVVhlOx9L2s5EaHhw) for sermon records hosting and [timeline](http://timeline.knightlab.com/) for displaying 
* [cordova](https://cordova.apache.org/) for mobile app building
* [materilaize](http://materializecss.com/) for main css framework and [vuejs](vuejs.org) as simple js framework


#### Home page

There are two basic function in here:

* main activities for sunday service which are managed by google sheet and demonstrated by vue
* calendar - a public google calendar which could be collaborated by authorized church members. 


#### About us

This is a simple html file which includes our states of faith and simple profiles of our elders

#### Messages

This page shows a time line of all sermons video of our church. Setting up timelinejs in this page is not hard. The trick part is to turn the audio file into youtube youtube compatible format video file then upload to youtube. 

The code below need to run a *nix system and with ffmpeg installed,


```sh
#!/bin/bash

# convert audio and image to video with youtube format
function youtube {
	ffmpeg -loop 1 -framerate 1 -i logo.png -i $1 -c:v libx264 -preset medium -tune stillimage -crf 18 -c:a copy -shortest -pix_fmt yuv420p $(dirname $file)/$(basename $file .wma).mkv
}

# convert all .wma to youtube video
function convertDir {
	for file in $1/*;do
		youtube $file
	done
}
# 
for dir in Teaching_recorders/*;do
	convertDir $dir
done
```

The code above cover all wma files inside Teaching_recorders/any-sub-directory/*.wma into video file. The ratio of logo.png **must be 16:9**.

#### Notices

This pages contains Roster and and player list which are both built with google sheet and vue

#### Contacts

This pages contacts a route map 

* google embed map for showing location of our church - destination
* google autocomplete api for getting the start location

and google form for submit form

###  Conclusion

The website is hosted in firebase which is free, the loading time of website is ok, just check mpcf.com.au. As the website use ajax to fetch data from google sheet, it is not SEO optimized. However, google is promoting google place. Thus, we put our church on google place which is fine for us in google search.

