# Creating Network Functions
Here, we will learn how to create Network Functions in Software and deploy them as middleboxes while also learning how they are deployed in an enterprise network. We implement this mainly using the Linux Utilities, "iptables" and "route". <br>
We will be creating 2 Network Functions:
- Firewall
- Network Address Translator (NAT)
<br>
Network functions are always deployed at network chokepoints which enables all incoming/outgoing traffic to flow through these middleboxes before entering the network.<br>

The topology will be: <br>
Host --> Middlebox (NF) <-- Host
<br><br>
Hosts and Middleboxes will both be deployed as docker containers but since they perform different functions they will have different images.
We have already made image for hosts named "endpoint". We will use the same to make hosts.<br><br>

### Creating Docker image for Middlebox
Our aim is to create containers for Firewall and NAT from this same new image. Both have the same image, but altered respective containers to perform their specific functions. So, we selectively configure the container to behave as either a firewall or NAT.
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
Firewall is a network security device used to protect private networks from malicious traffic by using rules to either accept, reject or simply drop incoming packets. This filtering is done by matching packet headers against rule-table by comparing src/dst IP addresses, src/dst ports and protocol used such as UDP/TCP/ICMP.<br><br>
We will be creating the following topology:<br>
![Screenshot from 2021-05-11 22-34-57](https://user-images.githubusercontent.com/72344834/117856486-48786900-b2a9-11eb-8a0d-274e266b87ba.png)

<br>
Each container behaves as a separate virtual host. One such container placed at the network edge will serve as our Firewall. We communicate between 2 containers using switches - for which we will use Open VSwitch (or OVS). Here's how to set them up. <br>

#### Create Open VSwitch Virtual Switches
<pre><code>
sudo ovs-vsctl add-br ovs-br1
sudo ovs-vsctl add-br ovs-br2
</code></pre>
The switch "ovs-br1" already exists if we have run other examples prior to this.
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

#### Configure Firewall to block incoming packets
<pre><code>
docker exec firewall iptables -A FORWARD -i eth1 -p tcp --destination-port 22 -j DROP
</code></pre>
All incoming packets on destination port 22 are blocked.

<br><br>
Verify reachability between endpoint hosts using "ping" <br>
Use Linux tool "nc" to generate TCP traffic and use "tcpdump" to record packet traces on endpoints.<br><br>

#### Start Listening on destination port 22
<pre><code>nc -l 22</code></pre>

#### Initiate Connection to destination port on dst host
<pre><code>nc 192.168.1.2 22 </code></pre>

<br>
Send data on TCP connection by typing plaintext<br>
we have blocked all traffic on port 22 using firewall, so tcpdump should show this.

<br><br>
## Implementing NAT
NAT stands for Network Address Translation which maps one IP address space to another like private network IP space into public IPs. It is useful since publicly-addressable IPs are limited. So, we can have a small number of public IPs with a very large internal network where the NAT multiplexes this small set of public IPs to larger set of internal IPs.
