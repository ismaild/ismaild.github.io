---
layout: post
title: "Install Oracle on CentOS 6.4 with Vagrant and Ansible"
date: 2013-11-10 01:31
comments: true
categories: [vagrant, ansible, virtualbox, oracle]
keywords: "vagrant, ansible, virtualbox, oracle"
description: "Instructions on how to install oralce on CentOS 64 with Vagrant and Ansible"
---
This post will cover installing oracle xe on CentOS 6.4 using vagrant and ansible. If you have not already, read the first post on [creating a vagrant centos base box](/2013/11/vagrant-centos-64-base-box/)

The source code is available at: https://github.com/ismaild/vagrant-centos-oracle

### About Ansible

[Ansible](http://www.ansibleworks.com/) is an IT orchestration engine/configuration management system, that lets you easily describe how you would like your servers to look and then automate it. It differs from Chef and Puppet, as it requires nothing to be installed on the server, it uses ssh. Tasks are described in yml, which can be written and read even by non programmers. 


### Vagrant init

Now that we have the [CentOS 6.4 base box](/2013/11/vagrant-centos-64-base-box/), we can add it to vagrant with: 

{% codeblock lang:bash %}
$ vagrant box list
centos-64-x86_64 (virtualbox)
{% endcodeblock %}

Vagrant uses a `Vagrantfile` to describe the type of machine you would like to build, and the file should be stored in the root directory of your source code. Next we create the `Vagrantfile`

{% codeblock lang:bash %}
$ mkdir ../vagrant-centos-oracle
$ cd ../vagrant-centos-oracle
$ vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
{% endcodeblock %}

Edit the `Vagrantfile` and change `config.vm.box` to: 

{% codeblock lang:ruby %}
  config.vm.box = "centos-64-x86_64"
{% endcodeblock %}

Now we run:

{% codeblock lang:bash %}
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'centos-64-x86_64'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] Booting VM...
[default] Waiting for machine to boot. This may take a few minutes...
[default] Machine booted and ready!
[default] Mounting shared folders...
[default] -- /vagrant
$ vagrant ssh
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$
{% endcodeblock %}

Great it works, we have a basic CentOS 6.4 Minimal install working, this is exactly like the box we created with veewee. Not too interesting just yet. We can add other customizations, like increase the memory size by adding this to the `Vagrantfile`

{% codeblock lang:ruby %}
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end
{% endcodeblock %}

You will also notice, that `/vagrant` folder is a shared folder with your host machine, the root of the directory where the `Vagrantfile` is stored.

{% codeblock lang:bash %}
[vagrant@localhost ~]$ ls /vagrant/
Vagrantfile
[vagrant@localhost ~]$ touch /vagrant/abd.txt
[vagrant@localhost ~]$ ls /vagrant/
abd.txt  Vagrantfile
[vagrant@localhost ~]$ exit
logout
Connection to 127.0.0.1 closed.
$ ls
Vagrantfile abd.txt
{% endcodeblock %}

### Creating the playbook

Ansible has playbooks, which basically describe all the steps ansible needs to execute to get your system to the required state. All the steps are described in a yml file, with specific keywords for each task. You can read more at: http://www.ansibleworks.com/docs/playbooks.html

Before we start with our oracle playbook, we need to create a few directories.

{% codeblock lang:bash %}
$ mkdir provisioning
$ touch provisioning/oracle-xe.yml
$ mkdir oracle
$ touch oracle/xe.rsp
{% endcodeblock %}

Before installing oracle, a few base packages are needed. Lets ensure our playbook caters for these:

{% codeblock oracle-xe.yml %}
---
- hosts: all
  sudo: yes
  tasks:
    - name: ensure packages required are installed
      yum: pkg=$item state=latest
      with_items:
        - libaio
        - bc
        - flex
        - unzip
{% endcodeblock %}

* `hosts:` specifies which hosts to run the playbook on 
* Then we tell ansible to use `sudo` to run the commands
* We then have the list of tasks we need to run
* The `yum` command can be used to install packages, specifying them with `pkg=`
* `with_items` lets you specify multiple items and ansible will loop through all of the items on the list and run yum for each package
* For the `name:` you can use anything that describes the task.

Unfortunately due to oracle licensing, you will need to accept the license agreement and download the oracle rpm from: 

http://www.oracle.com/technetwork/products/express-edition/downloads/index.html

Save the zip file to the `oracle` directory. 

The next thing we need to include in our playbook, is unzipping and installing oracle:

{% codeblock oracle-xe.yml %}
    - name: unzip oracle rpm
      command: /usr/bin/unzip -q /vagrant/oracle/oracle*.rpm.zip -d /vagrant/oracle creates=/vagrant/oracle/Disk1
    - name: install oracle
      shell: /bin/rpm -ivh /vagrant/oracle/Disk1/oracle-xe-11.2.0-1.0.x86_64.rpm creates=/u01
{% endcodeblock %}

* `creates=` defines a directory that is created when the task runs, if the box is reloaded and the directory exists, the task will be skipped.
*  You will need this or to ignore errors as the oracle installation returns an error if it is already installed.

Then we configure oracle and the vagrant user environment

