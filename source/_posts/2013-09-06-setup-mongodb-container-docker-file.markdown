---
layout: post
title: "Setup a MongoDB container with a docker file"
date: 2013-09-06 15:54
comments: true
categories: [docker, mongodb, saas]
keywords: "docker, mongodb, saas"
description: "Setup up mongodb container using a docker file"
---
In our previous post we went through the steps to [Setup a Docker Container With MongoDB](/2013/09/setup-a-docker-container-with-mongodb/) manually. 

Docker has a simple DSL that lets you automate all of these steps to make a conainer.

###Docker file syntax

Every line in a docker file has the following structure: `INSTRUCTION arguments`

Comments are ignored, and the first line in the docker file should contain the command `FROM <image`

Commands available [full details](http://docs.docker.io/en/latest/use/builder/)

- FROM (select the base image)
- MAINTAINER (Set the author field for images)
- RUN (run a command, and commit)
- CMD (the default execution command for the container)
- EXPOSE (set the port to be publicly exposed)
- ENV (set environment variables)
- ADD (add files from source and copy them to the container)
- ENTRYPOINT (configure the container to run as an executable)
- VOLUME (add a volume)
- USER (set the user)
- WORKDIR (set working directory)

###MongoDB dockerfile
We create a dockerfile, and just use all the same commands we used previously `touch Dockerfile`

{% codeblock lang:text %}
FROM ubuntu:latest

RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/10gen.list

RUN dpkg-divert --local --rename --add /sbin/initctl
RUN ln -s /bin/true /sbin/initctl

RUN apt-get update
RUN apt-get install mongodb-10gen

RUN mkdir -p /data/db

EXPOSE 27017
CMD ["usr/bin/mongod", "--smallfiles"]
{% endcodeblock %}

Then we issue:

`sudo docker build -t codiez/mongodb .`

and start it up with...

`sudo docker run -d codiez/mongodb`