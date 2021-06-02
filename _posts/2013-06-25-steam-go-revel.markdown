---
layout: post                                                                                                                  
title:  "Go + Revel Framework + Steam"
date:   2013-06-25 18:41:55
categories: go api revel
---

# Experience with Revel

This post gives a high level view of my experience writing an API using Go and the Revel Framework. The API was used to interact with the Marvel Heroes MMO game and Steam, the game portal. A more detailed explanation of how the application was built is yet to come.

The goal was to build an API which third-party applications can call to integrate with our system. A relatively high throughput was needed since we planned for multiple applications to use this API, and we are not a company that has a lot of hardware to throw around. It needed to efficient since it would be sharing the same servers with other applications. Steam was the first application, and we had four weeks to build and deploy it to production.

We decided to use [Go](http://golang.org/) for the following reasons:

1) It came with a queue out-of-the box (channels). We initially thought of using PHP + Gearman, but that would mean convincing operations to deploy yet another piece of software.

2) The combination of Go routines and channels made for a very high throughput API.

3) It was new and cool.

Never underestimate that third reason. I had a lot of fun building this application with another developer. I’ve learned that if you do not inject fun into your projects at work, you’ll end up hating work. It’s probably true for a lot of things in life … but I digress.

We _could_ have built the application from scratch, but found the Revel Framework. We wanted something that was small and fast, but still did everything we needed. Revel gave us routing and a test module out of the box and we were off creating an API. 

The test module was heaven sent. Writing high quality code just got easier. With Revel, we wrote a test for every use case, every function, and every model in our application. Our build process was tied to the test suite. The build failed if any test failed. 

By the time we deployed the application it simply worked … in production. That’s right, in four  weeks it was production ready. Big thanks to Rob Figueiredo and company for steadily improving that framework.


[MongoDB](http://www.mongodb.org/) was the database of choice. We did not know what the data were going to look like and we had fours weeks to deliver. NoSql provided us a way to handle that situation and the fact that some of our core systems already used MongoDB made it an easy choice.

Looking back, MongoDB was a very good choice. I had to change the “schema” of the DB multiple times. Steam’s unreliable technical support and sandbox were two big reasons for that. I'll explain that later, but if you’ve ever worked with Steam, you probably know what I’m talking about.

Running on a VM with Kubuntu, 512MB memory on a Windows 7 machine, the application was able to service 600+ requests per second comfortably. The queue began filling up at 800+ requests per second. I used JMeter for load testing.

Yes, I know that’s shabby performance reporting on my part, but the last time I load tested our staging environment I brought the servers down and was asked not to load test again. At least not in staging hardware. So, simply take into account that production consisted of two servers each having 16 GB of RAM, and what you get is production grade throughput. To give you an idea of how much of an overkill this was, at peak times of the game, our application uses 1% of the server’s CPU.

For a glimpse of some key features of the application, here is some sample code. Like I mentioned earlier, I will give implementation details soon. I simply wanted to give an overview of using Go and Revel to build an API..
