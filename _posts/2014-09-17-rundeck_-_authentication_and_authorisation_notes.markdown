---
title: "Rundeck - Authentication and Authorisation Notes"
date: 2014-09-17 08:09
classes: wide
categories: infra
---

[Rundeck](http://www.rundeck.org) has a powerful authentication and
authorisation infrastructure making it suitable for use in organisations with
many different teams. For example the developers can define, modify
and delete projects and jobs while the operations folks can only execute, view 
or kill jobs within projects.

I am writing this page because **it took some fiddling** before our installation
could authenticate and authorise users in the way we wanted. Hopefully this will
save someone else some trouble.

<section id="table-of-contents" class="toc">
  <header>
    <h3>Overview</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section>

# LDAP Configuration Highlights

In most organisations a directory of some sort will already exist; such as
Active Directory or NDS. We need to configure Rundeck to use one of these. I will touch
on the process involved highlighting the parts I that modified to get our 
set up working; see the [auth
docs](http://rundeck.org/docs/administration/authenticating-users.html) for full
details.

## JAAS Login Module

I needed the following in my JAAS config before things would work; a full example
follows below

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

Below is a complete example that worked for me - note the "ldap" on line one 
and also the file name; in this case */etc/rundeck/jaas-ldap.conf*

<figure>
    <figcaption>File: /etc/rundeck/jaas-ldap.conf</figcaption>
{% highlight html %}
{% raw %}
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
{% endraw %}
{% endhighlight %}
</figure>

Then edit the **/etc/rundeck/profile** and modify the **RDECK_JVM** to use "-Djava.security.auth.login.config=/etc/rundeck/jaas-ldap.conf". See [authenticating users](http://rundeck.org/docs/administration/authenticating-users.html#ldap) 
for a full example; see "Step 2 - Specify login module".

# Allowing multiple groups to log in

Rundeck seems overly focused on having The One Group to rule them all. If you
are not in the One Group then you can't log in at all. But, thankfully, there is
a way to work around this with the above mentioned **supplementalRoles** JAAS
configuration attribute. 

This attribute *adds a particular* role to all users
logging in. This means that you can leave the *$RDECK_BASE/server/exp/webapp/WEB-INF/web.xml* file as default. See [authenticating
users](http://rundeck.org/docs/administration/authenticating-users.html) for
full details.

# Authorisation via *.aclpolicy Files

I highly encourage you to read [the fine
manual](http://rundeck.org/docs/administration/access-control-policy.html). After a careful RTFM I came to the following conclusions

* All /etc/rundeck/*.aclpolicy files are read
* There are two types of "things" you can grant/deny access to
  * Specific resources - this is a concrete resource or set of resources (if
      using a regex match)
  * Resource types - this applies to all the "types" of a given resource e.g.
      all "jobs", or all "nodes".

As best I could determine you need both definitions to grant specific, non admin access. Our
particular use case was to have the following groups have the following access

* Admin Group (Admin_Group in example below) - This is the standard admin group
    ACL that allows access to everything.
* Ops Group (Ops_Group in example below) - This group needs to view, execute and
    kill jobs.

So without further ado, the examples

### /etc/rundeck/admin.aclpolicy
{% highlight yaml %}
description: Admin, all access.
context:
  project: '.*' # all projects
for:
  resource:
    - allow: '*' # allow read/create all kinds
  adhoc:
    - allow: '*' # allow read/running/killing adhoc jobs
  job:
    - allow: '*' # allow read/write/delete/run/kill of all jobs
  node:
    - allow: '*' # allow read/run for all nodes
by:
  group: [Admin_Group]

---

description: Admin, all access.
context:
  application: 'rundeck'
for:
  resource:
    - allow: '*' # allow create of projects
  project:
    - allow: '*' # allow view/admin of all projects
  storage:
    - allow: '*' # allow read/create/update/delete for all /keys/* storage content
by:
  group: [Admin_Group]
{% endhighlight %}

### /etc/rundeck/ops.aclpolicy

This one was more tricky as, in my case, I had to not only specify the generic types 
but also the specific resources using regex. I could not just specify **only** the generic types.

{% highlight yaml %}
---
description: "Ops Engineers can launch jobs but not edit them"
context:
  project: .*
for:
  resource:
    - equals:
        kind: 'node'
      allow: [read,update,refresh]
    - equals:
        kind: 'job'
      allow: [read,run,kill]
    - equals:
        kind: 'adhoc'
      allow: [read,run,kill]
    - equals:
        kind: 'event'
      allow: [read,create]
  job:
    - match:
        name: '.*'
      allow: [read,run,kill]
  adhoc:
    - match:
        name: '.*'
      allow: [read,run,kill]
  node:
    - match:
        nodename: '.*'
      allow: [read,run,refresh]
by:
  group:
    - Ops_Group

---
context:
  application: rundeck
description: "Ops Engineers can launch jobs but not edit them"
for:
  project:
    - match:
        name: '.*'
      allow: [read]
  system:
    - match:
        name: '.*'
      allow: [read]
by:
  group:
    - Ops_Group
{% endhighlight %}

# Troubleshooting

I found the following helpful

* [The Administrator Guide](http://rundeck.org/docs/administration/index.html)
* The audit log - rundeck.audit.log (in /var/log/rundeck when installed from RPM)
* The service log - service.log (in /var/log/rundeck when installed from RPM)

Happy Rundecking!

