---
layout: post
title:  "Building a system from scratch"
date:   2014-05-25 18:41:55
categories: tech
---

### Given: 

Small company. I am the only developer. There is a beta system that is already generating money.


### Goals:

1.	Build a system that makes a lot of money
2.	Build a system that is thoroughly tested ( I don’t want to get calls at 3 a.m. because something in production is broken)
3.	Build something I can be proud of.
4.	Build a system that can be managed easily by a very small team, i.e. me, myself, and I.

### Questions to ask when I get there
1.	What features in the current system makes us the most money? Harden that piece of code.
2.	What features would you like to add which will make us the most money? Make that my top priority.


### Miscellaneous Thoughts

1.  Since I don't have a QA team, I would need to harden my code with [unit testing](http://laravel.com/docs/testing#defining-and-running-tests) and integration tests. Laravel should help with testing. Unit tests will run during [continuous builds](http://en.wikipedia.org/wiki/Continuous_integration) to the dev environment. “Git push” will trigger the tests.
2. Integration tests will simulate user interaction with the API, load testing, and checks for data integrity. The idea is to identity problems in the system which appear only after long periods of up-time (think several days). This kind of testing complements unit tests very well. Unit tests run for short periods of time and integration testing runs against a staging environment for days.
3. Since I am big on testing, any existing system should be re-factored for easy testing. This often results in more readable code.
4. Even if it's a small team, have a well-defined deployment process. This will save my butt if something goes wrong in production.
5. Since I don't have a DevOps team, I would consider putting the app in the cloud [PAAS over IAAS[(
6. Make sure data are backed up. At least the important ones. Transactions always needs back-up.
7. Get metrics on everything. This is useful for billing, future sales, and generating revenue.

