---
layout: post
title: "Redis High Availability Options"
date: 2014-11-28 09:11
categories: general
---

This article outlines the various Redis HA options and the tradeoffs between
them as well as touching how service discovery might be used.
There are two main options (other than not doing Redis HA)

* **Redis with Sentinel** - Requires quite a few components but is currently more production ready and does not require client modification.
* **Redis Cluster** - Results in a simpler architecture (from an admin point of
view) but, according to the Redis developers, not yet production ready and
requires client (app library) support. So your application needs to be modified
in order to understand Redis clustering (primarily the redirects)
 
I will explain it all below as I understand it - feedback welcome!

# Redis With Sentinel and Twemproxy

**Credit**: The ideas for Redis + Sentinal come from [this excellent blog post](http://www.jambr.co.uk/Article/redis-twemproxy-agent) written by Karl Stoney.

#### Consistent Hashing

This set up uses consistent hashing to spread data across many master instances
and HA is achieved by a) using this consistent hashing (via Twemproxy) to
ensure data is placed and retrieved from the correct master and b) having
slaves attached each master instance for eventual failover (via Sentinel).  


#### Architecture

The most promising architecture requires quite a few components to achieve high
availability but I can't think of a better way to do it at the moment; small
proof-of-concept tests show that it works. The stack could be something like
this (**[picture here](http://jambr.blob.core.windows.net/articleimages/redis-sentinel.png)**)
for a given environment.

1.  Redis servers (obviously) - at least two instances to be master and slave
2.  [Redis sentinel](http://redis.io/topics/sentinel) - Triggers master/slave 
changes in order to ensure availability (at least two instances) 
3.  [Twemproxy](https://github.com/twitter/twemproxy) - Transparently sends
requests to the current master (at least two instances) 
4.  [Twemproxy agent](https://github.com/Stono/redis-twemproxy-agent) or [smitty](https://github.com/areina/smitty) - Restarts twemproxy to send requests to the new master when a failover
happens (runs alongside twemproxy querying Sentinel for failover events) 
5.  Load balancer pair - Some kind of highly available (e.g. using VRRP or
[Pacemaker](http://www.linux-ha.org/wiki/Pacemaker) load balancer at the TCP
level to protect against twemproxy failures. Typically haproxy or a commercial
appliance can fulfil this role.

(Again, a
**[picture](http://jambr.blob.core.windows.net/articleimages/redis-sentinel.png)**
says a thousand words)

Depending on your application and tolerance of technological footprint you could
possibly get away with just one master and a set of slaves but if your application
uses Redis in a serious way you'll want to spread the data across multiple masters.

#### Service Discovery

The role of Consul/Service Discovery Registry in all this is that it informs the various components as follows

* Tells Sentinel where to find the Redis masters for the environment it's in
* Tells Twemproxy Agent (or Smitty) where to find the Sentinel host for the environment it's in
* Tells the applications where to find either the LB or Twemproxy (if not using an LB)

# Redis Cluster

[Redis cluster](http://redis.io/topics/cluster-tutorial) 
does not use consistent hashing but rather shards the data across
all nodes participating in the cluster; the concept of masters and slaves are
still used with Redis cluster but in a different way. This is explained in
detail on the Redis [cluster tutorial](http://redis.io/topics/cluster-tutorial) 
but I'll go over it here for pass on the info and also to cement the knowledge in my head.

#### Cluster Sharding

During bootstrap you set up some Redis servers in "cluster mode" and then kick
them off all at once with a special "cluster create" command specifying the
amount of replicas per master. A replica being the "slave" - so if you have 4
masters then you'll get 4 slaves for a total of 8 instances. So you'd need to
provision 8 instances ahead of time by starting them up in that "cluster mode".

They will then divide themselves up with a fixed set of "slots" (16384). So if
you have 4 master nodes they'll each get 4096 slots as follows 

* Node A : 0 - 4095
* Node B : 4096 - 8191
* Node C : 8192 - 12288
* Node D : 12289 - 16384

#### Handling Failures

If one of the master nodes fail then a slave is automatically promoted to
master and the show goes on. Upon recovery the former master will rejoin the
cluster and continue as a slave. The downside being, that if you add a new
master, you have to manually re-shard the data by moving slots from existing
instances to the new instance. Maybe they'll make this transparent in the
future.

#### Client Support

How does a client know who to connect to? The nice thing is that, because the
client must be aware of the cluster, it can connect to any instance and it will
receive redirects telling it where to write to for a given "set" command (or
where to fetch from for a given "get"). From an architectural perspective we
can treat the Redis cluster more like a "pool" of servers rather than a set of
distinct components.

#### Service Discovery

As for service discovery - you could have a Consul agent on each cluster node
advertising it as available for taking requests. As long as the client can understand
the Redis clustering protocol additions then you can connect to a slave or master and send
read or write requests.

