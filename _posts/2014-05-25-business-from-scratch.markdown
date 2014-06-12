---
layout: post
title:  "Building a system from scratch"
date:   2014-05-25 18:41:55
categories: tech
---

# If I were to build a system from scratch (kinda). 

Given: Small company.  I am the only developer.  There is a beta system that is already generating money.


## Goals: 
1. Build a system that makes a lot of money
2. Build a system that is thoroughly tested ( I don't want to get calls at 3am because something in production is broker) 
3. Build something I can be proud of. 
4. Build a system can be managed easily by a very small team. i.e. me, myself and I. 

## Question to ask when I get there
1. What features in the current system makes us the most money? Harden that piece of code.
2. What features would you like to add which will make us the most money?  Make that my top priority.


## Stuff

* Since I don\'t have a QA team, I would need to harden my code with unit testing and integration tests.  
Laravel should help with [testing]( http://laravel.com/docs/testing#defining-and-running-tests).
Unit tests will run during [continuous](http://en.wikipedia.org/wiki/Continuous_integration) builds to the dev environment.  "git push" will trigger the tests. 


* Integration tests will simulate user interaction with the API, load testing and checks for data integrity.  The idea is to identity problems in the 
system which appears only after long periods of up-time (think several days).  This kind of testing complements unit tests very well.  Unit tests runs
for short periods of time and Integration testing runs against a staging environment for days. 

* Since I am big on testing any existing system should be refactored for easy testing.  This often results in more readable code.    

* Even if it\'s a small team, have a well defined deployment process.  This will save my butt if something goes wrong in production. 

* Since I don\'t have a devops team, I would consider putting the app in the cloud 
[PAAS over IAAS[(http://apprenda.com/library/paas/iaas-paas-saas-explained-compared/). 

* Make sure data is backedup. At least the important ones.  Transactions always needs backup. 

* Get metrics on *everything*. This is useful for billing, future sales and generating revenue.  
 
