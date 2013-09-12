---
layout: post
title: "Setup Oracle instant client and ruby oci8 gem on Mac"
date: 2013-09-12 22:17
comments: true
categories: [oracle, mac, ruby, oci]
keywords: "oracle, mac"
description: "Setup Oracle instant client on Mac OsX and ruby oci8 gem"
---
Recently i have had to get my dev environment setup to connect to oracle for a project.

Getting the client setup can sometimes be a real pain with a Mac. Oracle does provide client libraries but they are painfully slow in updating them and fixing bugs. There was an [outstanding segmentation fault bug for over 2 years](https://forums.oracle.com/thread/2320019?start=0&tstart=0). 

Here are the steps to get it setup on Mac, and it would probably work for ubuntu etc, just have not tested it.

First grab the 64 bit client from: 

http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html

You need:

- instantclient-basic-macos.x64-11.2.0.3.0.zip
- instantclient-sqlplus-macos.x64-11.2.0.3.0.zip
- instantclient-sdk-macos.x64-11.2.0.3.0.zip

You can also grab the lite version, if you do not need any translations.

Unzip the files:

{% codeblock lang:bash %}
cd ~/Downloads
#basic
unzip -qq instantclient-basic-macos.x64-11.2.0.3.0.zip

#lite
unzip -qq instantclient-basiclite-macos.x64-11.2.0.3.0.zip

unzip -qq instantclient-sqlplus-macos.x64-11.2.0.3.0.zip 

unzip -qq instantclient-sdk-macos.x64-11.2.0.3.0.zip 
{% endcodeblock %}

## Move files

{% codeblock lang:bash %}
cd instantclient_11_2
mkdir -p /usr/local/oracle/product/instantclient_64/11.2.0.3.0/bin
mkdir -p /usr/local/oracle/product/instantclient_64/11.2.0.3.0/lib
mkdir -p /usr/local/oracle/product/instantclient_64/11.2.0.3.0/jdbc/lib
mkdir -p /usr/local/oracle/product/instantclient_64/11.2.0.3.0/rdbms/jlib
mkdir -p /usr/local/oracle/product/instantclient_64/11.2.0.3.0/sqlplus/admin
{% endcodeblock %}

{% codeblock lang:bash %}
mv ojdbc* /usr/local/oracle/product/instantclient_64/11.2.0.3.0/jdbc/lib/
mv x*.jar /usr/local/oracle/product/instantclient_64/11.2.0.3.0/rdbms/jlib/

# rename glogin.sql to login.sql
mv glogin.sql /usr/local/oracle/product/instantclient_64/11.2.0.3.0/sqlplus/admin/login.sql

# Move lib & sdk
mv *dylib* /usr/local/oracle/product/instantclient_64/11.2.0.3.0/lib/
mv sdk /usr/local/oracle/product/instantclient_64/11.2.0.3.0/lib/sdk

mv *README /usr/local/oracle/product/instantclient_64/11.2.0.3.0/
mv * /usr/local/oracle/product/instantclient_64/11.2.0.3.0/bin/
{% endcodeblock %}

## Setup TNS Names

{% codeblock lang:bash %}
mkdir -p /usr/local/admin/network
touch /usr/local/admin/network/tnsnames.ora
{% endcodeblock %}

Put in your tnsnames, example:

{% codeblock tnsnames.ora lang:text %}
 ORADEMO=
 (description=
   (address_list=
     (address = (protocol = TCP)(host = 127.0.0.1)(port = 1521))
   )
 (connect_data =
   (service_name=orademo)
 )
)
{% endcodeblock %}

## Setup your environment

Create a file to store your oracle client environment variables with `touch ~/.oracle_client`

Add the following to it:

{% codeblock .oracle_client %}
export ORACLE_BASE=/usr/local/oracle
export ORACLE_HOME=$ORACLE_BASE/product/instantclient_64/11.2.0.3.0
export PATH=$ORACLE_HOME/bin:$PATH
export DYLD_LIBRARY_PATH=$ORACLE_HOME/lib:$DYLD_LIBRARY_PATH
export TNS_ADMIN=$ORACLE_BASE/admin/network
export SQLPATH=$ORACLE_HOME/sqlplus/admin
{% endcodeblock %}


Then run:

{% codeblock lang:text %}
echo "source ~/.oracle_client" >> ~/.bash_profile
source ~/.bash_profile
{% endcodeblock %}

Which will add the environment variables to your `.bash_profile`, You can also find this in my [dotfiles](https://github.com/ismaild/dotfiles)

## Test Sql*Plus works

{% codeblock lang:bash %}
sqlplus user/pass@orademo

SQL*Plus: Release 11.2.0.3.0 Production on Thu Sep 12 09:19:55 2013

Copyright (c) 1982, 2012, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.2.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> select table_name from user_tables;
{% endcodeblock %}

You could also store multiple versions of the client, with different `.oracle_client` files, with a small shell script you could switch between different versions (i.e like when oracle does not fix a bug for 2 years! )

## Install Ruby oci gem

{% codeblock lang:bash %}
cd /usr/local/oracle/product/instantclient_64/11.2.0.3.0/lib
ln -s libclntsh.dylib.11.1 libclntsh.dylib
ln -s libocci.dylib.11.1 libocci.dylib

# Make sure sdk directory is in /usr/local/oracle/product/instantclient_64/11.2.0.3.0/lib
gem install ruby-oci8
{% endcodeblock %}

Test that it works...

{% codeblock lang:ruby %}
irb
irb(main):001:0> require 'oci8
irb(main):006:0> o = OCI8.new('user','pass','127.0.0.1/orademo')
=> #<OCI8:user>
irb(main):011:0> o.exec('select * from dual') do |r| puts r.join(','); end
X
=> 1
irb(main):012:0> exit
{% endcodeblock %}

If you have issues connecting with an SID or Service Name, try using the IP.

You now have a working oracle database client, and can connect to it from ruby.