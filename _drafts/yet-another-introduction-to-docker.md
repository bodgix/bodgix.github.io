---
title: "Yet Another Introduction to Docker"
layout: post
category: linux
tags: linux,docker
---
Why Docker
----------

Containers are the next big thing now. Take a look around and you'll see them almost everywhere in the tech landscape. Docker, on the other hand, is by far the most popular set of tools augmenting the pure containers with features that make them so easy to use and powerful.

I've been following news and reading articles on docker. I've also been thinking a lot about the pain points and problems I've come across in various places I've worked and I really think that docker could solve a lot of them. I especially like how it bundles the application and it's dependencies together and allows to run applications side by side even if they have conflicting dependencies.

Docker for sysadmins
====================

What I believe may be interesting in this post is that it's an introduction to docker based on a simple but real world troubleshooting task from the system engineering and administration domain. If you're like me just beginning with docker, I hope that this post can get you started and also show that it can be used in small daily tasks as much as in bigger projects like CI pipelines and production software delivery systems.

About this post
---------------

In the next section, I'm giving some context and describing the tools I've used in my example. I'm also explaining briefly why the tools are suitable for this task. There are links to the manual pages of the said tools included. If you're already familiar with tools like *docker-machine*, *docker-compose*, *boot2docker*, you can skip the next section completely and head straight to the [My task][mytask] section.

Getting started
---------------

There is a good getting started guide at the docker.com website: [link][docker-starting] with instructions on how to install docker on your operating system. Docker can currently run on Linux with a new enough kernel only because it's using features like namespaces and cgroups which only Linux provides. Because of that, installing docker on another OS always requires some kind of virtualization system which runs Linux ([boot2docker][boot2docker] at this moment is used.) but you don't need to install the virtualization software and the virtual machine because docker provides packages which bundle together docker and the dependencies for a given OS. All the examples in this article were ran and work on Mac OS. There shouldn't be many differences for other OSes.

### docker-machine

[docker-machine][docker-machine] is a new way to get started on MaxOS. It starts a virtual machine on VirtualBox or even Amazon EC2 and is using a local docker client which talks to the docker daemon on the virtual machine. A really useful feature of docker-machine is that it allows for running multiple machines to simulate a multi-host environment.

### docker-compose

Even though it is possible to start a full OS inside a container with init, sshd, syslogd, etc. and some people and companies actually recommend running some additional daemons alongside your app in the container (see [the baseimage-docker hackernews thread][baseimage-docker]), most of the times you will want to run a single app inside a container. For this reason I decided to use a few containers in my troubleshooting lab and I'm using [docker-compose][docker-compose] to orchestrate the containers and allow to describe my entire troubleshooting in code so that I can share it easily and track changes in git or other version control system. I can also build it in a repeatable way without writing long docs that no one has time to read like this post.

My task <a href="#mytask">&nbsp;</a>
-------------------------------------

[docker-starting]:  http://docs.docker.com/mac/started/           "Getting started with docker"
[boot2docker]:      http://boot2docker.io/                        "Boot2docker website"
[docker-machine]:   https://docs.docker.com/machine/              "Docker machine"
[baseimage-docker]: https://news.ycombinator.com/item?id=7258009  "baseimage-docker"
[docker-compose]:   https://docs.docker.com/compose/              "docker-compose"
[mytask]:           {{ post.url + '#my_task' }}                   "My task"
