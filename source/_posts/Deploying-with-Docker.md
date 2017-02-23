---
title: Deploying an app with Docker
tags:
  - docker
  - devops
  - containerization
date: 2017-02-22 21:22:42
---

![There goes my app, safe in its little container... Image credit: Miguel Cardona Jr.](/images/2017/02/ship-at-sea.jpg)

In this post I take a look at deploying an app using Docker. If you're not sure what Docker is or why it's cool, have a read through of my low-down on Docker [here](/2017/02/22/Docker-Thinking-inside-the-box/).

<!-- more -->

<h3>First... Deploying your app without Docker</h3>

Here's one way to deploy your app **without** Docker: You write your code, you push it up to GitHub. Then you need a host. If you are using Digital Ocean, you go and create a droplet based on the configuration your app will need. You might also have to install some global dependencies.

This configuration step is important. Some apps will need to run on a specific operating system. Some will have global dependencies (e.g. Node) that will need to be installed. You may well even need a specific *version* of that OS and those dependencies.

Once your host is configured, you can pull down your code from GitHub (or {% link set up auto-deployment https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps %}), and cross your fingers that the app runs the same on your host as it did on your laptop.

Below is a basic diagram of this method:

![](/images/2017/02/one-way-to-deploy.png)

<h3>Deploying your app with Docker</h3>

Docker removes the finnicky step of configuring your host. You don't need to worry about making sure the host has the types and versions of operating systems and global dependencies that your app needs, because your Docker container will have everything your app needs to run on any system. (Well, any system that's running a Docker Engine.)

Here's one way that you can deploy with Docker:

1. Write code
2. Create Dockerfile
3. Build image
4. Push image to Docker Hub
5. Get a host
6. Pull down image to your host
7. Create and run a container

{% blockquote %}
NOTE: In case you're confused about the terminology, an 'image' is basically the blueprint that lets you create containers. A 'container' is a running instance of an image.
{% endblockquote %}

![](/images/2017/02/deploy-with-docker.png)

<h4>Step 1. Write your code</h4>

I won't help you with building your app in this post, other than to give you some encouragement... You got this!

<h4>Step 2. Create a Dockerfile (and maybe a .dockerignore file)</h4>

Create a file called Dockerfile in the root directory of your app. Your Dockerfile is a list of all the things you want to put into your image: files, environment dependencies and commands.

Here's an example Dockerfile:
{% codeblock %}
# Tell Docker what image you want your image to be based on.
# Docker will look locally for this, and if it can't find
# it, then Docker will look for it on hub.docker.com
FROM node:argon

# Create app directory inside the container and set it to
# the working directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# COPY over package.json first to install app dependencies
COPY package.json /usr/src/app/
RUN npm install

# Copy the rest of our files across
COPY . /usr/src/app

# Install vim so we can add our secret API key manually
RUN apt-get update
RUN apt-get install -y vim

# Expose the port that you want to use. (In our case it
# should match the port specified in the server.js file)
EXPOSE 8080

# Finally, run 'npm start'
CMD [ "npm", "start"]
{% endcodeblock %}

The comments in the above codeblock should explain what's happening clearly enough. For more info, check out the {% link Dockerfile reference docs https://docs.docker.com/engine/reference/builder/ %}.


<h4>Step 3. Build an image</h4>

When you go to build your image, Docker will find the Dockerfile and use it as instructions on how to build the image.

To build an image, we use <code>docker build</code> from within the root directory of our app:
{% codeblock %}
docker build -t lukewilson/funky_app_image .
{% endcodeblock %}

The <code>-t</code> flag tells docker to tag this image with the name <code>lukewilson/funky_app_image</code> (the naming convention is your docker username followed by the image name).

