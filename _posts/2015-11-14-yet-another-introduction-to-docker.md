---
title: "Yet Another Introduction to Docker"
layout: post
category: linux
tags: linux, docker, hp, ldap, ilo, openldap, management console, hardware, hardware management
description: "Creating a lab for reverse engineering and decrypting HP iLO's LDAP requirements with Docker, docker-machine and docker-compose"
keywords: "docker, HP, iLO, LDAP, OpenLDAP, schema, docker-compose, docker-machine, Linux, sysadmin"
---

Docker for sysadmins
--------------------

I've been reading and hearing a lot about Docker but haven't had a chance to use it. Until recently, I've been using Vagrant when I needed to stand up a Linux system on my laptop for playing, testing, troubleshooting etc. I especially liked Vagrant for how easy it is to make files from your local workspace available inside the Linux system.

Building a lab with Docker
==========================

We're using quite a few HP servers in our infrastructure and recently I was trying to setup LDAP authentication for HP's iLO interface. Turns out that HP didn't allow for a lot of configuration to adapt to an existing LDAP schema, so I wanted to reverse engineer what kind of schema is required by iLO.

In order to do this, I decided to build a test LDAP server and point an HP server at it. So why not use docker to run this server on my laptop.

I also wanted to take a tcpdump traffic capture in case LDAP logs alone do not provide enough information. HP requires encrypted LDAP over SSL, so I needed a kind of a man-in-the middle SSL decryption tunnel, so that I could tap my tcpdump at the unencrypted end of the tunnel.

<pre>
  -------------               -------------              ------------
  |    iLO    | <=== SSL ===> |  stunnel  | <----o-----> | OpenLDAP |
  -------------               -------------      |       ------------
                                                 |
                                            ------------
                                            |  tcpdump |
                                            ------------
</pre>

Chaining containers together
============================

I could have created a single container for all the components, but the Docker way is to run a single process per container, as opposed to running a full operating system in a container. There are some challenges introduced with this approach, and some [argue][baseimage-docker] that it's better to run at least a minimal infrastructure like *sshd* and *rsyslog* inside your app's container, but most of the community seems to agree that it's better to separate the responsibilities and build a separate container for each responsibility.

So I've created 3 containers. One for each of the components of my lab.

#### Enter docker-compose

Thanks to [docker-compose][docker-compose], it's very easy to create and orchestrate multi-container environments. It's similar in this regard to Vagrant which also let's you create multi VM projects. With docker, however, it's much more lightweight because each container is basically just a Linux process so starting containers comes without the overhead of booting an OS and also docker images do not necessarily have to contain a filesystem with the whole OS, they just need to contain binaries and libraries required by the process which is being ran inside the container.

OpenLDAP server behind stunnel
==============================

I created the following project

<pre>
.
├── README.md
├── captures
│   └── placeholer
├── docker-compose.yml
├── docker-openldap-master
│   ├── Dockerfile
│   ├── README.md
│   └── files
│       ├── 1refint.ldif
│       ├── allow_bind_v2.ldiff
│       ├── back.ldif
│       ├── front.ldif
│       ├── memberof_load_configure.ldiff
│       ├── more.ldif
│       ├── sssvlv_config.ldif
│       └── sssvlv_load.ldif
├── stunnel
│   ├── Dockerfile
│   ├── ldap.conf
│   └── my_init.sh
└── tcpdump
    └── Dockerfile
</pre>

