# Using Docker
Contains step-by-step procedure to create Dockerfile, custom images and deploy containers to behave as virtual hosts

#### TUTORIAL ON HOW TO USE DOCKER

Step-1: Create a new Dockerfile
	
	$ mkdir -p nginx-image;  	//create fresh directory
	$ cd nginx-image/  		//open the location
	$ touch Dockerfile		//creates new Dockerfile 
	
	$ vim Dockerfile		//opens it using vim editor
	

Step-2: Initialize an image 

	you can copy a base image and make changes to it as required. here we are copying the base image from ubuntu:trusty distribution
	
	$ FROM ubuntu:trusty		//copies base image
	
	$ RUN apt-get update && apt-get install -y iperf3
	$ RUN apt-get install -y tcpdump && mv /usr/sbin/tcpdump /usr/bin/tcpdump
	$ RUN apt-get install -y hping3
	

Step-3: Create image by building it

	 $ sudo docker build -t endpoint:latest
	
	- This creates an image named "endpoint" using Dockerfile.
	- Navigate to the directory where Dockerfile is located before executing this command.

Step-4: Create containers for that image

	`$ sudo docker run -d --privileged --name=int --net=none endpoint:latest tail -f /dev/null`
	
	- Creates docker container named "int" which is your host
	- Disable default (docker) networking using "--net=none". this is because we are builing out own network.
	- Container should run with "priveleged" flag to allow running of system calls by guest
	- "-d" flag makes the container run as a daemon
	- Container needs to be alive for the duration of the experiment so make it do something useless like monitoring the contents of an empty file keeping it alive forever
