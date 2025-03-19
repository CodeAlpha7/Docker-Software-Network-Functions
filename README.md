# Using Docker
Contains base docker images used to build containers representing virtual hosts with steps to communicate between hosts using Open VSwitch. Explains how to build software network functions using docker. 

Contains step-by-step procedure to create Dockerfile, custom images and deploy containers to behave as virtual hosts

### TUTORIAL ON HOW TO USE DOCKER

#### Step-1: Create a new Dockerfile
	
	$ mkdir -p nginx-image;  	//create fresh directory
	$ cd nginx-image/  		//open the location
	$ touch Dockerfile		//creates new Dockerfile 
	
	$ vim Dockerfile		//opens it using vim editor
	

#### Step-2: Initialize an image 

You can copy a base image and make changes to it as required. <br>
Here we are copying the base image from ubuntu:trusty distribution
	
	$ FROM ubuntu:trusty		//copies base image
	
	$ RUN apt-get update && apt-get install -y iperf3
	$ RUN apt-get install -y tcpdump && mv /usr/sbin/tcpdump /usr/bin/tcpdump
	$ RUN apt-get install -y hping3
	

#### Step-3: Create image by building it

	 $ sudo docker build -t endpoint:latest
	
- This creates an image named "endpoint" using Dockerfile.
- Navigate to the directory where Dockerfile is located before executing this command.

#### Step-4: Create containers for that image

	$ sudo docker run -d --privileged --name=int --net=none endpoint:latest tail -f /dev/null
	
- Creates docker container named "int" which is your virtual host.
- Create as many hosts as you want by changing name.
- Disable default (docker) networking using "--net=none". this is because we are builing out own network.
- Container should run with "priveleged" flag to allow running of system calls by guest
- "-d" flag makes the container run as a daemon
- Container needs to be alive for the duration of the experiment so make it do something useless like monitoring the contents of an empty file keeping it alive forever
<br>

### Removing Containers
The above command will create a container, but it will keep running in the docker server if not stopped.<br>
You also cannot create another container with the same name.

#### To check all existing containers
<pre><code>docker container ls -a</code></pre>
This gives a list of all existing containers with their container_id, image, names and status.

#### To STOP existing container
<pre><code>docker container stop f5a94c143c2a </code></pre>
Here, syntax is [container_id] instead of f5a94c143c2a

#### To REMOVE am existing container
<pre><code>docker container rm f5a94c143c2a </code></pre>
