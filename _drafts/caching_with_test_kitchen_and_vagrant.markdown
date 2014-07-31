---
layout: post
title: "Caching with Test Kitchen and Vagrant on your Workstation"
date: 2014-07-30 11:07
categories: general
---

# Introduction

In this post I would like to describe how to use a combination of the following

- Polipo caching web proxy
- The vagrant-cachier plugin
- Kitchen configuration 
- Tying it all together

Note that I am using this in a plain .kitchen.yml within my cookbook - I have not
used .kitchen.vagrant.local.yml to override these things as the
[documentation](http://www.rubydoc.info/github/opscode/test-kitchen/#Overriding__kitchen_yaml_with__kitchen__driver__local_yml)
was a bit sparse. I want to wra all that I did to get it working in a single 
guide as much for me 6 months from now as others finding this in their
Googling.

None of this would be possible without this excellent [Gist](https://gist.github.com/fnichol/7551540) by [fnichol](https://gist.github.com/fnichol). This [issue report](https://github.com/test-kitchen/kitchen-vagrant/issues/90) was also useful.

# Assumptions

I've made the following assumptions which may or may not impact following the
examples

- This guide is written for OS X - if you are on something else then this stuff
should still work. Just adapt for your local set up.
- You are using Vagrant with VirtualBox 

# Step N - Create VirtualBox host only network

# Step N - Polipo

## Install

Using [Homebrew](http://brew.sh/) we can install Polipo 

```
$ sudo brew install polipo
```

## Configure

What I did was give Polipo a configuration file in ``` ~/etc/polipo.conf``` for
my user. You can put it anywhere but just take note of where you placed it. 

It initially only binds 
