---
title: "The Jenkins Environment Variable Layer Cake"
date: 2025-08-14 14:44
classes: wide
use_math: true
categories: jenkins
---

Jenkins has a wide range of ways that the omnipresent `env` map in pipeline code (e.g. Jenkinsfile) is filled. Most of
this applies to other job types too such as freestyle. I was recently confused about how these variables travel from
each layer so drew a picture to help clarify my thoughts and sharing here just in case someone finds it useful.

This picture is not meant to be exhaustive, there are probably lots of other ways:

![jenkins-env](/images/jenkins-env.jpg)
