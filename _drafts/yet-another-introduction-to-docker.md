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

I haven't had a chance to use docker in the infrastructures I manage as a sysadmin, so I'm not going to write a lot about docker's pros and cons or use cases, as I don't feel I've enough knowledge or experience. There a lot of blog posts and articles written by much smarter and more experienced people who have used docker and other related technologies in their environments. In this post I'll just present my own experience of jumpstarting with docker.

What I believe may be interesting in this post is that it's an introduction to docker based on a simple but real world troubleshooting task from the system engineering and administration domain. If you're like me just beginning with docker, I hope that this post can get you started and also show that it can be used in small daily tasks as much as in bigger projects like CI pipelines and production software delivery systems.