{% codeblock oracle-xe.yml %}
    - name: configure oracle
      shell: /etc/init.d/oracle-xe configure responseFile=/vagrant/oracle/xe.rsp 
      ignore_errors: True
    - name: setup oracle environment
      shell: /bin/echo 'source /u01/app/oracle/product/11.2.0/xe/bin/oracle_env.sh' >> /home/vagrant/.bash_profile
{% endcodeblock %}

* `shell` does what it says on the box
*  We need to pass the oracle configure script a response file `xe.rsp` so it does not wait for input
*  Then we setup the vagrant users environment

Add the following to `oracle/xe.rsp`

{% codeblock text %}
ORACLE_HTTP_PORT=8080
ORACLE_LISTENER_PORT=1521
ORACLE_PASSWORD=manager
ORACLE_CONFIRM_PASSWORD=manager
ORACLE_DBENABLE=y
{% endcodeblock %}

We need to tell Vagrant to use the ansible playbook, by adding the following to the `Vagrantfile`. We also set the ansible verbose setting to extra, so we can see what is going on.

{% codeblock lang:ruby %}
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/oracle-xe.yml"
    ansible.verbose = "extra"
  end
{% endcodeblock %}

We can now test that oracle works from inside the VM:

{% codeblock lang:bash %}
$ vagrant up
$ vagrant ssh
Last login: Sun Nov 10 11:35:22 2013 from 10.0.2.2
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$ sqlplus system/manager@localhost

SQL*Plus: Release 11.2.0.2.0 Production on Sun Nov 10 12:37:23 2013

Copyright (c) 1982, 2011, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
{% endcodeblock %}

### Vagrant port fowarding - connect from your host

If we try an connect from the host to the VM, it wont work. To get this to work we will use vagrant port forwarding by adding the following to our Vagrantfile.

{% codeblock lang:ruby %}
  config.vm.network "forwarded_port", guest: 8080, host: 8080, auto_correct: true
  config.vm.network "forwarded_port", guest: 1521, host: 1521, auto_correct: true
{% endcodeblock %}

We also need to modify our playbook, to disable iptables and to configure oracle to accept remote connections, from outside the VM.

{% codeblock oracle-xe.yml %}
    - name: stop ip tables
      shell: service iptables stop
    - name: set oracle listener
      sudo: False
      shell: ORACLE_HOME=/u01/app/oracle/product/11.2.0/xe /u01/app/oracle/product/11.2.0/xe/bin/sqlplus 
        system/manager@localhost < /vagrant/oracle/set_listener.sql
{% endcodeblock %}

Also create a new file: `oracle/set_listener.sql`

{% codeblock set_listener.sql %}
EXEC DBMS_XDB.SETLISTENERLOCALACCESS(FALSE);
quit;
/
{% endcodeblock %}

Lets see if we can connect from our host to the VM:

{% codeblock lang:bash %}
$ vagrant provision
$ sqlplus system/manager@localhost

SQL*Plus: Release 11.2.0.3.0 Production on Sun Nov 10 15:19:50 2013

Copyright (c) 1982, 2012, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> exit
$ curl -v http://localhost:8080/apex/f?p=4950:1
{% endcodeblock %}

Great, it works. This may seem like an awful amount of work just to setup a centos vm with oracle, though remember, Once you have done this once, no one else needs to follow the same steps again. All they need to do is issue a `vagrant up` from your source code repo. 

You can view the full source for this tutorial at: https://github.com/ismaild/vagrant-centos-oracle

The full `Vagrantfile` and `oracle-xe.yml` are below.

### Vagrantfile

{% codeblock lang:ruby %}
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos-64-x86_64"

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/oracle-xe.yml"
    ansible.verbose = "extra"
  end

  config.vm.network "forwarded_port", guest: 8080, host: 8080, auto_correct: true
  config.vm.network "forwarded_port", guest: 1521, host: 1521, auto_correct: true
end
{% endcodeblock %}


### Ansible Oracle Playbook

{% codeblock oracle-xe.yml %}
---
- hosts: all
  sudo: yes
  tasks:
    - name: ensure packages required are installed
      yum: pkg=$item state=latest
      with_items:
        - libaio
        - bc
        - flex
        - unzip
    - name: unzip oracle rpm
      command: /usr/bin/unzip -q /vagrant/oracle/oracle*.rpm.zip -d /vagrant/oracle creates=/vagrant/oracle/Disk1
    - name: install oracle
      shell: /bin/rpm -ivh /vagrant/oracle/Disk1/oracle-xe-11.2.0-1.0.x86_64.rpm creates=/u01
    - name: configure oracle
      shell: /etc/init.d/oracle-xe configure responseFile=/vagrant/oracle/xe.rsp
      ignore_errors: True
    - name: setup oracle environment
      shell: /bin/echo 'source /u01/app/oracle/product/11.2.0/xe/bin/oracle_env.sh' >> /home/vagrant/.bash_profile
    - name: stop ip tables
      shell: service iptables stop
    - name: set oracle listener
      shell: ORACLE_HOME=/u01/app/oracle/product/11.2.0/xe /u01/app/oracle/product/11.2.0/xe/bin/sqlplus
        system/manager@localhost < /vagrant/oracle/set_listener.sql
{% endcodeblock %}


