---
layout: post
title: Speed up your Oracle Database provisioning with Docker and Ansible
tagline: Using Docker and Ansible to provide Oracle Database instances into Containers
categories: devops
tags: [oracle database, ansible, docker, devops]
author: jeqo
---
Warming up before [AMIS 25th Conference](http://www.amis-conference.com/Program)
event where I will be presenting with my friend
and colleague Arturo Viveros (@gugalnikov)
[about Oracle SOA Suite provisioning](http://www.amis-conference.com/Session-Catalog#session1168),
I want to share some practices that help us to provide Oracle Database instances
between developers and improve our productivity.

Since I started working with Oracle technologies, almost 7 years ago, provide
Oracle Database instances has been always a *not so easy* process. It demands
configuring the operating system with the right packages and kernel params,
then prepare user and groups, and after that install the Database engine.
Once you have your engine installed, you can start creating database instances.

Now that you have a running Database instance, you are able to
start your database development: creating schemas, running SQL scripts,
and connect your applications to store data.

So, should I repeat this process on each developer workstation?

There are several options to improve this process: share one database between
all developers, creating and share a VM image between developers, or
actually use an automation tool like Vagrant and Packer to create images,
or just provide the installation automated a tool like Puppet/Chef.

I will share how we are doing it now in a current project using Docker
as Container platform, and Ansible as our automation tool.

Let's get started...

# Defining the process

After several months working with automation tools, I understand a key principle
about automate provisioning: **"divide and conquer"**.

Because provision some type of systems is not always a simple process as:
*download, unzip and run*, in some cases you need to define your
provisioning process in steps, so you can define this steps as checkpoints,
and don't get bored re-running process from scratch. I mean: from configuring
your OS for running Oracle Database to actually run your SQL scripts is a
long way right? Even if you automate it or do it manually.

In this case, as I explained before, we have 3 major steps:

- Prepare your OS and install database software
- Create database instance
- Run scripts

And each step will become a checkpoint, i.e. a Docker image.

I won't go into every detail about installing Oracle Database, because there
are plenty information about this. At Sysco, we have developed several
Ansible roles to automate installation and configuration of Oracle software:
http://github.com/sysco-middleware. Therefore, I will cover only how do we
separate this process into steps using Docker and Ansible.


> If you need to check more about how Docker integrates with Ansible,
> take a lot to my previous post:
> http://jeqo.github.io/blog/devops/ansible-agentless-provisioning/


# Step 1: Install Oracle Database

Go to the first repository called: docker-image-oracle-database
http://github.com/sysco-middleware/docker-image-oracle-database

Here is how we create an image with Oracle Database 11g (or 12c) installed
using Ansible:

{% highlight yaml %}
- hosts: 127.0.0.1 # >>> (1)
  connection: local
  vars_files:
    - vars/main.yml
  tasks:
    - name: create container
      docker:
        name: tmp-oracle-database
        image: "{{ base_image }}"
        command: sleep infinity
        volumes:
          - "/installers/oracle/db/11.2/database/11.2.0.4:/srv/files"
        state: started

    - add_host:
        name: tmp-oracle-database
        groups: docker
        ansible_connection: docker

- hosts: tmp-oracle-database # >>> (2)
  connection: docker
  roles:
    - role: sysco-middleware.oracle-database
      oracle_database_version: 11g
      oracle_database_edition: SE
      oracle_database_installer: /srv/files/database/runInstaller

- hosts: 127.0.0.1 # >>> (3)
  connection: local
  vars_files:
    - vars/main.yml
  tasks:
    - name: docker commit
      command: "docker commit tmp-oracle-database {{ image_name }}:{{ item }}"
      with_items: "{{ tags }}"

    - name: docker kill
      command: "docker kill tmp-oracle-database"

    - name: docker rm
      command: "docker rm tmp-oracle-database"
{% endhighlight %}

We have 3 main steps here:

- First we create the containers using a variable that contains the base
image: "syscomiddleware/oraclelinux:6.7" that is based in Oracle Linux official
image, with some packages installed. And then we add the running docker container
as an Ansible host.

- Second, we connect to the running docker container, and run our
Oracle Database role (https://github.com/sysco-middleware/ansible-role-oracle-database)

- Finally, after installing Oracle database software, I create a checkpoint:
commit container as an image, kill and remove running container, and
optionally (and if you have a private repository) push your image.
(check this issue about sharing
 Oracle software inside container images:
 https://github.com/oracle/docker-images/issues/97)

To run this process you just need:

{% highlight shell %}
$ ansible-playbook main.yml
{% endhighlight %}

To check that our "oracle-database" image is created successfully, just run

{% highlight shell %}
$ docker images
REPOSITORY                        TAG                    IMAGE ID            CREATED             SIZE
syscomiddleware/oracle-database   oraclelinux-11.2.0.4   15eb5554debd        2 hours ago         6.301 GB
syscomiddleware/oracle-database   latest                 ab59ccd81cba        2 hours ago         6.301 GB 
{% endhighlight %}

The main goal here is that we have a checkpoint that represents a container
with Database engine installed. This is a unique image that can be reused to
move forward.

# Step 2: Create database instance

Once we have the Oracle Database image, we won't have to reinstall it again! :)
...as far as we keep using Docker and at least until we find out how to
improve the installation process.

So, we can move forward from this point up to the next stage: create a database
instance.

Let's go and checkout this repository:
https://github.com/sysco-middleware/docker-image-oracle-database-instance

Here we follow the same approach as previous step: prepare temporal containers,
run Ansible roles and commit image.

You can check the main.yml file here: https://github.com/sysco-middleware/docker-image-oracle-database-instance/blob/master/main.yml

And as you see, we are using another Ansible role called "oracle-database-instance"
that is used to create an instance and prepare listener:

{% highlight yaml %}
- hosts: tmp-oracle-db-instance
  connection: docker
  roles:
    - role: sysco-middleware.oracle-database-instance
      oracle_database_version: 11g
      oracle_database_sid: orcl
      oracle_database_global_name: orcl
      oracle_database_template_name: General_Purpose.dbc
      oracle_database_admin_password: welcome1
      oracle_database_auto_memory_mgnt: TRUE
      oracle_database_memory_percentage: 80
      oracle_database_memory_total: 1024
      oracle_database_type: MULTIPURPOSE
      oracle_database_listener_name: LISTENER
      oracle_database_listener_port: 1521
      oracle_database_init_params: JAVA_JIT_ENABLED=FALSE
{% endhighlight %}

After running this Ansible playbook, we will have a new image called:
*oracle-database-instance* tagged with its corresponding OS and version.

But this process is a little bit different from previous case:
As you can see in this main.yml file, there is an additional step:

{% highlight yaml %}
- name: build image
  command: "docker build -t {{ image_name }} docker"
{% endhighlight %}

This is an important step, because it involves the usage of Dockerfile to
prepare a Docker image.

To give a small background about this: Docker is prepared to isolate
process and files, and by convention you should run only 1 process by
container. To define this process, you will use a Dockerfile to specify
which command should be run, and this process should persist over time, because
if it ends, your container will be stopped.

In our case, we need to start our database instance. And to do this we will use a
Dockerfile. But, as you know, we don't have a out-of-the-box script that starts
the instance and keep this process alive and printing logging messages.
But we can create something similar:

{% highlight shell %}
LISTENERS_ORA=$ORACLE_HOME/network/admin/listener.ora

cp "${LISTENERS_ORA}.tmpl" "$LISTENERS_ORA" &&
sed -i "s/%hostname%/$HOSTNAME/g" "${LISTENERS_ORA}" &&
sed -i "s/%port%/1521/g" "${LISTENERS_ORA}" &&
bin/lsnrctl start &&
bin/dbstart $ORACLE_HOME
{% endhighlight %}


> Thanks to GitHub user "wnameless" that share how to do this in its Docke Image
> for Oracle XE: https://github.com/wnameless/docker-oracle-xe-11g


Here, we not only start and read a log file, but update our listener file. Why?
Because, also by convention, every time a container starts, container's hostname
gets updated.
So, to keep our database instance consistent, we have to update our listener.ora accordingly.

Here is our Dockerfile:

{% highlight go %}
FROM tmp-oracle-db-instance

MAINTAINER Jorge Quilcate <jorge.quilcate@sysco.no>

USER oracle

ENV ORACLE_HOME /home/oracle/product/oracle_home
ENV ORACLE_SID orcl

WORKDIR $ORACLE_HOME

ADD listener.ora.tmpl network/admin/listener.ora.tmpl
ADD startup.sh .

CMD sh startup.sh && tail -f startup.log
{% endhighlight %}

As you can see, I start the process and tail the startup log file to keep our
container running.

And each time we want to run this image, this process will be executed, unless
you override it in your *"docker run"* command.

You can test this image by running:

{% highlight shell %}
$ docker run -it syscomiddleware/oracle-database-instance:oraclelinux-11.2.0.4 
LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 25-MAY-2016 17:00:37

Copyright (c) 1991, 2013, Oracle.  All rights reserved.

Starting /home/oracle/product/oracle_home/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 11.2.0.4.0 - Production
System parameter file is /home/oracle/product/oracle_home/network/admin/listener.ora
Log messages written to /home/oracle/product/diag/tnslsnr/02b9ae1e3ab1/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=02b9ae1e3ab1)(PORT=1521)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 11.2.0.4.0 - Production
Start Date                25-MAY-2016 17:00:42
Uptime                    0 days 0 hr. 0 min. 3 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /home/oracle/product/oracle_home/network/admin/listener.ora
Listener Log File         /home/oracle/product/diag/tnslsnr/02b9ae1e3ab1/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=02b9ae1e3ab1)(PORT=1521)))
The listener supports no services
The command completed successfully
Processing Database instance "orcl": log file /home/oracle/product/oracle_home/startup.log
Total System Global Area  801701888 bytes
Fixed Size		    2257520 bytes
Variable Size		  276827536 bytes
Database Buffers	  515899392 bytes
Redo Buffers		    6717440 bytes
Database mounted.
Database opened.
SQL> Disconnected from Oracle Database 11g Release 11.2.0.4.0 - 64bit Production

