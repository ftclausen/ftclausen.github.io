---
title: "Jenkins Slave on Windows Prep"
date: 2016-06-16 15:06
classes: wide
categories: general
---

While not sexy we have a product that is officially supported on Windows and
uses SQL server so in order to expand our Windows CI efforts we need to add lots
of Windows + SQL Server combo slaves.

This article outlines how to provision a Windows node to maximise its readiness
to act as a Jenkins slave.

What the Chef cookbook at the core of this HOWTO does is

* **Install Git**
* **Install the JDK**
* **Install the JDK JCE extensions**
* **Set the registry ownership so that the master can install the slave remotely**
* **Set a schedule task to stop the Jenkins service at boot**

Why the last point? Because the master will manage the Jenkins slave process on the slave 
and if it is already running then it is unable to start it immediately.

# Items that already need to be in place

* **chef-client** - this needs to be part of the base Windows image if you don't
    want to install it every time.
* **Chef tool chain on your workstation** - The [Chef Development
    Kit](https://downloads.chef.io/chef-dk/) provides a convenient way to do
    this.

# Install Location

In the bootstrap.json attributes file I adhere to the following conventions

* JDK Location - `c:\jdk1.8`
* Jenkins Slave Location - `c:\jenkins`
* Provisioning tool home - `c:\chef-jenkins-provisioner`

# Steps

## Clone the chef_jenkins_provisioner repo

This lives [over on Github](https://github.com/ftclausen/chef_jenkins_provisioner)

## Customise bootstrap.json

You need to set the following to values that reflect your environment. I'll list
these as Chef node attributes to make navigating **bootstrap.json** easier :

* `node['java']['windows']['url']` - URL to Windows JDK. I recommend using a local mirror.
* `node['java']['windows']['package_name']` - Set this to exactly the name the package gets when installed by the MSI installer. E.g. *Java SE Development Kit 8 Update 91 (64-bit)*
* `node['java']['windows']['checksum']` - The SHA256 checksum of the installer file
* `node['jenkins']['windows']['set_acl_binary_url']` - The URL to the [SetACL.exe](https://helgeklein.com/setacl/documentation/command-line-version-setacl-exe/) binary
* `node['jenkins']['windows']['set_acl_binary_url_checksum']` - The SHA256 checksum of the binary
* `node['jenkins']['windows']['jce_export_policy_jar']` - The URL to the US_export_policy.jar file
* `node['jenkins']['windows']['jce_export_policy_checksum']` - Checksum for above
* `node['jenkins']['windows']['jce_local_policy_jar']` - The URL to the local_policy.jar
* `node['jenkins']['windows']['jce_local_policy_checksum']` - Checksum for above

## Vendor the cookbook_chef_jenkins_provisioner

We now need to make a self-contained installer archive that can be extracted and
allow installation of the Jenkins slave without a Chef server or even an
Internet connection if your resources are all within your perimeter.

Making a cookbook and its dependencies available for Chef server-less is called
"vendoring" and is done with the `berks` command as follows **from within the
`cookbook_chef_jenkins_provisioner` cookbook directory**

    berks vendor ../cookbooks

which will output the cookbook and dependencies to `chef_jenkins_provisioner`
repo. The directory structure will look like this after you've vendored the
`cookbook_chef_jenkins_provisioner` cookbook :

    .
    ├── README.md
    ├── bootstrap.json
    ├── config.rb
    ├── cookbook_chef_jenkins_provisioner
    │   ├── Berksfile
    │   ├── Berksfile.lock
    │   ├── README.md
    │   ├── attributes
    │   ├── files
    │   ├── metadata.rb
    │   ├── recipes
    │   └── templates
    └── cookbooks
        ├── apt
        ├── build-essential
        ├── chef_handler
        ├── chef_jenkins_provisioner
        ├── compat_resource
        ├── dmg
        ├── git
        ├── java
        ├── mingw
        ├── seven_zip
        ├── windows
        ├── yum
        └── yum-epel

## Zip up chef_jenkins_provisioner repo as a whole

Zip up the whole repo (so the `chef_jenkins_provisioner` directory) e.g.

    cd ..
    zip -r chef_jenkins_provisioner.zip chef_jenkins_provisioner

then copy this file to the Windows server and extract to `c:\` i.e. `c:\chef_jenkins_provisioner`

## Run stand-alone Chef to prep the machine

Once all of the above has been done then we can get Chef to do the rest of the heavy lifting

    cd c:\chef_jenkins_provisioner
    chef-client -z -c config.rb -j bootstrap.json

this will install the JVM, Git and set all the required registry permissions

## Add as Jenkins slave

Now we need to add the slave by going to the main Jenkins UI as an admin user and

* Select **Manage Jenkins**
* Select **Manage Nodes**
* Select **New Node**

and of most importance is the **launch method** which needs to be `Let Jenkins control 
this Windows slave as a Windows service`. Provide the rest of the pertinent info
such as the credentials and host name.

Once that is done you should be good to add this machine as a Jenkins slave.

# Further automation possibilities

* Run all this on a template Windows instance and then create an image from it.

# Troubleshooting

* Reboot - sometimes Chef mysteriously won't run directly after installation or if you have any other "weird" issues give the box a reboot.
* Try to 1) Stop the Jenkins service then 2) immediately attempt to start the agent again from the Jenkins master UI.
* Check the Windows event viewer under *Windows Logs* -> *Application* for error messages.
* Check the detailed [DCOM troubleshooting steps](https://wiki.jenkins-ci.org/display/JENKINS/Windows+slaves+fail+to+start+via+DCOM) on the Jenkins project wiki.

