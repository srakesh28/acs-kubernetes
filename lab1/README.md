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

# a.	Installing Docker locally on your machine

•	For windows installations, follow instructions at https://docs.docker.com/docker-for-windows/

•	For Mac machines, follow instructions at https://docs.docker.com/docker-for-mac/ 

•	Another option is to start a VM on your Azure subscription using Ubuntu or centOS image and installing Docker using Linux instructions at https://docs.docker.com/engine/installation/linux/ubuntu/ to ensure you have the latest docker version.

# b.	Running a Simple App in a Container locally

Open a command-line terminal, and run some Docker commands to verify that Docker is working as expected. 

   Check the latest version of docker is installed using:
   $> docker version

   Run docker ps and docker run hello-world to ensure the daemon is working as expected
   $> docker ps
    
 Now, you can run a simple hello-world container using “docker run.” This command will:
 
    a) check to see if you had the hello-world software image
    b) downloaded the image from the Docker Hub
    c) loaded the image into the container and “ran” it
  
   $> docker run hello-world
    
 Now, we will run a simple webserver on your local machine using the nginx docker image. 
 
 Run a simple webserver in a container on your local machine
$> docker run -d -p 80:80 --name webserver nginx

 In a web browser, go to http://localhost/ to bring up the home page. 

 Run docker ps while your web server is running to see details on the webserver container.
 $> docker ps 
    CONTAINER ID        IMAGE                COMMAND                  CREATED              STATUS              PORTS                              NAMES
    56f433965490        nginx                "nginx -g 'daemon off"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, 443/tcp   webserver

 If you want to stop the webserver, type: docker stop webserver and start it again with docker start webserver
 $> docker stop webserver
 $> docker start webserver

To stop and remove the running container with a single command, type: docker rm -f webserver.
$> docker rm -f webserver


# c.	Creating and Running Docker images using a Dockerfile 

Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using “docker build” users can create an automated build that executes several command-line instructions in succession.

In the following steps, you will create a docker file, to use in building an image and then run the new image on your local machine

First step is to create a new directory to include all you need to build the image.

On MacOS
$> mkdir mydockerbuild; cd mydockerbuild
On Windows
c:\ md mydockerbuild
c:\ cd mydockerbuild


Now, you are ready to create a new text file named Dockerfile to include details about your image. For example, copy and paste the following Dockerfile to your directory.  

FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay


For this exercise, we used the Whalesay image, which is based on Ubuntu Liunx distribution. Adding a RUN statement updates all packages in your image and install the fortunes program into the image. Finally, adding a CMD statement tells the image the final command to run after its environment is set up. This command runs fortune -a and sends its output to the cowsay command.

Now, you are ready to build an image from your docker file. The -t parameter gives your image a tag, so you can run it more easily later. Don’t forget the . command, which tells the docker build command to look in the current directory for a file called Dockerfile.

$> docker build -t docker-whale . 

You can now check that you have this new image on your local machine using docker images

$> $ docker images

REPOSITORY           TAG          IMAGE ID          CREATED             SIZE
docker-whale         latest       c2c3152907b5      4 minutes ago       275.1 MB
docker/whalesay      latest       fb434121fc77      4 hours ago         247 MB
hello-world          latest       91c95931e552      5 weeks ago         910 B

Now, you are ready to run your new docker image using docker run
$ docker run docker-whale
 ______________________________________
< You will be successful in your work. >
 --------------------------------------
        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/

# d.	Defining and running multi-container Docker applications using Docker compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a Compose file to configure your application’s services. Then, using a single command, you create and start all the services from your configuration.

Using Compose is basically a three-step process.

1.	Define your app’s environment with a Dockerfile so it can be reproduced anywhere.

2.	Define the services that make up your app in docker-compose.yml so they can be run together in an isolated environment.

3.	Lastly, run “docker-compose up” and Compose will start and run your entire app.


In this exercise, we will create a python app that reads from a redis running in a different container.

1. Create a directory for the project:
$> mkdir composetest
$> cd composetest

Verify compose is installed
$> docker-compose version

2. Create a file called app.py in your project directory and paste this in:
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)

3. Create another file called requirements.txt in your project directory and paste this in, to define the application’s dependencies:
flask
redis

4. Create a Dockerfile that builds a Docker image. The image contains all the dependencies the Python application requires, including Python itself. In your project directory, create a file named Dockerfile and paste the following:
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]

This tells Docker to:
Build an image starting with the Python 3.4 image.
Add the current directory . into the path /code in the image.
Set the working directory to /code.
Install the Python dependencies.
Set the default command for the container to python app.py

5. Define your services in a Compose file by creating a file called docker-compose.yml in your project directory and paste the following:

version: '2'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"

This Compose file defines two services, web and redis. The web service:
Uses an image that’s built from the Dockerfile in the current directory.
Forwards the exposed port 5000 on the container to port 5000 on the host machine.
Mounts the project directory on the host to /code inside the container, allowing you to modify the code without having to rebuild the image.
The redis service uses a public Redis image pulled from the Docker Hub registry.

6. Build and run your app with Compose from your project directory. Start up your application using docker-compose up.

 $> docker-compose up
 Pulling image redis...
 Building web...
 Starting composetest_redis_1...
 Starting composetest_web_1...
 redis_1 | [8] 02 Jan 18:43:35.576 # Server started, Redis version 2.8.3
 web_1   |  * Running on http://0.0.0.0:5000/
 web_1   |  * Restarting with stat


Compose pulls a Redis image, builds an image for your code, and start the services you defined.
Enter http://0.0.0.0:5000/ in a browser to see the application running.
If you’re using Docker on Linux natively, then the web app should now be listening on port 5000 # on your Docker daemon host. If http://0.0.0.0:5000 doesn’t resolve, you can also try  http://localhost:5000.
If you’re using Docker Machine on a Mac, use docker-machine ip MACHINE_VM to get the IP address of your Docker host. Then, open http://MACHINE_VM_IP:5000 in a browser.
You should see a message in your browser saying:
Hello World! I have been seen 1 times.
Refresh the page.
The number should increment.

7. Typing control-c will shutdown the containers.  If you started Compose with docker-compose up -d, you’ll probably want to stop your services once you’ve finished with them:

$> docker-compose stop

# e.	Running Sample App locally
Now you are ready to run the CAT sample app on your local machine. This sample app is a ASP .NET app which you will also use on Azure with the Orchestrator in the second portion of the lab. This code material are available at https://github.com/dave-read/container-service-dotnet-continuous-integration-multi-container. 

Clone the repo from github into a new local directory on your machine using: 

$> mkdir ASPNETAPP
$> cd ASPNETAPP  
$> git clone https://github.com/dave-read/container-service-dotnet-continuous-integration-multi-container 
$> cd container-service-dotnet-continuous-integration-multi-container

Now, compile the ASP .NET Core application code. This uses a container to isolate build dependencies that is also used by VSTS for continuous integration:

docker-compose -f docker-compose.ci.build.yml run ci-build

On Windows, you currently need to pass the -d flag to docker-compose run and poll the container to determine when it has completed). Note: if you get an error about not being able to access a shared folder, open the Docker for Windows settings from the system tray, and in the Shared Drives tab, ensure that the drive where your lab files are stored is checked (enabled).

docker-compose -f docker-compose.ci.build.yml run –d ci-build

Now build Docker images and run the services:

docker-compose up --build

The frontend service (service-a) will be available at http://localhost:8080.
  

