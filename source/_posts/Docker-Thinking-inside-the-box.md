---
title: 'Docker: Thinking inside the box'
tags:
- docker
- containerization
- devops
---

![When I first figured out what Docker did, I could barely *contain* my excitement. Image credit: Erwan Hesry](/images/2017/02/containers.jpg)

Docker is an awesome app delivery technology. If you've ever wondered why your app works on your machine, but not on your web host, or your mate's laptop, or your other OSX/Windows/Linux machine, then you should check out Docker.

<!-- more -->

Docker is the world's leading software containerization platform. It allows you to create 'containers' on a physical (or virtual) machine. Each container includes the app itself, along with everything the app will need to run... but nothing more.

The beauty of it is that it will run exactly the same on all hosts (as long as those hosts have the Docker engine), irrespective of operating system type, the version of Node installed on the machine, or other environmental variables.

<h3>Containerization vs Virtualization</h3>

*Hang on... isn't this just doing the same thing as a virtual machine?*

Well, both provide an isolated environment in which to run an application. But there are some not-so-subtle differences. Let's quickly compare virtualization and containerization using the houses vs apartments analogy that {% link Docker's Mike Coleman uses https://blog.docker.com/2016/03/containers-are-not-vms/ %}.

![](/images/2017/02/apartment-house.png)

Houses have their own infrastructure: their own plumbing, electricity, heating, gas and water. On top of this, houses need to have certain things to make it a house. At a minimum, they generally have two or more bedrooms, a kitchen, a living room, a yard, and a bathroom. Oh, and a roof.

In contrast, each apartment has access to shared infrastructure (handled by the apartment building). Each apartment uses only what it needs. An apartment doesn't need a roof because the apartment building takes care of that. Apartments come in a wide variety of configurations: from a small studio to a luxury penthouse. Some apartments have full kitchens, some just have 'kitchenettes'. Some will have a balcony, some won't.

So, like apartments in our analogy, containers make use of Linux's resource isolation features to share the resources of the host on which they sit. They don't need their own OS, RAM or CPU. Anything specific that the app needs will be in the container with it but it doesn't need any of the things a VM needs to run, making it much lighter than a VM.

Virtual machines (our houses) will each need a full copy of whatever operating system it needs, as well as a full copy of all the hardware that the OS needs to run... VMs are hungry beasts eating up a lot of RAM and CPU cycles. As a result, they are much 'heavier' to ship.

Additionally, just like an apartment building, a host can have multiple containers running on it, all sharing its resources:

![](/images/2017/02/multi-container.png)

One last important thing to note is that VMs can take minutes to boot, whereas containers can many times be spun up in less than a second. Speedy!

<h3>So, want to give Docker a go?</h3>

Check out the next blog post where we explore [how to deploy an app to Digital Ocean using Docker](/2017/02/22/Deploying-with-Docker/).
