---
layout: post                                                                                                                  
title:  "Go + Revel Framework + Steam"
date:   2013-06-25 18:41:55
categories: go api revel
---

This post gives a high level view of my experience writing an API using Go and the Revel Framework.
The API was used to interact with the Marvel Heroes MMO game and Steam, the game portal.
A more detailed explanation of how the application was built is yet to come.

The goal was to build an API which 3rd party applications can call to integrate with our system.
A relatively high throughput was needed since we planned for multiple applications to use this API and we
are not a company that has a lot of hardware to throw around.  It needed to efficient since it\’ll be
sharing the same servers with other applications.  With the first application being Steam we had 4 weeks to build and deploy to production.

We decided to use Go for the following reasons:

1. It came with a queue out-of-the box (channels).  We initially thought of using PHP + Gearman, but that would mean
 convincing operations to deploy yet another piece of software.  
2. The combination of Go routines and channels made for a very high throughput API.
3. It was new and cool.


Never underestimate reason #3.  I had a lot of fun building this application alongside another developer.
I’ve learned that if you do not inject “fun” into your projects at work, you’ll end up hating work.
It’s probably true for a lot of things in life… but I digress.

We could have built the application from scratch, but found the Revel Framework.  
We wanted something small and fast, but did everything we needed.  Revel gave us routing and a test
module out-of-the-box and we were off creating an API.  The test module was heaven sent.
Writing high quality code just got easier.  With Revel we wrote a test for every use case,
every function and every model in our application.  Our build process was tied to the test suite.
The build failed if any test failed.  By the time we deployed the application it simply worked…in production.  That’s right, in 4 weeks it was production ready.  Big thanks to Rob Figueiredo and company for steadily improving that framework.

[MongoDB](http://www.mongodb.org/) was the database of choice.  We did not know what the data was going to look like and we had 4 weeks to deliver.  NoSql provided us a way to handle that situation and the fact that some of our core systems already used MongoDB made it an easy choice. Looking back, MongoDB was a very good choice. I had to change the “schema” of the DB multiple times.  Steam’s unrealiable technical support and sandbox were two big reasons for that. I’ll get to that soon. But if you’ve ever worked with Steam you probably know what I’m talking about.

Running on a VM with Kubuntu, 512Mb memory on a Windows 7 machine the application was able to service 600+ requests/second very comfortably.  The queue began filling up at 800+ requests/second. I used JMeter for load testing.

Yes, I know that’s shabby performance reporting on my part, but the last time I load tested our staging environment I brought the servers down and was asked not to load test again. At least not in staging hardware.  So, simply take into account that production consisted of 2 servers each having 16 Gigs of ram and what you get is production grade throughput.   To give you an idea of how much an overkill this was, at peak times of the game, our application uses 1% of the server’s CPU.

For a glimpse of some key features of the application here is some sample code. Like I mentioned earlier, I will give implementation details soon. I simply wanted to give an overview of using Go and Revel to build an API.
