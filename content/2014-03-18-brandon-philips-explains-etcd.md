---
title: "Brandon Philips Explains etcd"
date: "2014-03-18T00:00:00-07:00"
template: "post"
draft: true
slug: "brandon-philips-explains-etcd"
category: "DevOps"
tags:
  - etcd
  - coreos
  - brandonphilips
  - docker
description: "To find out more about etcd, last week I met up with another member of the CoreOS team and a creator of etcd, Brandon Philips."
---
_This post was originally published on [ActiveState's blog](https://www.activestate.com/blog/), but is longer available there._

Last month I [spoke with Ben Golub](/ben-golub-explains-docker-inc), CEO of Docker Inc. We briefly discussed service discovery and how it relates to Docker. A close cousin of Docker is [etcd](https://github.com/etcd-io/etcd), since etcd and Docker seem to swim in the same circles. Neither is dependent on the other, but they are both components of building highly scalable systems and both have come about in the past 12 months.

Not long after Docker came on the scene we saw CoreOS begin to build their OS, designed specifically for running Docker containers. I [interviewed CoreOS CEO Alex Polvi](/alex-polvi-explains-coreos) last August to find out more about CoreOS and the component that they were building called etcd. We touched briefly on etcd, but did not go into details.

Since I spoke with Alex, we have kept a close eye on the project. I have seen etcd come up in conversation many times and start to make its way into several open-source projects. Notably, it is now being used in at least 2 components of Cloud Foundry.

To find out more about etcd, last week I met up with another member of the CoreOS team and a creator of etcd, [Brandon Philips](https://twitter.com/BrandonPhilips).

## What Is etcd?

I asked Brandon for his brief introduction into what etcd is.

“It’s a data-store for really important information. It’s tolerant of nodes going down. It gives you a way to store configuration data with consistent changes and distributed locks. The data is always available and always correct.”

A few uses-cases of etcd that Brandon listed are service discovery, master election, registration of services and coordination of process across a cluster. He noted that when you are coordinating a process across multiple nodes, you do not want to lose any data and you need to keep state.

_**Update 2013/03/20:** On the Docker mailing-list, Evan Krall asked whether the above quote implied that etcd solved CAP, to which Brandon replied, “etcd doesn’t solve CAP. The underlying consensus algorithm for etcd is Raft; it is consistent and partition tolerant in the CAP terms. What I meant by “available” is that the data is available for reads when quorum is lost.”_

## Starting Point

etcd started as a project within CoreOS. CoreOS is striving to “secure the Internet” from the server side. This means servers that can be scaled across large infrastructures and support many of the websites we all visit on a daily basis. When the scale increases, outages become an everyday (or every minute) occurrence. CoreOS wanted a good solution for handling fault tolerance, which is vital to anyone working with servers at scale.

Google internally created [Chubby](https://research.google/pubs/pub27897/) for dealing with their fault-tolerance and Yahoo! created the open-source project [http://zookeeper.apache.org/](http://zookeeper.apache.org/). These both provide distributed configuration and distributed locking.

The features that Brandon and his team wanted in etcd were for it to be highly-available and “approachable”. For high availability they wanted a consensus-based key-value store. To make it approachable they chose to go with a pure HTTP API, since anyone can interact with HTTP and it is supported by many languages.

I mentioned to Brandon that other similar projects, such as [Doozerd](https://github.com/ha/doozerd), offer a lighter-weight protocol, such as protocol buffers, and wondered if it was something they were looking at. Brandon said that HTTP is working well and so far they have not seen a need to change this.

## Alternatives

etcd is often compared to ZooKeeper and Doozerd. These all solve a similar problem.

Brandon admitted that ZooKeeper is obviously more battle-tested than the much younger etcd project. He had used ZooKeeper when he worked at Rackspace, but he was not intimately familiar with it.

ZooKeeper is not recommended for virtual environments. This is the key reason ActiveState chose Doozerd over ZooKeeper when we added clustered configuration into our Cloud Foundry solution, Stackato.

Doozerd was created by Heroku as a light-weight in-memory alternative to ZooKeeper. This became a key part of Stackato v2.0 and provided centralized multi-node configuration to replace local disk YAML files.

## Doozerd

In Stackato v3.0, we decided to drop Doozerd from our product due to some limitations we were hitting. This was at the same time that etcd came on the scene, but it was obviously too young to be a candidate. Instead we replaced Doozerd with Redis, using its pub-sub in place of Doozerd’s push notifications.

I asked Brandon if they had looked at Doozerd when building etcd. Brandon said that Doozerd is very hard to get into. The [Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science)) algorithm it uses is generally hard understand and it is difficult to add new features to Doozerd. In comparison, he said, the [Raft algorithm](https://en.wikipedia.org/wiki/Raft_(computer_science)), that etcd uses for consensus, can easily be explained on a white-board.

## The Raft Algorithm

Before looking at Doozerd, Brandon and his team had read the Raft paper by Diego Ongaro and John Ousterhout, entitled “In Search of an Understandable Consensus Algorithm.” They were excited about the paper and bounced some ideas off the brain of Heroku’s [Blake Mizerany](https://twitter.com/bmizerany). Heroku were the original creators of Doozerd.

The implementation of the raft algorithm is actually implemented separately to the etcd code-base. CoreOS helped build the [go-raft library](https://github.com/goraft/raft) that etcd then builds upon.

## Go

Like Doozerd, Docker and many others in this new era of cloud technologies, etcd and go-raft are written in [Go](https://golang.org/). Brandon described this as the sweet-spot for developing concurrency. It is a statically typed language with properties for concurrency that people can understand.

He has worked with Lua, but found that state is spread across too many closures. With Node.js he has found the callback structure difficult to reason about.

Go gave the right primitives. “Google did a great job with standard library. It impresses me every day,” said Brandon.

## Rolling Upgrades

One feature that etcd has that ZooKeeper has just added is rolling upgrades. This allows mixed versions of etcd to be running across the cluster. Each node publishes its etcd version and the maximum version that all nodes support is the version the cluster will support.

To allow this backwards compatibility, they have versioned the internal API of etcd.

I asked Brandon if there is a cut-off to the number of backwards compatible versions they can support. How far back can the latest version go to be backwards compatible? Brandon told me that luckily the etcd API is very simple and consists of simple commands. Therefore, they have not yet hit any limitations in providing backwards compatibility.

## Scaling

With consensus-based clusters, Brandon tells me, you usually need between 5 and 7 nodes. This allows 2 to 3 machines to go down without losing quorum.

etcd v0.4 will bring the concept of “proxies.” A “proxy” is a member of the cluster that simply passes off the requests to a “follower.” A follower is a full member of the consensus as defined by the raft protocol. Proxies store no data themselves.

When a follower node goes down, a proxy node can be automatically promoted to a follower. This is 2 step process. First the proxy must obtain a full snapshot of the data and second they must get all the updates since the snapshot was taken.

I asked Brandon what the cost of bringing up new follower was. He said it is dependent on the size of the data and it can get expensive if you are storing a lot of data. He suggested that the data in consensus systems should be kept small. This is because it is designed for storing really important information and its data is usually stored in memory.

## Docker

There is no standard way yet to do service discovery with Docker, but etcd seems like the best contender.

As Ben Golub, CEO of Docker Inc, said in my [previous post](/ben-golub-explains-docker-inc), when developing Docker, their “primary job is to build the hooks” into these external systems and not tightly couple Docker with any one solution.

Docker 0.65 introduced “Links“, which is a security feature for explicitly coupling container interfaces together, rather than exposing them to the whole world.

So, for instance, if container A was an application and container B was a database server, container A could be spawned with the “docker run” argument –link, which would specify container B. Container B’s database would be accessible to container A’s application, but accessible to nothing else.

```
Container A --> Container B
```

The problem here is that if the address of the database in container B changes, then you need to restart the application in container A. To decouple this, the “Ambassador Pattern” was introduced.

```
Container A --> Ambassador --> Container B
```

The ambassador is a light-weight container that simply redirects the traffic from the application in container A to the database service in container B. If the location of B changes, then we now only have to restart the ambassador container, since its interface for container A is consistent.

Taking this one step further, this can be expanded to work over the network by involving ambassadors at either side of the network.

```
Container A --> Ambassador >> network >> Ambassador --> Container B
```

Alex Polvi of CoreOS [expanded on this idea](http://coreos.com/blog/docker-dynamic-ambassador-powered-by-etcd.html) to include etcd into the equation. If your ambassadors can communicate with each other via etcd, then you start to have a good model for service discovery without having to bake the location of services, or knowledge of etcd, into your applications.

Of course, if the location of your etcd service is also prone to change, then you require an additional layer of ambassadors, as Alex demonstrated in his example.

Brandon tells me that when it comes to Docker, everyone is still feeling out the best way to do service discovery.

## Long-Polling Watchers

One of the features I liked about Doozerd was the ability of a client to watch a specific tree of the data and retrieve push notifications immediately when data changes. etcd has this same feature.

etcd’s interface is pure HTTP, so obviously this requires the watchers to be implemented with long-polling HTTP requests.

With Doozerd, client support was limited. Implementing good clients, or fixing existing clients, was fiddly and error-prone. Therefore I am a big fan of this HTTP approach that the etcd team has adopted. HTTP is widely supported.

The way watchers have been implemented means there is no room for updates to fall through the cracks, even if there is a lag between one long-polling HTTP request timing-out and establishing the next one request. The trick is using a monotonically-incrementing integer for each change to etcd.

With each HTTP request to etcd, we get the “index” which tells us when each data value was created and when it was updated. To wait for the next update, we simply pass our last known index with waitIndex and specify wait=true to long-poll until an update occurs.

```
curl -L 'http://127.0.0.1:4001/v2/keys/foo?wait=true&waitIndex=7'
```

Doozerd has the same concept, but uses the term “revision” instead of “index.”

## Discovery

To help with cluster setup, CoreOS have provided a service at [discovery.etcd.io](https://etcd.io/docs/v3.4.0/dev-internal/discovery_protocol/). This provides etcd “in the cloud” as a way to do peer discovery. When an etcd node in our cluster comes up it can find other etcd nodes in your cluster via this mechanism.

You can also run your own peer discovery endpoint quite easily.

## Fleet

The discovery service can also be used with CoreOS’s latest project, [fleet](http://coreos.com/blog/cluster-level-container-orchestration.html). This is a distributed init system which builds on etcd and [systemd](https://en.wikipedia.org/wiki/Systemd).

## Hidden Key-Space

Brandon told me that it is possible to have your etcd data live off the radar. If you prefix the keys with underscores then they will be excluded from GET requests by default. This means they need to be explicitly requested.

This reminded me of Memcached, where there is no easy way to get all the keys in the data-store. Brandon pointed out that the administrator will always be able to see on-disk what data is stored.

## Adoption

I hear a lot of talk about etcd. I was curious what Brandon was seeing. He said he sees a lot of interest, but mostly from start-ups.

The etcd mailing list is active, but they also see interactions from GitHub through pull requests and GitHub Issues. Brandon said that he personally prefers the long-threaded conversations you get from a mailing list over GitHub. This is especially true when discussing new APIs, since it is hard to discuss this on GitHub.

Currently, there are about 40 contributors to the project.

## Cloud Foundry

Brandon sees one of the biggest sources of interest in etcd from the Cloud Foundry project. Specifically this is from the folks working on the newer Diego project. When talking with the Diego authors, he said they were pleasantly surprised to hear that the author the Raft paper was also named Diego.

Within the Cloud Foundry project, Diego is currently in incubation and is a potential redesign of the DEA component of Cloud Foundry. The DEA is responsible for hosting application containers, but it is likely that Diego will do much more than that. etcd is a core component of this new strategy.

Another new component of the Cloud Foundry project, which is relying on etcd, is HM9000 (Health Manager 9000). This is a distributed version of the existing Health Manager, which ensures uptime for the correct number of application instances.

## etcd v0.3

etcd is currently at version [0.3](http://coreos.com/blog/etcd-0.3.0-released/). This version added improved cluster discovery.

The API is at version 2 and they are trying to keep it stable.

They want to treat etcd like a web service. Features will only be added to the API, rather than changing it or deleting from it. This ensures that they will not break usage as it develops.

As they move forward with development, they plan to focus on scaling requirements.

## Conclusion

etcd is a tool you want to have in your arsenal when scaling up your architecture.

Centralized configuration and service discovery are two key components of building dynamic service-oriented systems. With IaaS and scale comes volatility and an increase in the number of machines going up and down. This is especially true if you are scaling on-demand to get the best value out of your infrastructure budget.

etcd gives you a way to manage the key details of your cluster’s configuration in real-time. It should not be used for everything, as consensus systems like etcd work best with small datasets. Use it for orchestrating your clusters, registering and locating dynamic services.