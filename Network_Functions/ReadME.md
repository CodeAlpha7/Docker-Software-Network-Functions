# Creating Network Functions
Here, we will learn how to create Network Functions in Software and deploy them as middleboxes while also learning how they are deployed in an enterprise network.
We will be creating 2 Network Functions:
- Firewall
- Network Address Translator (NAT)
<br>
The topology will be: <br>
Host --> Middlebox (NF) <-- Host
<br><br>
Hosts and Middleboxes will both be deployed as docker containers but since they perform different functions they will have different images.
We have already made image for hosts named "endpoint". We will use the same to make hosts.<br><br>

### Creating Docker image for Middlebox
Our aim is to create containers for Firewall and NAT from this same new image. Both have the same image, but altered respective containers to perform their specific functions.
<br>
From the procedure mentioned before, we create another Dockerfile and create a new image "nf", short for Network Function. The contents of this new Dockerfile are:
<pre><code>FROM ubuntu:trusty

RUN apt-get update && apt-get install -y iptables-persistent
RUN apt-get install -y wondershaper tcpdump
RUN mv /usr/sbin/tcpdump /usr/bin/tcpdump</code></pre>

<br>
Now, create the Docker image "nf" using the command:
<pre><code>docker build -t nf:latest .</code></pre>

<br><br>
## Implementing Firewall
We will be creating the following topology:<br>
![Screenshot from 2021-05-11 22-34-57](https://user-images.githubusercontent.com/72344834/117856486-48786900-b2a9-11eb-8a0d-274e266b87ba.png)

<br>
From the previous tutorials we have seen how to create containers and connect them using switches. But let us still sum up everything to make work easy. <br>

#### Create Open VSwitch Virtual Switches
<pre><code>
sudo ovs-vsctl add-br ovs-br1
sudo ovs-vsctl add-br ovs-br2
</code></pre>
The switch "ovs-br1" already exists for those who have done the previous tutorials
<br><br>

#### Create Host containers from "endpoint" image
<pre><code>
docker run -d --privileged --name=int --net=none endpoint:latest tail -f /dev/null
docker run -d --privileged --name=ext --net=none endpoint:latest tail -f /dev/null
</code></pre>
We have already done this before but created containers named "host1" and "host2"
<br><br>

#### Create Middlebox container from "nf" image
<pre><code>
docker run -d --privileged --name=firewall --net=none nf:latest tail -f /dev/null
</code></pre>

#### Connect Hosts to OVS Switches
<pre><code>
sudo ovs-docker add-port ovs-br1 eth0 int --ipaddress=192.168.1.2/24
sudo ovs-docker add-port ovs-br2 eth0 ext --ipaddress=145.12.131.92/24
</code></pre>

#### Connect middlebox to OVS switches
<pre><code>
sudo ovs-docker add-port firewall eth0 ovs-br1 --ipaddress=192.168.1.1/24
sudo ovs-docker add-port firewall eth1 ovs-br2 --ipaddress=145.12.131.74/24
</code></pre>

#### Setup Routes on endpoint Hosts
<pre><code>
docker exec int route add -net 145.12.131.0/24 gw 192.168.1.1 dev eth0
docker exec int route add -net 192.198.1.0/24 gw 145.12.131.74 dev eth0
</code></pre>
These routes enable endpoint host "int" to know how to send packets to the endpoint host "ext" and vice versa via an intermediate gateway.

#### Enable Network Forwarding on Firewall Container
<pre><code>
docker exec firewall sysctl net.ipv4.ip_forward=1
</code></pre>