We end our <code>docker build</code>command by giving it the context <code>.</code> which sends the context (i.e. your app's root directory and everything in it) to the Docker daemon so it can build your image. Don't forget the dot!

<h4>Step 4. Push your image up to Docker Hub</h4>

To achieve this, you'll need to be logged in to Docker. Just type the following, and then enter your Docker account creds at the prompts.
{% codeblock %}
docker login
{% endcodeblock %}

Then you can just push up your image to Docker Hub by typing the following (swapping in your own image's name):
{% codeblock %}
docker push lukewilson/funky_app_image
{% endcodeblock %}

<h4>Step 5. Get a host that's running Docker Engine</h4>

A bunch of platforms currently support Docker, including {% link AWS https://aws.amazon.com/docker/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=docker_b&sc_content=docker_general_e&sc_detail=aws%20docker&sc_category=docker&sc_segment=88859590122&sc_matchtype=e&sc_country=US&s_kwcid=AL!4422!3!88859590122!e!!g!!aws%20docker&ef_id=WK4@YQAABXqc4jEG:20170223014401:s %} and {% link Digital Ocean https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-docker-application %}. At the time of writing, {% link Heroku's Docker support https://devcenter.heroku.com/articles/container-registry-and-runtime %} is in beta.

<h4>Step 6. Pull down your image from Docker Hub to your host</h4>

I like Digital Ocean for getting up and running with a minimum of fuss, so all I do is SSH into my droplet, and type the below command to pull down the latest version of the image:

{% codeblock %}
docker pull lukewilson/funky_app_image
{% endcodeblock %}

{% blockquote %}
Truth be told, you don't actually need to do this step before you create a container (see step 7), because when you give <code>docker run</code> an image's name, it will try to find it locally first. If Docker can't find the image, Docker will search for it on {% link hub.docker.com https://hub.docker.com %}.

However, you would need to do this step if you wanted to update an already existing container. The process would be pull down the updated image, then delete the current container, then recreate a new container based on the just-pulled-down updated version of the image.
{% endblockquote %}

<h4>Step 7. Create a container based on that image, and run it</h4>

{% codeblock %}
docker run --name funky_app_container -d -p 80:8080 lukewilson/funky_app_image
{% endcodeblock %}

This tells Docker to create and run a new container with the name <code>funky_app_container</code>.

The <code>-d</code> flag is short for 'detach' and tells Docker to run the container in the background.

The <code>-p</code> flag publishes a container's port (e.g. 8080) to the host (80). So anything that goes to the host's port 80 will be directed to the container's port 8080.

Finally, we pass the name of the image that we want Docker to use as a blueprint in creating our container (<code>lukewilson/funky_app_image</code>).

And that's it! If you go to your droplet's IP address, you should see your deployed app!

Lastly, here are some commands you might find helpful while using Docker:

{% codeblock %}
Docker: Helpful commands
----

----------------------------------------------------------------
Installing docker engine
----------------------------------------------------------------
https://docs.docker.com/engine/installation/
Once it's downloaded, make sure it's running (on OSX should see whale icon in top task bar)


----------------------------------------------------------------
Checking versions
----------------------------------------------------------------
  docker --version
  docker-compose --version


----------------------------------------------------------------
Login
----------------------------------------------------------------
docker login
(then enter creds at prompts)


----------------------------------------------------------------
Pulling images
----------------------------------------------------------------
To pull down an image from docker hub:
  docker pull <image name>
  e.g. docker pull nginx


----------------------------------------------------------------
Create a container from existing image
----------------------------------------------------------------
To create and run a container:
  docker run --name <give it a name> -p <host port>:<container port> <image-name>
e.g.
  docker run --name my-nginx -p 80:3003 nginx

If you want to run it as a daemon (in the background), add a -d flag
e.g.
  docker run --name my-nginx -d -p 80:3003 nginx


----------------------------------------------------------------
Starting and stopping container from existing image
----------------------------------------------------------------
If you want to start or stop a container:
  docker start <container name>
  docker stop <container name>

e.g.
  docker stop my-nginx


----------------------------------------------------------------
Finding existing images and containers on a machine
----------------------------------------------------------------
To find existing images:
  docker images

To find containers currently running on a machine:
  docker ps

To find all containers on a machine (running or not):
  docker ps -a


----------------------------------------------------------------
Updating images
----------------------------------------------------------------
To update an image:
1. Finish coding

2. Build new image locally
From within the base directory of your code (where the Dockerfile should be):
  docker build -t <username/image-name> .
NOTE: Don't forget the period at the end!
NOTE: The -t is --tag i.e. the name that your image will be tagged with in Docker hub.

3. Login
docker login
//enter creds

3. Push image up to docker hub
  docker push <username/image-name>



----------------------------------------------------------------
Updating an existing container
----------------------------------------------------------------
To update a container:
1. Pull down the new image from docker hub
docker pull <image_name>

2. Stop the current container
docker stop <container_name>

3. Delete the current container
docker rm <container_name>

4. Create and run a new container based on the new image you pulle down
docker run --name <container_name> -d -p <host port>:<container port> <image_name>
{% endcodeblock %}

