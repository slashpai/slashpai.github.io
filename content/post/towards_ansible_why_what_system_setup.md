---
author: Pai
categories:
- infratructure as code
date: "2020-10-14"
description: Ansible Series
featured: false
series:
- Ansible
tags:
- Ansible
thumbnail: /img/pawel-czerwinski-B_GEOp9u8h8-unsplash.jpg
title: Towards Ansible -> What, Why and System Setup
---

This will be a blog series to learn Ansible automation tool from scratch. There are no pre-requisite as such other than basic linux administration skills. This post is first one in the ansible series

## What is Ansible

RedHat Ansible is an open source configuration management, application deployment and orchestration tool enabling Infrastructure as Code. It provides us enormous options to automate our Infrastructure. Unlike its alternatives Chef, Puppet it require minimum coding effort which is a great news for people without programming experience. It will help anyone to automate their tasks easily if they are familiar with YAML.

## Why Ansible

Imagine that you want to install and configure a web app on a node. Its very easy right? You just need to install the dependencies for the web app and then start your app on that node. Then you decided that you want to scale your application by having more instances. You need to repeat the same steps on those nodes. What if this is on a live environment where you need to do this configuration quickly? This is where a configuration management helps you. Once you have stored your configuration as a code, all you need to do is apply this on a new node to provision that node with required configs. Ansible does this in clean way with minimum coding and without much overhead.

Forget about Infrastructure Automation for a while, consider a personal scenario where you purchased a new laptop and you need to configure this machine from scratch. Think about the changes you've made to your laptop since first time you booted it up. Now imagine making similar changes to 10, 100 or 1000's more computers in your organization. Configuration management tools are what makes these change rollouts easier by enforcing standards.

Ansible is a push based architecture where you have to initiate the configuration required from a control node, which will be mostly your workstation which has Ansible installed. There is no need to install an agent or any other binary on remote node to use Ansible. Hence its very easy to use Ansible within a short span of time once you have a control node which can access the remote nodes using ssh.

## Setup your development environment

**Disclaimer: These are purely my recommendations and what I learnt from other developers. You can still work on Ansible skipping some of these.**

* Install [pyenv](https://github.com/pyenv/pyenv) to manage and install python versions
* Install stable version of python 3.5 or above using pyenv. Its not recommended to use system Python since in case if you did any mistake unknowingly it can leave your system in inconsistent state
* Set the newly installed version of python as default one or use it as local one from where you will be running Ansible
* Install Ansible using [pip](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-with-pip)
* Install Plugins/Extensions for Ansible, YAML in your IDE which will help you write code easily. For Visual Studio Code extension is available from Microsoft and RedHat
* Verify ansible command is working fine

```bash
 slashpai@pai  ~  (⎈ |kind-pai:default) ansible --version
 ansible 2.9.6
  config file = None
  configured module search path = ['/home/slashpai/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/slashpai/.pyenv/versions/3.8.1/lib/python3.8/site-packages/ansible
  executable location = /home/slashpai/.pyenv/versions/3.8.1/bin/ansible
  python version = 3.8.1 (default, Mar  5 2020, 18:59:28) [GCC 7.4.0]
 slashpai@pai  ~/github/myrepo/slashpai.github.io   master ●✚  (⎈ |kind-pai:default)
```

* Install [virtual Box](https://www.virtualbox.org/) and [vagrant](https://www.vagrantup.com/) (this is needed to create virtual machines in the next section)
* Install zsh shell and use [ohmyzsh](https://github.com/ohmyzsh/ohmyzsh) framework. This is optional but very useful when you are a developer since it supports a lot of plugins and flexibility over bash
* Install [bat](https://github.com/sharkdp/bat). This is again optional but quite useful tool to view config files etc
* Create a github account if not already there, if you wish to store your code over there

In next post we will see how to create test machines using vagrant and get familiarize with ansible.
