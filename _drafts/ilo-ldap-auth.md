---
layout: post
title: "LDAP authentication and authorization in HP iLO"
---

Central user management
=======================

I'm pretty sure that I don't have to convince anyone that central access
management is a must. Although it's possible to do it in many different ways,
LDAP is the de-facto standard at least in the Linux world.

The above statement is somewhat general, because even LDAP as a method itself
can come in many different flavors, there are different caching strategies,
sometimes it makes sense to continue using local user databases (passwd files)
on some servers, there are different implementations of the LDAP server. But in
this post, I will only consider using LDAP as your central user database, without
going into much detail how this database is used by the 