bin/dbstart: Database instance "orcl" warm started.

{% endhighlight %}

From other terminal you can check container name and inspect for its IP address:
{% highlight shell %}
$ docker ps
CONTAINER ID        IMAGE                                      COMMAND                  CREATED              STATUS              PORTS               NAMES
21713e2f2c59        syscomiddleware/oracle-database-instance   "/bin/sh -c 'sh start"   About a minute ago   Up About a minute                       adoring_yalow
{% endhighlight %}

Let's take a minute to understand the process: We will have images by steps
from our provisioning process, and each step can be tagged by version and OS
(and any other relevant information). This will create a group of images that
will be reusable, and if we have issues, we can identify and solve specific
tasks, instead of re-run everything from scratch.

As learning any other technology, this will take some time at the beginning
of the process, but as we understand and collaborate to improve this images,
it will worth our effort.

First, you can see that a container name is assigned randomly: "adoring_yalow"

{% highlight shell %}
$ docker inspect adoring_yalow
...
"Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "c72c01d764b3aa7f30ffa220ed91a15aa1bb2f7c3396008601cc0137512612cb",
                    "EndpointID": "ea1e3277916bcebaa6fbba42ab6cdea6ebd597a2ce20c274b415f4ad89f05bee",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02"
                }
            }
...
{% endhighlight %}

