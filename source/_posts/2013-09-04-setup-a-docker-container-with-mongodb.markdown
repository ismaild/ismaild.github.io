---
layout: post
title: "Setup a Docker container with MongoDB"
date: 2013-09-04 09:09
comments: true
categories: [docker, mongodb, saas]
keywords: "docker, mongodb, saas"
description: "Setup MongoDB in a docker container"
---
If you have not yet setup docker, follow the [install docker](/2013/09/hello-docker/) post before this. 

First, lets take a look at what docker images we have.

{% codeblock lang:bash %}
dock@saas:~$ sudo docker images
REPOSITORY          TAG                 ID                  CREATED             SIZE
ubuntu              12.04               8dbd9e392a96        4 months ago        131.5 MB (virtual 131.5 MB)
ubuntu              12.10               b750fe79269d        5 months ago        24.65 kB (virtual 180.1 MB)
ubuntu              latest              8dbd9e392a96        4 months ago        131.5 MB (virtual 131.5 MB)
ubuntu              precise             8dbd9e392a96        4 months ago        131.5 MB (virtual 131.5 MB)
ubuntu              quantal             b750fe79269d        5 months ago        24.65 kB (virtual 180.1 MB)
{% endcodeblock %}

Notice we have the images locally, so if we run the hello world the command runs instantly, without the need to download images again. 

Now, lets check if we have and docker containers running:

{% codeblock lang:bash %}
dock@saas:~$ sudo docker ps
ID                  IMAGE               COMMAND             CREATED             STATUS              PORTS
{% endcodeblock %}

As expected, we do not have any containers currently running.

Install MongoDB
---------------

Start an interactive shell in a container with: `sudo docker run -i -t ubuntu /bin/bash`

{% codeblock lang:bash %}
root@f9d75a04ce10:/# apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

root@f9d75a04ce10:/# echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/10gen.list

root@f9d75a04ce10:/# apt-get update

# initctl fix
root@a1f0680f8458:/# dpkg-divert --local --rename --add /sbin/initctl

root@a1f0680f8458:/# ln -s /bin/true /sbin/initctl

root@a1f0680f8458:/# apt-get install mongodb-10gen

root@a1f0680f8458:/# mkdir -p /data/db

root@a1f0680f8458:/# mongod
mongod --help for help and startup options
Tue Sep  3 14:10:36.469 [initandlisten] MongoDB starting : pid=125 port=27017 dbpath=/data/db/ 64-bit host=a1f0680f8458

exit
{% endcodeblock %}

MongoDB starts succesfully in the container. Now we need to commit the container, to save the state and push to the docker index. 

`sudo docker commit a1f0680f8458 codiez/mongodb`

Remember to replace your container id, mine was `a1f0680f8458` which you can see at the prompt, and your `<username>/<container_name>`

{% codeblock lang:bash %}
dock@saas:~$ sudo docker commit a1f0680f8458 codiez/mongodb

dock@saas:~$ sudo docker login
Username: codiez
Password: 
Email: ismail@codiez.co.za
Login Succeeded
dock@saas:~$ sudo docker push codiez/mongodb
The push refers to a repository [codiez/mongodb] (len: 1)
Processing checksums
Sending image list
Pushing repository codiez/mongodb (1 tags)
Pushing 8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c
Image 8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c already pushed, skipping
Pushing tags for rev [8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c] on {https://registry-1.docker.io/v1/repositories/codiez/mongodb/tags/latest}
Pushing f0ab8043e4e8135379d35410a4847769efb9245d8d4817cb24a2196c434c8506
{% endcodeblock %}

Once that completes, we can start MongoDB in the container, and map it to a port by running:

`sudo docker run -d -p 27017 codiez/mongodb /usr/bin/mongod --smallfiles`

This basically runs the container, based on the `codiez/mongodb` image, with the command `/usr/bin/mongod` and maps the default mongodb port 27017 to an external port.

Check that the container is running and get the port:

{% codeblock lang:bash %}
dock@saas:~$ sudo docker ps
ID                  IMAGE                   COMMAND             CREATED             STATUS              PORTS
a1f0680f8458        codiez/mongodb:latest   /usr/bin/mongod     5 seconds ago       Up 4 seconds        49157->27017     
{% endcodeblock %}


You can also inspect the image to grab the port by running `sudo docker inspect container_id`

Now, we can test an external connection to MongoDB, test from your local machine and connect to the VM IP with the port for your container:

{% codeblock lang:bash %}
$ mongo 192.168.0.21:49157
MongoDB shell version: 2.4.5
connecting to: 192.168.0.21:49157/test
> db
test
> db.posts.insert({title:"Hello MongoDB in Docker"})
> db.posts.find()
{ "_id" : ObjectId("5227058d112c68baaa3b94d9"), "title" : "Hello MongoDB in Docker" }
> 
{% endcodeblock %}

*Update*: Stop the container and restart it since we do not want to commit with the test data.

{% codeblock lang:bash %}
dock@saas:~$ sudo docker stop a1f0680f8458
dock@saas:~$ sudo docker run -d -p 27017 codiez/mongodb /usr/bin/mongod --smallfiles
{% endcodeblock %}


One final commit to save the command and port mapping. Also notice you do not have to enter in the entire container id when running commands, just the first few characters. 

{% codeblock lang:bash %}
dock@saas:~$ sudo docker ps
ID                  IMAGE                   COMMAND                CREATED             STATUS              PORTS
298a43e2f98e        codiez/mongodb:latest   /usr/bin/mongod --sm   35 seconds ago      Up 34 seconds       49164->27017        
dock@saas:~$ sudo docker commit -run '{"Cmd": ["/usr/bin/mongod", "--smallfiles"], "PortSpecs": [":27017"]}' 298a4 codiez/mongodb
{% endcodeblock %}

Now that we have an image, we can run `docker pull codiez/mongodb` to grab the container and run it with `docker run -d codiez/mongodb`

The next post will discuss [automating the creation of a container with a dockerfile](/2013/09/setup-mongodb-container-docker-file/).


