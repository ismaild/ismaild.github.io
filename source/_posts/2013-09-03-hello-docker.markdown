---
layout: post
title: "Hello docker"
date: 2013-09-03 01:06
comments: true
categories: [docker, mongodb, saas]
keywords: "docker, mongodb, saas"
description: "A docker tutorial to build a mongoDB service"
---
I have been playing around recently with [Docker](http://www.docker.io). It could really simplify deployments, creating SAAS services or even creating your own personal PAAS. 

### What is docker?

> Docker is an open-source project to easily create lightweight, portable, self-sufficient containers from any application. The same container that a developer builds and tests on a laptop can run at scale, in production, on VMs, bare metal, OpenStack clusters, public clouds and more. 


### Containers vs Virtual Machines

**Virtual Machines:** require a complete operating system image, with allocated resources to run. They take a long time to bootup, and have quite a bit of overhead.

**Containers:** are much more lightweight, since there is no overhead of a complete virtual environment, with the kernel managing the memory and access to the file system. This also means you can bootup an application in seconds.

### Install docker

The simplest method would be to use vagrant to set it up on Mac or Linux. (http://docs.docker.io/en/latest/installation/vagrant/)

#### Setup with vagrant
{% codeblock lang:bash %}
git clone https://github.com/dotcloud/docker.git
cd docker
vagrant up
vagrant ssh
{% endcodeblock %}

#### Manual Setup
- Install virtualbox - (https://www.virtualbox.org)
- Launch a new VM, and install ubuntu 12.04 (64 bit) - (http://www.ubuntu.org)

Docker works best with the 3.8 kernel due to a bug in lxc which can cause some issues if you are on 3.2.  run `uname -r` to check which version you are on, and run the following if you are not on 3.8

{% codeblock lang:bash %}
sudo apt-get update

sudo apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring

sudo reboot
{% endcodeblock %}

Then run...

{% codeblock lang:bash %}
sudo apt-get update

sudo apt-get upgrade

sudo apt-get install python-software-properties git-core build-essentials ssh

sudo apt-get update

sudo add-apt-repository ppa:dotcloud/lxc-docker

sudo apt-get install lxc-docker
{% endcodeblock %}

Now, hello world... docker style.

{% codeblock lang:bash %}
dock@saas:~$ sudo docker run ubuntu /bin/echo hello world
[sudo] password for dock: 
Pulling repository ubuntu
Pulling image 8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c (precise) from ubuntu
Pulling image b750fe79269d2ec9a3c593ef05b4332b1d1a02a62b4accb2c21d589ff2f5f2dc (quantal) from ubuntu
Pulling 27cf784147099545 metadata
Pulling 27cf784147099545 fs layer
Downloading 94.86 MB/94.86 MB (100%)
hello world
{% endcodeblock %}

#### What just happened?
- docker downloaded the base image from the docker index
- it created a new LXC container
- It allocated a filesystem for it
- Mounted a read-write layer
- Allocated a network interface
- Setup an IP for it, with network address translation
- And then executed a process in there
- Captured its output and printed it to you

In the next post, we will discuss setting up a [docker container with mongodb](/2013/09/setup-a-docker-container-with-mongodb/).