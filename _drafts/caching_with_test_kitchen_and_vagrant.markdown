---
layout: post
title: "Chef integration testing on the go"
date: 2014-07-30 11:07
categories: general
---

# Introduction

In this post I would like to describe how to, potentially, allow you to do
integration test with Test Kitchen, Vagrant and VirtualBox in a manner that will
require minimal network traffic. For example if you're on the road and tethering
with your phone. The techniques here can also be used back at base to
significantly speed up integration test runs thereby reducing coffee
consumption.

What we'll cover in this post is

- Set up a host-only network to provide something stable for Polipo to bind to
- Configure the Polipo caching web proxy
- Install the vagrant-cachier plugin for caching remote\_file calls as these don't
  seem to respect the http\_proxy environment variable
- Kitchen configuration 
- Tie it all together

Note that I am using this in a plain .kitchen.yml within my cookbook - I have not
used .kitchen.vagrant.local.yml to override these things as the
[documentation](http://www.rubydoc.info/github/opscode/test-kitchen/#Overriding__kitchen_yaml_with__kitchen__driver__local_yml)
was a bit sparse.

Finally I want to use [Docker](https://www.docker.com/) but our production
environments currently run regular VMs so for completeness I will still need to
test on a complete OS running in a VM.

None of this would be possible without this excellent [Gist](https://gist.github.com/fnichol/7551540) by [fnichol](https://gist.github.com/fnichol). This [issue report](https://github.com/test-kitchen/kitchen-vagrant/issues/90) was also useful.

# Assumptions

I've made the following assumptions which may or may not impact following the
examples

- This guide is written for OS X - if you are on something else then this stuff
should still work. Just adapt for your local set up.
- You are using Vagrant with VirtualBox; this should be set up and working.

# Step N - Create VirtualBox host only network

We need to create a host only virtual network for the following reasons

- Local interface (ethernet or Wireless)  IP address can vary depending 
  on where you are which means the VMs don't have a consistent proxy IP to 
  refer to. Using this gives us that IP we can use in our Kitchen config.
- When we are totally disconnected then this virtual network will remain usable
  for both the host (your workstation) and the VMs. 
- Sometimes when transitioning between locations the NAT interfaces, in my
  experience, don't transition with and need a restart.
 
What you need to do is the following

1. Open the virtual box GUI
2. Go to the "global" preferences in the main menu (not one associated with a
   VM)
3. Click on "Network" -> "Host-only Networks". Then click on the "+" network
   card button

![network prefs]({{site.url}}/assets/vbox_net_prefs.png)

4. That will bring up a details box where you can fill in your subnet (I used
   10.111.111.0/24) with virtual interface address 10.111.111.1.

![network details]({{site.url}}/assets/vbox_net_details.png)

5. We won't be using this network for assigning IPs to VMs so you can disable
   DHCP in the next tab
6. Click OK -> OK to save it all.

# Step N - Polipo

Now that we have a persistent IP address for Polipo to bind to we can get it
configured.

## Install

Using [Homebrew](http://brew.sh/) we can install Polipo 

```
sudo brew install polipo
```
## Configure

What I did was give Polipo a configuration file in ``` ~/etc/polipo.conf``` for
my user. You can put it anywhere but just take note of where you placed it. This
is the configuration I used to get started with

```
logLevel = 0xFF # Debug logging
logFile = /tmp/polipo.log
allowedClients = 127.0.0.1, 10.111.111.0/24
proxyAddress = 0.0.0.0 # Bind to all interfaces
disableIndexing=false # Allow us to see the index status
disableServersList=false # Allow us to see which servers are being cached
maxAge=100d # This will mostly be used for transient testing so keep the cached
files for a while
maxExpiresAge=100d # Ditto
objectHashTableSize=0 # Auto manage the object table size
objectHighMark=16384 # I think this means keep max 16384 items in the cache
```

## Start Manually

Start Polipo manually and see if it works, check the above log file for errors

```
polipo -c ~/etc/polipo.conf
```

then try and test it - if you can see Google's main page's HTML source then its
working

```
httpd_proxy=http://10.111.111.1:8123 curl http://www.google.com
```

## Launchd Configuration

It would be handy to have this run all the time so I have created a launchd file
you are free to use. 

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <false/>
    </dict>
    <key>Label</key>
    <string>com.example.polipo-http-cache</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/polipo</string>
        <string>-c</string>
        <string>/put/path/here/polipo.conf</string>
    </array>
    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
```

Replace "/put/path/here/polipo.conf" with the location of your Polipo config
file. Save that file in  /Library/LaunchAgents/Polipo-Caching-HTTP-proxy.plist.

Then start the service with

```
sudo launchctl -w load /Library/LaunchAgents/Polipo-Caching-HTTP-proxy.plist
```


# Future Enhancements

- Make it totally offline with DNS caching via something like dnsmasq.
