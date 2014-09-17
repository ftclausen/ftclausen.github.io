---
layout: post
title: "Rundeck - Authentication and Authorisation Notes"
date: 2014-09-17 08:09
categories: general
---

# Introduction

[Rundeck](http://www.rundeck.org) has a powerful authentication and
authorisation infrastructure making it suitable for use in organisations with
many different teams. For example the all developers can be given authorisation to
trigger deployments to all test environments but only a subset of developers and
operational folk can be given permission to deploy to production.

I am making this page because **it took some fiddling** before our installation
could authenticate and authorise users. I am hoping to save someone else similar
issues if they happen to stumble upon this while Googling.

# LDAP Configuration Highlights

In most organisations a directory of some sort will already exist; such as
Active Directory or NDS. We need to configure Rundeck to use these. I will touch
on the process involved highlighting the parts I that needed that differed from
the defaults; see the [auth
docs](http://rundeck.org/docs/administration/authenticating-users.html) for full
details.

## JAAS Login Module

It took me ages to get this working because I needed the following in my JAAS
config; see below for a full example

* **forceBindingLogin=true** - Make sure that we are binding as the search user
    first; I suspect it needs this to read the password hashes as the
    authenticated users did not seem to be able to read their own hash.
* **userPasswordAttribute="unicodePws"** - *This may differ* in your case. Make sure
    you specify the correct attribute for a user that contains the password
    hash.
* **forceBindingLoginUseRootContextForRoles="true"** - As above, in our case, we
    needed to search for roles using a privileged bind user.
* **userObjectClass="posixAccount"** and **roleObjectClass="groupOfNames"** -
    Make sure this corresponds to the actual object classes that contains your
    user and group attributes. For example *inetOrgPerson* and *group* are
    common user and group object classes respectively.
* **roleSearchSubtree=true** - Search the whole groups tree
* **supplementalRoles="user"** - This is more specific to Rundeck itself; this
    is required if you wish to have more than one group authenticating to
    Rundeck. You need to make sure this matches the *security-role* element in
    $RDECK_BASE/server/exp/webapp/WEB-INF/web.xml - more on that below

Below is a complete example that worked for me - note the "ldap" on line one and also the file name; in thise case */etc/rundeck/jaas-ldap.conf*

{% highlight properties linenos=table %}
ldap {
    com.dtolabs.rundeck.jetty.jaas.JettyCachingLdapLoginModule required
    debug="true"
    contextFactory="com.sun.jndi.ldap.LdapCtxFactory"
    providerUrl="ldaps://ldap0.example.com"
    bindDn="cn=ldapclientuser,ou=service-users,o=services"
    bindPassword="secret"
    authenticationMethod="simple"
    forceBindingLogin="true"
    userBaseDn="o=myorg"
    userRdnAttribute="cn"
    userIdAttribute="cn"
    userPasswordAttribute="unicodePwd"
    userObjectClass="posixAccount"
    forceBindingLoginUseRootContextForRoles="true"
    rolebasedn="ou=usergroups,ou=groups,o=myorg"
    roleNameAttribute="cn"
    roleMemberAttribute="member"
    roleObjectClass="groupOfNames"
    roleSearchSubtree=true
    cacheDurationMillis="300000"
    reportStatistics="true"
    supplementalRoles="user";
};
{% endhighlight %}
# Authorisation via *.aclpolicy Files


