---
layout: post                                                                                                                  
title:  "NSQ as a broker for LogStash"
date:   2014-08-20
categories: docker nsq logstash elasticsearch kibana
---

*Updated December 5, 2014:* Use official NSQ [`images`](https://registry.hub.docker.com/repos/nsqio/).

This post shows one way to use NSQ as a broker for LogStash.  At the time of this writing LogStash did not come with NSQ support, but it did support TCP input. The plan is to send nsq messages to LogStash via TCP. 

The basics steps to happiness are:

1. Run four Docker images.
2. Feed the consumer.

That's it, really.  All the work has been done for you.  Isn't Docker beautiful?

### Step 1 : Pull the Docker images.

The four docker images are : 
[`nsqd`](https://registry.hub.docker.com/u/nsqio/nsqd/)
[`nsqlookupd`](https://registry.hub.docker.com/u/nsqio/nsqlookupd/)
[`nsq-to-tcp`](https://registry.hub.docker.com/u/rexposadas/docker-nsq-to-tcp/), 
[`LogStash`](https://registry.hub.docker.com/u/rexposadas/docker-logstash/).

Half of these containers are described in the [official docs](http://nsq.io/deployment/docker.html).  Those docs talks about connecting nsqd and nsqlookupd via Docker containers. 

	1. nsqio/nsqlookupd
	2. nsqio/nsqd
	3. rexposadas/docker-logstash
	4. rexposadas/nsq_to_tcp

We will run the containers in the order shown above. 

*PROTIP:** If you want to remove all containers from your system in order to start from a clean slate, you can run:

	docker rm -f `docker ps --no-trunc -aq`

Then run 
	
	docker ps -a 

The output from the last call should give you an empty list. 

### Step 2

In this step we will run the docker containers. 

	Tip:

		If you are using boot2docker you can set your environment variables in a way to make the commands below accurate when copy-and-pasting. 

		I have a docker_env.sh file that looks like this:

		export DOCKER_HOST_IP=192.168.59.103

		# required by boot2docker
		export DOCKER_HOST=tcp://$DOCKER_HOST_IP:2376
		export DOCKER_CERT_PATH=/Users/rexposadas/.boot2docker/certs/boot2docker-vm
		export DOCKER_TLS_VERIFY=1

The official docs for connecting nsqd and nsqlookupd are [here](http://nsq.io/deployment/docker.html).  I will not duplicate documentation here but setting up the connection between nsqd and nsqlookupd is required for the rest of the setup to work. Once you have that working, it's time to get Logstash and the consumer working.

I would run the following commands in separate terminals so that you can see the output.  

	docker run --name logstash -p 514:514 -p 9292:9292 -p 7000:7000 
		-e LOGSTASH_CONFIG_STRING='input {tcp {port => 7000}} output {stdout { }}' 
		rexposadas/docker-logstash

	docker run -d --name consumer 
		-e TOPIC=test 
		-e LOOKUPD_ADDR=$DOCKER_HOST_IP:4161
		-e OUTPUT_TCP_ADDR=$DOCKER_HOST_IP:7000 
		rexposadas/docker-nsq-to-tcp


Using the `--name` flag is not necessary. You can leave it out. But, I find it convenient to see what containers I currently have running. It's also useful when linking continers.  If you don't specify the name of the running created containers, Docker will give it a name. 

The last container, `docker-nsq-to-tcp`, simply takes messages from nsq and forwards them to the TCP port that LogStash is listening on. In our example it's port `7000`.

Regarding setting the LogStash config: I know, I know. You can set the LogStash config via a file. But sometimes I like using the variable I created `LOGSTASH_CONFIG_STRING` since it lets me dynamically create the the configuration.  If you would like to point LogStash to a config file, that will work as well. 

### Testing the Setup:

LogStash took about 15 seconds to get up and running in my machine (4 gigs of RAM, Ubuntu Precise).  You will need to wait until it's finished starting up to see the message printed in stdout.  

In one terminal make the following call:

	curl -d 'hello world 1' 'http://$(DOCKER_HOST_IP):4151/put?topic=test'

You should see LogStash output the message. 


### Bonus section: Setting NSQ as a broker for the ELK stack. 

The ELK stack being ElasticSearch, LogStash, Kibana.

In order to get this up and running we need to do the following:

1. Start the ElasticSearch docker container. 
2. Start LogStash and have it forward it's messages to ElasticSearch
3. Start Kibana and have it point to ElasticSearch.

Let's start the services. Make sure there are no LogStash containers running.  If you ran logstash the way it was described earlier, you can stop log stash by running: 

	docker stop logstash && docker rm logstash

Run elastic search

	docker run -d --name es -p 9200:9200 -p 9300:9300 dockerfile/elasticsearch

	docker run -d --name logstash -link es:es 
		-p 514:514 
		-p 9292:9292 
		-p 7000:7000 
		-e LOGSTASH_CONFIG_STRING='input {tcp {port => 7000}} output {elasticsearch {host => "172.17.42.1"}}' 
		rexposadas/docker-logstash

	docker run -d --name kibana 
		-p 80:80 -e ES_HOST=172.17.42.1 
		-e ES_PORT=9200 arcus/kibana


Notice the `-link` flag when I started LogStash.  For more on linking containers you can read [this](https://docs.docker.com/userguide/dockerlinks/).

Obviously, you can set the Kibana port to something else. All examples here uses the default port. 

### Scripts

For your convenience here is the entire setup in one [script](https://github.com/rexposadas/notes/tree/master/blog/nsqelk). Take a look at `all.sh`.

### Testing our setup

Hit  `http://<yourdocker_host_ip>:80` in your browser.  You should see the Kibana UI.  

Start sending messages to nsq:

	curl -d 'hello world 1' 'http://172.17.42.1:4151/put?topic=test'

Wait a minute.  It takes time for the message to go from the logstash, to elasticsearch to kibana. Refresh the Kibana UI and you should see this message in the UI.  
To make life exciting, get a log consumer like [beaver](https://github.com/josegonzalez/beaver) send messages to NSQ.