Full source code available at [https://github.com/bodgix/hp_ldap][hp_ldap]

docker-openldap-master is a project I found on [larrycai's github][docker-openldap], which is a containerized OpenLDAP server, with an init script which substitutes env variables into an slapd config template. stunnel and tcpdump are docker images I created using the *centos:6* official image as a base.

* stunnel
<script src="https://gist.github.com/bodgix/fe82203d3cdc16a4d4c3.js"></script>

* tcpdump
<script src="https://gist.github.com/bodgix/f1c85204fe76fab865a4.js"></script>

#### Linking containers and generating configs

When you start a container, it gets a random IP assigned by Docker. Linking containers together, adds an environmental variable and an */etc/hosts* entry inside the linked containers which allows one container to reach another.

Because the information is only available at runtime, it's quite common that a config file for the process running inside the container has to be generated at runtime. For more complicated cases, it may be a good idea to use some kind of a template system like ERB or jinja, but in my case the config is so simple that I'm just running *sed* from *my_init.sh* to generate the configs and start the stunnel process.

* *my_init.sh*
<script src="https://gist.github.com/bodgix/62eece1082046c5923c3.js"></script>

* *ldap.conf* template
<script src="https://gist.github.com/bodgix/34a205f3c21e01b7008c.js"></script>

#### Docker volumes

Except for tcpdump, my containers are stateless: I don't need to preserve anything across restarts. tcpdump, on the other hand, needs to save the packet captures for analysis later.

For that to work, I'm mounting a directory on the host: *./captures*, inside the tcpdump container. This is done by this lines in the *docker-compose.yml* file:

{% highlight yaml %}
  volumes:
    - ./captures:/tcpdump
{% endhighlight %}

#### Namespaces

Namespaces are core to Docker. It's a Linux kernel feature which allow the containerized processes to use the host's kernel and yet be separated from the host and other containers. Most of the time docker manages namespaces for you. However, in my case, I needed two of my containers to share the same network namespace. I needed *tcpdump* running inside it's own container to sniff the traffic going to the OpenLDAP container.

That's how easy it is with composer:

{% highlight yaml %}
net: "container:openldap"
{% endhighlight %}

3, 2, 1 ... GO!
===============

Now that we know all the bits and pieces, we're ready to actually run the set of containers.

First, we need to build the images using the Dockerfiles from our project

`docker-compose build`

When we have the containers, we can just start them up with:

`docker-compose up`

We'll see something similar to this:

<pre>
{14:35}[1.9.3]~/workspace/hp_ldap:develop ✓ ➭ docker-compose up
Recreating openldap
Recreating tcpdump
Recreating hpldap_stunnel_1
Attaching to openldap, tcpdump, hpldap_stunnel_1
openldap  | 564746c3 @(#) $OpenLDAP: slapd  (Ubuntu) (Sep 15 2015 18:19:13) $
openldap  | 	buildd@lgw01-53:/build/openldap-2QUgtL/openldap-2.4.31/debian/build/servers/slapd
tcpdump   | tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
stunnel_1 | LDAP_ALIAS is defined. I'm using
stunnel_1 | OPENLDAP_PORT env variable.
stunnel_1 | Using config:
stunnel_1 | foreground = yes
stunnel_1 | sslVersion = all
stunnel_1 | options = NO_SSLv2
stunnel_1 |
stunnel_1 | [ldaps]
stunnel_1 | accept = 636
stunnel_1 | cert = /etc/stunnel/cert.pem
stunnel_1 | key = /etc/stunnel/key.pem
stunnel_1 | connect = 172.17.0.2:389
stunnel_1 |
stunnel_1 | Running: /usr/bin/stunnel /tmp/tmp.F4IcBg9FiK
openldap  | 564746c4 hdb_db_open: database "dc=nodomain": unclean shutdown detected; attempting recovery.
stunnel_1 | 2015.11.14 14:35:48 LOG4[17:140141766498240]: Wrong permissions on /etc/stunnel/key.pem
stunnel_1 | 2015.11.14 14:35:48 LOG5[17:140141766498240]: stunnel is in FIPS mode
stunnel_1 | 2015.11.14 14:35:48 LOG5[17:140141766498240]: stunnel 4.29 on x86_64-redhat-linux-gnu with OpenSSL 1.0.1e-fips 11 Feb 2013
stunnel_1 | 2015.11.14 14:35:48 LOG5[17:140141766498240]: Threading:PTHREAD SSL:ENGINE,FIPS Sockets:POLL,IPv6 Auth:LIBWRAP
stunnel_1 | 2015.11.14 14:35:48 LOG5[17:140141766498240]: 512000 clients allowed
openldap  | 564746c4 hdb_db_open: database "dc=openstack,dc=org": unclean shutdown detected; attempting recovery.
openldap  | 564746c4 slapd starting
openldap  | 564746c4 bdb(dc=nodomain): BDB0060 PANIC: fatal region error detected; run recovery
</pre>

Notice how stdout of the containerized processes is shown on the terminal. We can see our containers running:

<pre>
{14:37}[1.9.3]~ ➭ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                  NAMES
954a7cea2e88        hpldap_stunnel      "/sbin/my_init.sh"       About a minute ago   Up About a minute   0.0.0.0:636->636/tcp   hpldap_stunnel_1
e9ede0e2b7cf        hpldap_tcpdump      "/bin/sh -c '/usr/sbi"   About a minute ago   Up About a minute                          tcpdump
fe36c01e427f        hpldap_openldap     "/bin/sh -c 'slapd -h"   About a minute ago   Up About a minute   0.0.0.0:389->389/tcp   openldap
</pre>

We can now run an ldapsearch against our chain of containers:

<pre>
{14:40}[1.9.3]~ ➭ LDAPTLS_REQCERT=never ldapsearch -x -H ldaps://$(docker-machine ip default):636
</pre>

Setting LDAPTLS_REQCERT=never is required so that ldapsearch will accept stunnel's self-signed certificate.

This is what we see in another console where docker-compose is running:

<pre>
stunnel_1 | 2015.11.14 14:40:49 LOG5[17:140141766493952]: ldaps accepted connection from 192.168.99.1:59350
stunnel_1 | 2015.11.14 14:40:49 LOG5[17:140141766493952]: connect_blocking: connected 172.17.0.2:389
stunnel_1 | 2015.11.14 14:40:49 LOG5[17:140141766493952]: ldaps connected remote server from 172.17.0.3:45885
openldap  | 564747f1 conn=1000 fd=22 ACCEPT from IP=172.17.0.3:45885 (IP=0.0.0.0:389)
openldap  | 564747f1 conn=1000 op=0 BIND dn="" method=128
openldap  | 564747f1 conn=1000 op=0 RESULT tag=97 err=0 text=
openldap  | 564747f1 conn=1000 op=1 SRCH base="" scope=2 deref=0 filter="(objectClass=*)"
openldap  | 564747f1 conn=1000 op=1 SEARCH RESULT tag=101 err=32 nentries=0 text=
openldap  | 564747f1 conn=1000 op=2 UNBIND
openldap  | 564747f1 conn=1000 fd=22 closed
stunnel_1 | 2015.11.14 14:40:49 LOG5[17:140141766493952]: Connection closed: 28 bytes sent to SSL, 60 bytes sent to socket
</pre>


There's also a new pcap file in the *captures* directory:

<pre>
{14:45}[1.9.3]~/workspace/hp_ldap:develop ✓ ➭ ls -l captures
total 8
-rw-r--r--  1 bogdan.katynski  WORKDAYINTERNAL\Domain Users  1194 Nov 14 14:45 ldap_2015-11-14_14:35:47.pcap
-rw-r--r--  1 bogdan.katynski  WORKDAYINTERNAL\Domain Users     0 Nov 11 10:42 placeholer
</pre>

And this is how it looks like when we open it in wireshark:

![wireshark](/assets/img/yet_another_introduction_to_docker/tcpdump.png)

We can see that although we used an *ldaps://* URL to connect to the LDAP server, tcpdump was capturing at the unencrypted end of the tunnel and the traffic is visible in clean text!

[Q.E.D.][qed] thank you!

[docker-machine]:   https://docs.docker.com/machine/              "Docker machine"
[baseimage-docker]: https://news.ycombinator.com/item?id=7258009  "baseimage-docker"
[docker-compose]:   https://docs.docker.com/compose/              "docker-compose"
[hp_ldap]:          https://github.com/bodgix/hp_ldap             "hp_ldap github repo"
[docker-openldap]:  https://github.com/larrycai/docker-openldap   "larrycai/docker-openldap"
[qed]:              https://en.wikipedia.org/wiki/Q.E.D.          "which is what had to be proven"
