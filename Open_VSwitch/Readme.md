## Testing Container Connectivty with Open VSwitch

Now, Let's run docker containers as virtual hosts and run Open VSwitch to implement virtual network topology. 
We can connect hosts to switches and test connectivity between them. Here we are implementing the following topology: <br>
#### Host --> Switch <-- Host
<br>
![Alt text](https://github.com/CodeAlpha7/docker/blob/main/Open_VSwitch/Basic_topology.png)
<br>

### Creating Virtual Hosts using Docker
From the previous demo, we have already created a dockerfile and an image. Based on that image we can run multiple hosts. <br>
After starting the terminal, run this command to start the docker service: <br>
          <pre><code> sudo systemctl start docker </code></pre> <br>

Now, create 2 containers for that image we have created:
<pre><code>docker run -d --privileged --name=host1 --net=none endpoint:latest tail -f /dev/null 
docker run -d --privileged --name=host2 --net=none endpoint:latest tail -f /dev/null </code></pre>
We have now created 2 virtual hosts "host1" and "host2"

<br><br>

### Creating Virtual Switch using Open VSwitch
We have created 2 containers based on previous image. Now start the Open VSwitch service using the command:
<pre><code> sudo /usr/share/openvswitch/scripts/ovs-ctl start </code></pre>
<br>
Create a virtual switch:
<pre><code> sudo ovs-vsctl add-br ovs-br1 </code></pre>
Here we created a switch named "ovs-br1"
<br><br>

### Creating Topology
Now connect the containers to the virtual switch:
<pre><code> sudo ovs-docker add-port ovs-br1 eth1 host1 --ipaddress="192.168.1.2/24"
 sudo ovs-docker add-port ovs-br1 eth2 host2 --ipaddress="192.168.1.3/24" </code></pre>

Here, we create a port "eth1" on container "host1" with an IP address "192.168.1.2/24" that connects to virtual switch "ovs-br1" <br>
With this we have connected host1 to host2 via switch "ovs-br1"
<br><br>

### Testing Connectivity
Now, execute a command on running container:
<pre><code> docker exec -it host1 ping -c 3 192.168.1.3 </code></pre>

here, we send 3 ICMP requests to host 192.168.1.3 from container "host1"
