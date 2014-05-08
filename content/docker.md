# Initial work with Docker
=========================



# Stuff I needed to figure out

## Have the docker daemon to listen to a different port.  Docker defaults to a unix socket. 

First kill the current docker that is running. I didn't know how to do it gracefully, so I killed the pid.

	sudo rm /var/docker.pid

Then restart the daemon

	sudo <path to>/docker -H 0.0.0.0:5555 -d &

More reference: http://docs.docker.io/use/basics/#bind-docker-to-another-hostport-or-a-unix-socket
