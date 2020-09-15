---
title: "Service Discovery Solutions"
date: "2014-05-28T00:00:00-07:00"
template: "post"
draft: true
slug: "service-discovery-solutions"
category: "DevOps"
tags:
  - service discovery
  - chubby
  - zookeeper
  - doozerd
  - etcd
  - confd
  - consul
  - envconsul
  - serf
  - smartstack
  - 
description: "Overview of service discovery. What it is and does current technologies"
---
_This post was originally published on [ActiveState's blog](https://www.activestate.com/blog/), but is longer available there._

Last weekend I attended the [Polyglot Unconference](http://www.polyglotconf.com/) in Vancouver, which for the past 2 years has been successful in bringing together hundreds of smart people from around the Northwest to discuss all things tech.

During the unconference I ran a session on “Service Discovery” and this blog post will be a reflection of what was discussed.

## Definition

First, let’s start with a definition. What is “Service Discovery”?

The way I describe service discovery is that you have a cluster of machines running services that need to be found and need to find each other.

In large scale systems, where machines may come up and down, the location of specific services may change over time. How do applications and other components of your architecture, that rely on those services, find them and keep up to date with changes to their location?

Service discovery provides a centralized place to store up-to-date information on where a service is currently running. This is usually the list of IP and ports, but can also include additional information, such as whether a machine is a master or not.

Service discovery came about due to the volatile nature of large scale distributed systems. At scale, machines coming up and down will be a regular occurrence. Therefore, you will see that service discovery solutions are themselves designed to be distributed and highly available, due to the environment where they are most often used.

## Chubby

Many of the modern day service discovery solutions started after Google documented their internal tool, Chubby, in the paper “[The Chubby lock service for loosely-coupled distributed systems](http://static.googleusercontent.com/media/research.google.com/en//archive/chubby-osdi06.pdf)“.

Chubby provided locking across Google’s highly distributed systems. This locking enabled services to be able to elect new masters if an existing master died.

Chubby also provided a centralized place to store meta-data about the cluster. It is this meta-data that is at the core of service discovery, and where the details of where the service can be discovered is stored.

The paper gives an example of how Google used Chubby internally…

> Google File System uses a Chubby lock to appoint a GFS master server, and Bigtable uses Chubby in several ways: to elect a master, to allow the master to discover the servers it controls, and to permit clients to find the master.

Although Mike Burrows, who authored the paper, takes no credit for the algorithms or techniques used to create Chubby, it did pave the way for the open-source tools which followed.

> Building Chubby was an engineering effort required to fill the needs mentioned above; it was not research. We claim no new algorithms or techniques. The purpose of this paper is to describe what we did and why, rather than to advocate it.

Chubby is based on the Paxos algorithm for consensus between nodes on the current state of the data it stores. As Mike noted in the paper, at the time, “all working protocols for asynchronous consensus we have so far encountered have Paxos at their core.”

## ZooKeeper

> Because coordinating distributed systems is a Zoo

Yahoo! created [Apache ZooKeeper](https://zookeeper.apache.org/) as an open-source implementation based on the Chubby paper. Like Chubby it provides locking, master election and a centralized place to store meta-data on the clusters makeup.

ZooKeeper is used by a number of open-source distributed systems such as Juju, Katta, Mesos, Neo4j, Apache Hadoop, Apache Kafka and Apache Solr.

For a long time it has been the goto solution for orchestrating distributed systems and has removed the need to reinvent the wheel when building up such systems.

The algorithm ZooKeeper uses is [Zab](https://cwiki.apache.org/confluence/display/ZOOKEEPER/Zab+vs.+Paxos), which is similar to Paxos, but “is primarily designed for primary-backup systems, like Zookeeper, rather than for state machine replication.”

## Doozerd

[Doozerd](https://github.com/ha/doozerd) was created by Heroku as a light-weight alternative to ZooKeeper and provides a subset of its functionality.

Unlike ZooKeeper, Doozerd runs entirely in-memory. It uses the Paxos algorithm to come to consensus on the current state of affairs amongst its peers.

Doozerd is essentially a key-value store where keys are defined as file paths. Clients can read all the data stored in Doozerd or a subset (branch or path) of the data.

For instance, if my service configuration is stored in Doozerd, it is accessible to my applications. Depending on which service I require, I can either request all configuration under /services/mysql or all configuration under /services/postgresql.

```
/services/mysql/192.168.1.2:3306/username = "admin"
/services/mysql/192.168.1.2:3306/password = "password"
/services/postgresql/192.168.3.4:5432/username = "admin"
/services/postgresql/192.168.3.4:5432/password = "password"
```

This example is structured in such a way that if there were multiple instances of the service or multiple entry points (via IP and port) into that service, they would be available to the application. Riak would be an example where there would be many IPs from which to access the Riak database – essentially one for each node in the Riak cluster.

Another feature of Doozerd is the ability to watch configuration and respond to changes in real-time. It is fine to read the configuration when the application starts up or periodically, but this results in lag or potential down-time. Having updates pushed to the client when configuration changes is much more powerful and this is what Doozerd watchers provide. A client can issue a callback to be fired if a specific branch of the tree changes.

## etcd

[etcd](https://github.com/etcd-io/etcd) is a new but popular option for service discovery. Created by CoreOS Inc to complement their CoreOS Linux-based cloud operating system, etcd is a great option for anyone starting out with service discovery.

When etcd was announced last year, Doozerd was already failing to gain community traction. At ActiveState we used Doozerd as a component of Stackato, but dropped its usage due to several issues encountered with the technology. etcd appears to solve most of those issues and definitely does not suffer when it comes to a healthy community.

The data structure of etcd is the same as Doozerd, as mentioned above. It is a key-value store, where keys are file-like paths. Clients have the option to retrieve all data, all data under a specific path, or a specific key. etcd also offers the same watcher functionality, so that clients can get real-time updates when configuration changes.

etcd’s interface is a HTTP REST interface making it easy to consume in any programming language. Watchers are implemented using HTTP long-polling.

The [Raft algorithm](https://en.wikipedia.org/wiki/Raft_(computer_science)) is used for consensus between peers, which is based on the Paxos algorithm, but differs in being easier to explain and easier to implement. A common complaint of Paxos is that, while it is simple at its core, it has so many edge cases making it hard to implement.

You can read more about etcd in the [interview I did with etcd creator Brandon Philips](/brandon-philips-explains-etcd) earlier this year.

## Discovering Discovery

Service discovery provides a great way to find services across your infrastructure, but how do you find the service providing the service discovery?

Usually you will have a known starting point or rely on DNS to find the service. Sometimes DNS is not an option or might not fit – for instance if you are working with Docker containers. If you are firing up a cluster on Amazon AWS and the cluster comes up with all new IPs then it will be hard for etcd nodes to find each other and for other components of your system to find them.

CoreOS provides a service for those on the public cloud with [discovery.etcd.io](https://etcd.io/docs/v3.4.0/dev-internal/discovery_protocol/). This service runs etcd and provides you with a starting point to bootstrap your cluster.

## confd

[confd](https://github.com/kelseyhightower/confd) is a service that allows for easy integration between etcd and local configuration files on the machine where an application is to run. For instance, it can update your PostgreSQL configuration files from a template and variables stored in etcd. This is done whenever the values changes in etcd. This saves needing to integrate your applications directly with etcd, which helps with supporting legacy systems.

## Consul

Announced only [a month ago](https://twitter.com/mitchellh/status/456839193671385090), [Consul](https://www.consul.io/) is very young. That said, it is well documented and carries the HashiCorp banner.

HashiCorp was founded by Mitchell Hashimoto, who is the creator of Vagrant – the popular virtualization tool for developers. HashiCorp’s line-up of projects also includes Serf (mentioned below).

Like etcd, Consul is based on the Raft protocol for consensus. But Consul also makes uses of Serf’s gossip protocol for LAN and WAN membership status and leader election.

Like etcd and others, Consul’s features include service discovery and a similar key-value store, but also provides health checking functionality and the ability to span multiple data-centers out-of-the-box.

The multiple data-center support comes from Consul’s use of its eventually consistent gossip protocol. I find this first class support for multiple datacenters to be an interesting leap ahead of other solutions that leave it up to the implementer to figure out.

## envconsul

Similar to confd (see above) providing a bridge between etcd and applications, [envconsul](https://github.com/hashicorp/envconsul) provides a similar role between Consul and your applications.

envconsul is based on [envdir](http://cr.yp.to/daemontools/envdir.html) and wraps the startup of processes so that it can pass variables stored in Consul’s key-value store as environment variables into the process. Like confd, it also supports restarting processes when configuration values change.

## Serf

[Serf](https://github.com/hashicorp/serf) is also a product of HashiCorp and is a building block of Consul (see above). That said, Serf itself is a tool for light-weight service discovery and orchestration.

> The Serf agents periodically exchange messages with each other in much the same way that a zombie apocalypse would occur: it starts with one zombie but soon infects everyone. In practice, the gossip is very fast and extremely efficient.

Serf seems to have a strong ability to detect and diagnose down nodes and networks outages, due to its random probing and quickly communicating other nodes of issues.

## SmartStack

[SmartStack](https://medium.com/airbnb-engineering/smartstack-service-discovery-in-the-cloud-4b8a080de619#.m0x2ks9ja) is a service discovery solution from Airbnb, which has been battle-tested by them for over a year and publicly available for 6 months. I have heard good things about this from several Vancouver based companies and it was also mentioned in the Service Discovery session I ran at the Polyglot Conference.

SmartStack is composed of two components: [nerve](https://github.com/airbnb/nerve) and [synapse](https://github.com/airbnb/synapse).

Nerve provides the service registration (IP and port pairs) using ZooKeeper and Synapse then reads that information and configures a local HAProxy process to route local requests from each machine to appropriate service.

## Conclusion

This space is definitely heating up. 12 months ago there were few options for service discovery other than ZooKeeper or something homegrown. This post would definitely have been easier to write 12 months ago! As these tools become more widely adopted, I’m sure we will see them evolve and new solutions enter the arena.

ActiveState uses service discovery solutions such as Doozerd, etcd and others to build our Stackato private PaaS solution, so we are very interested in being up-to-date in this space.

For the most part, PaaS itself removes the need for service discovery – automatically wiring service credentials into application instances and freeing developers from the concerns of working with service discovery infrastructure. This is the ethos of the [12 Factor App](https://12factor.net/config) and something you will often hear ActiveState [talking about](https://www.activestate.com/blog/).

Though simple in design, tools like confd and envconsul are key to replicating the power of PaaS for homegrown infrastructure. They remove the burden from the developer for understanding the wiring of the service discovery setup.
