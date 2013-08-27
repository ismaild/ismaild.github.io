---
layout: post
title: "Uninstall MySQL & MacPorts, reinstall MySQL with Homebrew on Mac OSX"
date: 2013-08-27 08:58
comments: true
categories: [mysql, homebrew]
keywords: "mysql, homebrew"
description: "Uninstall mysql on mac osx, and reinstall with home brew"
---
I have recently have to set up my dev environment once again, which can be quite a bit of a pain, especially if you do not have something like [boxen](http://boxen.github.com/).

This time however i decided to avoid macports all together, one of the "benefits" of macports over homebrew is the ability to run different versions since macports will install into /opt , though its a feature i never used, and i have had a bunch of strange build incompatabilities that has taken me quite a bit of time to debug. 

Here are the steps

Uninstall macports
---------------

{% codeblock lang:bash %}
sudo port -fp uninstall --follow-dependents installed
{% endcodeblock %} 

{% codeblock lang:bash %}
sudo rm -rf /opt/local
sudo rm -rf /Applications/DarwinPorts
sudo rm -rf /Applications/MacPorts
sudo rm -rf /Library/LaunchDaemons/org.macports.*
sudo rm -rf /Library/Receipts/DarwinPorts*.pkg
sudo rm -rf /Library/Receipts/MacPorts*.pkg
sudo rm -rf /Library/StartupItems/DarwinPortsStartup
sudo rm -rf /Library/Tcl/darwinports1.0
sudo rm -rf /Library/Tcl/macports1.0
sudo rm -rf ~/.macports
{% endcodeblock %}

Uninstall MySQL
------------
1. Backup any databases you have using mysqldump
2. Stop MySQL if it is currently running, under settings > mysql

and then run:

{% codeblock lang:bash %}
sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /private/var/db/receipts/*mysql*
rm -rf ~/Library/PreferencePanes/My*
{% endcodeblock %}

You may also need to edit `/etc/hostconfig` and remove the line `MYSQLCOM=-YES-`

Install Homebrew & MySQL
------------------------

{% codeblock lang:bash %}
ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
brew install mysql
{% endcodeblock %}

Welcome to [nirvana](http://www.youtube.com/watch?v=hNVu55ZyC-Y&list=PL7AD6AA21F4F0E76E)

:)

Seriously, i have not yet had any compatibility or weird build errors with homebrew.