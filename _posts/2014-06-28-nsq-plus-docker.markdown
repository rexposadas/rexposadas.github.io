---
layout: post                                                                                                                  
title:  "Connecting dockerized nsqd and nsqlookupd"
date:   2014-06-28
categories: docker nsq
---

*Update:* NSQ has official [`images`](https://registry.hub.docker.com/repos/nsqio/). This blog was created prior to the the release of the official images.

This post details how to connect nsqd and nsqlookupd.  Nsqd and nsqlookupd will be running on separate
Docker containers.

# Links

* [nsq](http://nsq.io/) , [docker](http://www.docker.com/)
* nsqd [image](https://registry.hub.docker.com/u/rexposadas/nsqd/). 
* nsqlookupd [official image](https://registry.hub.docker.com/u/mreiferson/nsqlookupd/)


# Steps

The steps below was derived from the nsq [quick start](http://nsq.io/overview/quick_start.html) page.


## Run nsqlookupd.  

These are instructions straights from the official nsqlookupd docker image. 

	docker pull mreiferson/nsqlookupd	
	docker run --name lookupd -p 4160:4160 -p 4161:4161 mreiferson/nsqlookupd

## Run nsqd. 

First, let's get our docker ip. 

	ifconfig | grep addr

Find the docker ip from the list given in the command above. 

Second, let's run the nsqd container.  We will use a non-official container. This will allow us to connect to nsqlookupd from our container. 

The repository has [usage information](https://registry.hub.docker.com/u/rexposadas/nsqd/) in the docker index. 

	docker pull rexposadas/nsqd
	docker run --name nsqd -p 4150:4150 -p 4151:4151 \ 
	-e BROADCAST_ADDRESS=my.public.host.ip \
	-e LOOKUPD_ADDRESS=<host>:<port> \
	rexposadas/nsqd

Set `LOOKUPD_ADDRESS` to the docker IP and the TCP port of nsqlookupd. i.e. `dockerIP:4160`.

Let's say that my docker IP is `172.17.42.1` 

	docker run --name nsqd -p 4150:4150 -p 4151:4151 \
	-e BROADCAST_ADDRESS=172.17.42.1 \
	-e LOOKUPD_ADDRESS=172.17.42.1:4160 \ 
	rexposadas/nsqd

Note that I am using port 4160. That is the port exposed when we started the nsqlookupd container.  It is the default port for nsqlookupd. Take a look a the nsq [quick start](http://nsq.io/overview/quick_start.html). This can be changed obviously, but for now, keep it simple.


## Testing our setup. 

On one terminal watch nsqdlookupd. The command below lists all the topics nsqlookupd knows.

	watch -n 0.5 "curl -s curl http://172.17.42.1:4161/topics"

On another terminal watch nsqd with the use of [nsq_stat](http://nsq.io/components/utilities.html).

	watch -n 0.5 "curl -s http://127.0.0.1:4151/stats"

Create a topic.
	
    curl -d 'hello world 1' 'http://172.17.42.1:4151/put?topic=test'
 

You should now see the "test" topic in nsqlookupd. The terminal which you watch nsqlookupd should displaying something like this: 

	{"status_code":200,"status_txt":"OK","data":{"topics":["test"]}}

Watching nsqd should yield an output like so: 

	 [test           ] depth: 1     be-depth: 0     msgs: 1     

We can also see changes when we add a topic `test2`

	curl -d 'hello world 2' http://172.17.42.1:4151/put?topic=test2


Watching nsqlookupd and nsqd yields something similar to: 

	{"status_code":200,"status_txt":"OK","data":{"topics":["test","test2"]}}

	-----

	nsqd v0.2.28 (built w/go1.2.1)
	[test           ] depth: 1     be-depth: 0     msgs: 1       
	[test2          ] depth: 1     be-depth: 0     msgs: 1       
