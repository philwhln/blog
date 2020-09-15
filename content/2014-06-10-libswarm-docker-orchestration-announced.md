---
title: "libswarm – Docker Orchestration Announced"
date: "2014-06-10T00:00:00-07:00"
template: "post"
draft: false
slug: "libswarm-docker-orchestration-announced"
category: "DevOps"
tags:
  - libswarm
  - docker
  - solomon hykes
description: "Back in May 2013 I had interviewed Solomon Hykes to get the low-down on what Docker was. Now, exactly one year later, Solomon agreed to sit down with me to explain what he describes as “the second lego brick” – libswarm."
---
_This post was originally published on [ActiveState's blog](https://www.activestate.com/blog/), but is longer available there._

A year ago Docker was announced. Thanks to our 3 plus years experience working with Linux containers, ActiveState could clearly see its potential and understand Solomon Hykes’ vision for the future of containerization. We wasted no time making Docker part of Stackato.

Back in May 2013 I had [interviewed Solomon Hykes](/solomon-hykes-explains-docker) to get the low-down on what Docker was. Now, exactly one year later, Solomon agreed to sit down with me to explain what he describes as “the second lego brick” – libswarm.

## Docker

Before we get into what libswarm is, let’s first look at what Docker is – and more importantly what it is not.

Docker provides a standard interface to portable Linux containers. It offers an execution environment that comes in a simple package and can run on any Linux distribution.

For good reason, Docker has become extremely popular in the past 12 months with individuals, startups and large organizations. We see support starting to appear from Amazon Web Services, Google Compute Engine and a growing list of PaaS vendors, including ActiveState. The Cloud Foundry ecosystem now touches Docker at several points of integration.

As Solomon puts it, “Docker became popular because it was easy to use on a single node with a single container, which made it easy to use for prototyping and development.”

## libswarm

During his keynotes at Dockercon, Solomon introduced the next lego brick to follow Docker – “[libswarm](https://github.com/docker/libswarm)“.

“libswarm defines a standard interface to manage and orchestrate distributed systems. It is not tied to a particular distributed system,” he tells me.

The idea is that libswarm provides a consistent API into your clustered environment. It can work with any existing distributed system that implements an adapter (or “backend”) for libswarm to use.

Here’s a quick excerpt from the interview I did with Solomon for this blog post.









Why More Lego Bricks?
Running Docker on one machine is a breeze. Running it across multiple machines is not so simple. The solutions for a cluster of Docker hosts are numerous.

ActiveState provides a Cloud Foundry based PaaS, Stackato, that uses Docker under the covers, so you do not need to be concerned with the distribution of containers. There are other solutions such as Apache Mesos, CoreOS’s Fleet or Deis that also enables you to work with distributed Docker containers. This is not to mention all the [growing options for service discovery](/service-discovery-solutions), such as etcd, Consul or SkyDNS.

“The problem is that today there is no portable answer,” said Solomon. “There is a lot of competition. I think that’s the way it should be. I don’t think there should be a single dominant clustering system that everybody uses.”

So what is the answer? Portability and interoperability should be pillars of the cloud technology ecosystem. But how do we keep a vibrant, competitive and innovative cloud technology ecosystem, while still retaining some level of interoperability?

ActiveState strives for portability by retaining API compatibility with Cloud Foundry. Cloud Foundry itself plays well with other PaaS-related components such as Buildpacks. As we further integrate Cloud Foundry with Docker, we want to retain as much portability and interoperability with other Docker technologies as possible.

## Standard Interface

“Orchestration is pretty wide and libswarm takes a pretty pragmatic approach to it. You’ve got existing programs running on existing machines and they’re reachable over the network using existing protocols. Those programs are very diverse, those machines are all over the place and those protocols are very diverse.”

Solomon said that distributed systems such as Mesos, Cloud Foundry and Fleet have the notion of a task that is running. At some point you want to stop it or you want to kill it. “That’s low hanging fruit – it’s an easy thing to put in an abstraction,” he added.

Many of these solutions also have the concept of listing tasks or listing units of computation. You can select one from the list to take action on it. They all provide a way to see events, start all the components of an application, check access control and find out which user identity performed which actions. “There’s a large subset of these tasks, these interactions, that are really fundamentally the same and there’s room for an abstraction.”

Solomon admits that not everything can be extracted into a common-denominator across all systems. That is not the intent of libswarm. Instead it strives to address commonality and it is expected that the original backend APIs will still be used where needed.

“The difficulty is exposing an abstraction that is just thick enough to be useful, to save you time, but allows you to escape it when the abstract gets in the way,” says Solomon.

He said that there are existing abstractions that say “Forget you’re using Mesos, forget that you’re using BOSH, forget that you’re using DNS for service discovery. You’ll never have to know that fact again.” Solomon stressed that that is not what libswarm does: “We’re saying of course you need to know that and you are going to have to do things that are DNS specific and BOSH specific and Mesos specific. That should be possible, but it shouldn’t be mandatory.”

## Separation Of Concerns

When I [interviewed Docker Inc’s CEO Ben Golub](/ben-golub-explains-docker-inc) in February, something that stood out for me was the separation of concerns. He said that Developers care about what happens on the inside of the Docker container. Operations engineers care about what happens on the outside. I see this as being analogous to what libswarm is doing, but at a different level.

As an engineer building out and integrating with a distributed system, it makes sense to introduce as much decoupling as possible. Building a solution to work with the libswarm API means that you can swap out backends. It also means you can talk to multiple backends from different vendors in a consistent manner. Until you need to work with the specifics of any one backend system, you can build your solution in an abstracted way.

## The Lego Box

To be clear, libswarm is in no way an after-thought to the success of Docker. Docker came from dotCloud, which is obviously already running Docker at scale. Therefore, the understanding of how Docker fits into a larger system distributed system was always clear to them.

> “Docker was very much designed to build something bigger – to build distributed systems. Because that was part of the design, the API of Docker fits very well into a broader system.”

Solomon tells that there are many “lego bricks” that make up the public PaaS dotCloud. Docker is just the first one that they open-sourced. He said they wanted to make Docker a success before open-sourcing more components and that libswarm is not the last to be open-sourced. We will see more components in the future, but the rate at which components are released will likely increase.

## Docker API Superset

The Docker API was designed to be expanded for clustering and to essentially become the libswarm API. Therefore the libswarm API will initially be the same as the Docker API with additional cluster related APIs. Anyone working with Docker will already be familiar with libswarm’s API and will be able to start working with libswarm immediately.

Companies like Orchard are already using the Docker API in a similar way. You interface with their Docker API in the same way that you would with Docker on your local machine, but behind the scenes your Docker container is deployed within a clustered environment.

Solomon said that many companies and IT organizations are essentially rolling their own libswarm to build out clustered Docker containers. He thinks it is much better to combine this effort and build it together. Users of libswarm then have the option of building their own libswarm backend or taking an off-the-shelf backend.

## Who’s Involved?

Solomon and his colleagues at Docker Inc (formerly dotCloud Inc) are not alone in developing libswarm. In the same way that Docker was initially developed before it was released as open-source, there is a team of interested parties building out the initial implementation.

Google Cloud, Rackspace, Tutum, Deis, Orchard and Mesosphere are involved with others slowly coming on-board.

## Docker API v2

As the libswarm API evolves it will be the driver of the Docker API. At some point the libswarm API will actually become the Docker API v2. This means you can continue to use Docker as-is, or use the expanded functionality with a clustered backend.

## Backends / Adapters

(Solomon used the term “adapter” in the interview, but in the code it is “backend”, so these are synonymous).

Backends provide the implementation specific to different technologies.

Solomon describes a backend as “basically a new thread in memory that can be queried for information”. Queries can be “give me the list of nodes” or “give me a handle on this other node”.

It is completely up to the backend how it answers the query. For example, there is backend called the “simulator” backend. You initialize it a with list of nodes which are then stored in memory and simulate a network of nodes. You can query it in the same way you query other backends.

The “Fleet” backend connects to an etcd cluster and it will respond to node list queries by querying the information in the etcd cluster.

Backends can be wired together either across the network or within a running libswarm process. For instance, there may be a backend that utilizes both the Google Cloud backend and the Rackspace backend, providing a single libswarm API endpoint into you cluster.

Docker itself is just another backend to libswarm and was the first backend to be implemented.

## Tree Of Nodes

Solomon described the tree structure that libswarm uses. libswarm can talk to any node in your cluster that implements the libswarm API and ask for its child nodes, with which it can then interact.

“The API of libswarm exposes a tree of nodes”, he said.

Solomon also talked about “nesting” – “The fact that a node can have children nodes is a fundamental property of libswarm.”

This mapping-out of the tree of your infrastructure allows adapters to register new nodes anywhere in the tree.

## swarmd

libswarm is a library that can be embedded in your application. Alternatively you can simply use the swarmd daemon on your nodes, which itself uses the libswarm library.

## Polyglot

I asked Solomon whether using the Go programming language was a requirement for using libswarm, since the libswarm library is implemented in Go. Is it feasible to implement libswarm in other programming languages?

“It’s not just feasible – it’s actually right up there in the specs that it should be simple and made easy to port,” he said. “So the spec should be clear, using the simplest implementation choices so it’s easy to port.”

## Conclusion

Docker has been an incredible success and it is still early days for where it might go. As Solomon said, Docker solved the issues of portability and consistency between environments. libswarm takes Docker to the next level – orchestrating Docker clusters with the same interoperability.

He pointed out that they are not rebuilding an open-source version dotCloud piece-by-piece. Instead, they are learning from their experience of dotCloud and building these pieces with the community. That is what they did with Docker and it worked. They are using the same model with libswarm.

Solomon told me that the underlying technologies of Docker and libswarm are not new. Many of the pieces of core technology that they build upon have been around for 10 years. Some pieces have been around much longer. What they are doing is bringing the community together to utilize these technologies in a way that makes sense to everyone. He suggested that this should be the job of standards bodies, but that is unfortunately not working.

Solomon is pleased that the Docker community is a such happy place – unlike other open-source communities he has worked with. There are few egos and Solomon attributes this to people just wanting to see these technologies amalgamate into some real – something that should have happened a long time ago.

So what is this Docker/libswarm movement that we are seeing?

“It’s APIs backed by code and everyone’s welcome” – Solomon Hykes, Docker Inc.