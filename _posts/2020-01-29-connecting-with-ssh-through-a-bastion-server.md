---
layout: post
published: true
title: Connecting with SSH through a bastion server
categories:
  - sys-admin
author: tristan_dietz
tags:
  - sys-admin
  - ssh
  - security
date: '2020-01-29'
image: /img/user_upload/fortress.png
---
Using SSH is today an easy way to access a remote host. There are several ways to use it, from using default context with user/password credentials to different directions.

A system architecture is well designed when adapted to the structure it belongs to. A production environment can be a simple server, or you can use a load balancer, material or physical, and a few servers in a cluster on cutting-edge technology.

As an architect, the purpose is often to build a system adapted to needs at a lower cost as possible. That's a business view. An architect has to keep in mind that his architecture must adapt to the corporate structure: you don't necessarily need to be protected, such as a banking infrastructure, if you're a young developer modding your first Minecraft server.

However, you can try to protect yourself with different manners. Today, the topic is about using a bastion server to protect from accessing your systems.

## How does it work?

This pattern aims to access Linux instances through a unique entrance door. You can have public known servers to expose services and - as an example - an SSH server listening to this only way: the bastion server.

![From: AWS Blog]({{site.baseurl}}/img/user_upload/NM_diagram_061316_a.png)

## Setting up & configuration

In the future, we admit that you use only SSH keys as a connect method. SSH keys are made for a better world, simplified governance, and other nice stuff, so think about it twice, and please, use it!

**Actual configuration:**
- bastion server: _bastion_, the entrance door
- a Linux server: _web_, hosting an Apache server

We aim to connect to a remote server, _web_, by jumping on the bastion server. We can do one of the following methods.

### Dummy connection to _bastion_

You can first connect to _bastion_:
    
    you@localhost$ ssh user@bastion

And then

    user@bastion$ ssh user@web

But there are problems with this solution. For governance and security issues, you may want to use SSH keys. This way to do present a leak of security: you'll have to leave an SSH key allowed to connect to _web_ in the user's folder on _bastion_. And possibly on multiple web servers.

Your entrance door may suffer from attacks - and no, security through obscurity isn't a perfect practice. Leaving a private key on an exposed server is a bad practice and mustn't be done.

### A better way: connection to _bastion_ by keeping agent configuration

Using the credentials present on your _localhost_ computer is also a possibility. If you have the same private key to access to _web_, you can use an SSH option to forward the authentification agent:

    you@localhost$ ssh -A user@bastion
    
    user@bastion$ ssh user@web

You just grant access on _bastion_ of your SSH key. That's better, but that's not what we want. What we do want is to connect to _web_ and only to _web_.

### The excellent way: connecting through _bastion_

If your SSH key is  in an SSH agent, you can then use:

    you@localhost$ ssh -J user@bastion user@web

With this configuration, you use _bastion_ to Jump into _web_. We just connect to _bastion_ using an SSH connection and create a TCP forwarding from _bastion_ to achieve the connection to _web_.

In this case, your key is conserved on your _localhost_, and the context of your SSH connection is preserved until the last destination of your connection.

### What about automation?

Writing a jumping command can be tedious, especially if you have multiple bastions servers.

    you@localhost$ ssh -J user@bastion1,user@bastion2,user@bastion3 user@web

It is better to define it in a separate configuration file. This configuration file (by default: `~/.ssh/config`) is an excellent way to order your path. And set relevant configurations, such as an IP or a host with `Hostname` directive, a special `IdentityFile` if, like me, you prefer using a different one for each server you connect, and the `User`, which can change from a server to another.

> Note: the host aliasing can be done here, but impacts only SSH client, such as in /etc/hosts for sharing configuration and/or other software

    Host bastion
        Hostname 115.247.193.203
        IdentityFile ~/.ssh/bastion
        User user

    Host web
        ProxyJump bastion
        Hostname 10.10.1.51
        IdentityFile ~/.ssh/web
        User user

You may notice that we can use subnet inaccessibles with a standard Internet connection but allowed when passing by a ProxyJump.

This allows you to simply connect to the web the most straightforward way as possible:

    you@localhost$ ssh web

And that's it.

## Conclusion

Security is an everyday concern and must be adapted to each situation and not over-engineered. Your policies have to fit with the size of your architecture, your destination users, and the available wherewithals.

When designing an infrastructure, think about a fact: the level security is inversely proportional to the access ease. So [KISS](https://fr.wikipedia.org/wiki/Principe_KISS), and estimate well regarding the needs of your users or your business.