And test to connect from your favorite IDE using a JDBC URL like:

jdbc:oracle:thin:@172.17.0.2:1521:orcl



# Step 3: Run SQL scripts

This is a more "custom" step as you can use now use this images for different
purposes (e.g: create a schema, run SQL scripts, add more configurations, or
if you are working with Fusion Middleware products, you can create RCU schemas).

Depending on your use-case it will be easier, or more efective, to use
Dockerfile than Ansible playbooks.

In this case, I will show you how to create an schema to start your Java
application development, for instance.

I will recommend you to use Docker Compose (https://docs.docker.com/compose/)
just to simplify the execution of Docker command, and link containers together.

I have created a repository (https://github.com/jeqo/post-oracle-database-docker)
to host this sample.

You can check that there is a directory called "sample" that will be assumed as
the *project name* by Docker Compose.

Inside is a file called "docker-compose.yml" that defines *container services*:

{% highlight yaml %}
version: '2'
services:
  db:
    build: db
    ports:
      - "10521:1521"
{% endhighlight %}

In this case, it defines a service (a container), that will be built from
a Dockerfile inside a "db" directory, and will forward its 1521 port
to your hosts port 10521, so you can use it from a local application.

This Dockerfile contains instructions to create an schema called "test".

{% highlight go %}
FROM syscomiddleware/oracle-database-instance:oraclelinux-11.2.0.4

MAINTAINER Jorge Quilcate <jorge.quilcate@sysco.no>

ADD create-schema.sql .

RUN sh startup.sh && \
    while ! grep "bin/dbstart: Database instance \"orcl\" warm started." startup.log; do sleep 10; done && \
    echo exit | bin/sqlplus system/welcome1 @create-schema.sql

CMD sh startup.sh && \
    tail -f startup.log
{% endhighlight %}

Then, just execute "docker-compose up -d" and this container will be built and
started.

To check its execution, run "docker-compose logs -f" and that's it.

You can customize your Docker Compose file, or just start building more layers on
top of the database instance image.

At AMIS25, we will show how to use this instance to build a SOA Suite database
and then provide customs SOA Suite Domains, but also show different experiences
with different "DevOps" technologies. Hope to see you there!

> Originally posted in [jeqo's blog](http://jeqo.github.io/blog/devops/speed-up-oracle-database-provisioning-ansible-docker/)
