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
<br>
We have already made image for hosts named "endpoint".

### Creating Docker image for Middlebox
Our aim is to create containers for Firewall and NAT from this same new image. Both have the same image, but altered respective containers to perform their specific functions.
<br>
From the procedure mentioned before, we create another Dockerfile and create a new image "nf", short for Network Function. The contents of this new Dockerfile can be viewed from the file uploaded in this folder.
