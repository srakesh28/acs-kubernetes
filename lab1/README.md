# Contents / Agenda

 Simple App Containerization hands-on lab with Docker

Installing Docker locally

Running a simple app in a Docker container locally

Creating and running Docker images using a Dockerfile

Defining and running multi-container Docker applications using Docker compose


Running Sample App locally

# Lab Goals:

To get a primarily hands-on experience with Docker, Kubernetes and Kubernetes’ main features on Azure.

To set up a Kubernetes deployment in your subscription for future customer-demo purposes. 

To write and run a basic hello-world program on Docker

# 1. Simple App Containerization hands-on lab with Docker

a.	Installing Docker locally on your machine

•	For windows installations, follow instructions at https://docs.docker.com/docker-for-windows/

•	For Mac machines, follow instructions at https://docs.docker.com/docker-for-mac/ 

•	Another option is to start a VM on your Azure subscription using Ubuntu or centOS image and installing Docker using Linux instructions at https://docs.docker.com/engine/installation/linux/ubuntu/ to ensure you have the latest docker version.

b.	Running a Simple App in a Container locally

Open a command-line terminal, and run some Docker commands to verify that Docker is working as expected. 

   # Check the latest version of docker is installed using:
   # $> docker version

  # Run docker ps and docker run hello-world to ensure the daemon is working as expected
  $  $> docker ps
    
 Now, you can run a simple hello-world container using “docker run.” This command will:
 
    a) check to see if you had the hello-world software image
    b) downloaded the image from the Docker Hub
    c) loaded the image into the container and “ran” it
  
  # $> docker run hello-world
    

  

