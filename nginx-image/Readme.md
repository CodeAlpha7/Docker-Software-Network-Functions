## Dockerfile containing image
This is the Dockerfile containing the script for the image. <br>
The image is formed by the script written inside this Dockerfile.<br>
This script is the image based on which containers are created.

The below command copies a base image from the ubuntu:trusty distribution

<pre><code> $ FROM ubuntu:trusty </code></pre>

RUN commands are used to make changes to the base image to create your own custom image.<br>
Many more commands exist which can be used to alter your own custom image.
