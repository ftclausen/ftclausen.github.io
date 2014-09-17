---
layout: post
title: "Docker with Test Kitchen Notes"
date: 2014-09-09 16:15
categories: general
---

Docker seems like it can be a great help to integration testing of Chef
cookbooks; a good compliment to unit testing with Chefspec. I foresee a
potential work-flow such as 

0. Develop cookbook changes
1. Rubocop and Foodcritic checks
2. Chefspec tests
3. Docker and Test Kitchen tests
4. Deploy to actual machines

I will record some of my experiences here; both for my own reminders and also to
help others in their Googling. Where config snippets are shown they are just
that, snippets. The full config is at the end of this post.

# Installing

I am running OS X so used the ever useful [boot2docker](http://boot2docker.io/) method of 
launching containers. Boot2docker is also applicable for Windows.

If you are on Linux, with a recent enough Kernel, then you can just launch containers 
directly on your host machine

# Use Caching

While this is worthy of a whole blog post unto itself I recommend installing a
simple HTTP caching proxy on work workstation such as
[Polipo](http://www.pps.univ-paris-diderot.fr/~jch/software/polipo/) and
configuration test kitchen to use it. While modifying your Docker images is
probably the best long term fix for quick bootstrapping, having cached copies of
commonly fetched files can *greatly* speed things up. 

You can tell test kitchen to use your proxy by adding the following to your
"driver_config" section (replacing the IPs with
your own)

    driver_config:
      http_proxy: http://10.111.111.1:8123
      https_proxy: http://10.111.111.1:8123

NOTE: This does not yet seem to work for Chef "remote_file" resources. More
testing required.

# Use an init system

While I would normally recommend, with Docker, to just run your app in the
container unadorned by an init system \(such as Runit or Upstart\), for testing a
Cookbook it makes sense to use an init system. I suspect that normally you'd manage docker container
lifecycle operations with Chef *externally* to the container itself but in this
case Docker is not the item under management; it is merely a means to an end.

Thus, to activate an init system, to ensure things start somewhat normally and
your service resources work as expected, set the "run_command" in the
"driver_config" section to /sbin/init. For example:

      driver_config:
        run_command: /sbin/init
  
# Set up an SSH Key

For this you can probably modify the Docker image but I decided to use a
provisioning command just to make this always happen regardless of image type.
What I did was serve up a script (only accessible from my workstation) with my 
public SSH key contained within to be placed in ~kitchen/.ssh/authorized_keys.

    #!/bin/bash
    key="THE_PUB_KEY"
    dot_ssh="/home/kitchen/.ssh"
    if [[ ! -d "$dot_ssh" ]]; then
        mkdir "$dot_ssh"
    fi
    echo "$key" >> "$dot_ssh/authorized_keys"
    chown -R kitchen:kitchen "$dot_ssh"
    chmod 700 "$dot_ssh"
    chmod 600 "$dot_ssh/authorized_keys"

Then modify your Kitchen configuration to reference this by using the
"provision\_command" statement within the "driver\_config" section. For Example

    driver_config:
        provision_command: curl -sSL  http://10.111.111.1/setup_key.sh | bash -s

This allows "kitchen login" to work without a password.

# Example Configuration

    ---
    driver_plugin: docker
    driver_config:
      http_proxy: http://10.111.111.1:8123
      https_proxy: http://10.111.111.1:8123
      socket: tcp://192.168.59.103:2375
      provision_command: curl -sSL  http://10.111.111.1/setup_key.sh | bash -s
      run_command: /sbin/init

    provisioner:
      chef_omnibus_url: http://www.getchef.com/chef/install.sh
      name: chef_zero

    platforms:
        - name: centos-6.4

    suites:
      - name: default
        run_list:
          - recipe[ntp]
        attributes:
