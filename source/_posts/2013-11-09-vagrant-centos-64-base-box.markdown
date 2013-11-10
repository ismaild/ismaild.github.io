---
layout: post
title: "Build a Vagrant CentOS 6.4 base box"
date: 2013-11-09 15:39
comments: true
categories: [vagrant, ansible, virtualbox, oracle]
keywords: "vagrant, ansible, virtualbox, oracle"
description: "Instructions on creating a Vagrant CentOS 6.4 base box"
---
This post will discuss building a CentOs 6.4 vagrant box, and in later posts we will use this box and ansible to further define our machines, such as installing oracle xe.

### About Vagrant

[Vagrant](http://www.vagrantup.com/) lets you create and configure lightweight, reproducible, and portable development environments. All too often you will find teams developing off a central server, purely because of the pain of setting up a dev environment. Nothing is automated. There are many challenges with this approach. IMHO this slows down a team, and experimentation is discouraged and you end up "stepping on each others code". 

Vagrant can solve some these challenges, ever started a new project and spent a few days just getting your environment setup? Well with vagrant this should be a thing of the past.

### Before you start

You will need some baseline software before we begin. Make sure you have the following installed.

* [Git](http://git-scm.com/)
* [Vagrant](http://www.vagrantup.com/)
* [Ansible](http://www.ansibleworks.com/docs/intro_installation.html)
* [Virtualbox](https://www.virtualbox.org/)
* [Veewee](https://github.com/jedi4ever/veewee/blob/master/doc/requirements.md)

Lets check we have everything installed

{% codeblock lang:bash %}
$ git version
git version 1.8.3.4 (Apple Git-47)
$ vagrant --version
Vagrant 1.3.5
$ VBoxManage --version
4.3.2r90405
$ veewee version
Version : 0.3.12 - use at your own risk
{% endcodeblock %}

### Build a CentOS 6.4 base box
You can find a number of [vagrant base boxes online](https://www.google.co.za/search?q=vagrant+boxes), but i prefer creating one from scratch. We can do that easily with veewee.

First we clone the repo: `git clone https://github.com/ismaild/vagrant-boxes.git`. I have disabled chef and puppet since we will be using ansible. Feel free to add it back in if you want it, which you can by editing: `definitions/CentOS-6.4-x86_64/definition.rb`

Veewee will automatically try and download the required ISO files. If you already have them, you can put them in the `iso` directory and it will not need to download.

* http://yum.singlehop.com/CentOS/6.4/isos/x86_64/CentOS-6.4-x86_64-minimal.iso
* http://download.virtualbox.org/virtualbox/4.3.2/VBoxGuestAdditions_4.3.2.iso

Just make sure the version of the virtualbox guest additions matches your installed version of virtualbox.

Then run the following commands:

{% codeblock lang:bash %}
$ cd vagrant-boxes
$ veewee vbox list
# if no boxes defined
$ veewee vbox define CentOS-6.4-x86_64 CentOS-6.4-x86_64-minimal
# Build the box
$ veewee vbox build CentOS-6.4-x86_64
# Eject the disks from the running VM and shutdown.
# Package the box
$ vagrant package --base CentOS-6.4-x86_64 --output CentOS-6.4-x86_64.box
$ vagrant box add centos-64-x86_64 CentOS-6.4-x86_64.box
{% endcodeblock %}

Now we have a base CentOS 6.4 box to use. In the next post we will go through [setting up oracle with ansible](/2013/11/install-oracle-centos-64-vagrant-ansible/).
