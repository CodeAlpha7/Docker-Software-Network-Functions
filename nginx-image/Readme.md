This is the Dockerfile containing the script for the image.
The image is formed by the script written inside this Dockerfile.
This script is the image based on which containers are created.

The below command copies a base image from the ubuntu:trusty distribution
$ FROM ubuntu:trusty

RUN commands are used to make changes to the base image to create your own custom image.
Many more commands exist which can be used to alter your own custom image.
