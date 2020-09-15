---
title: "Ben Golub Explains Docker Inc"
date: "2014-02-27T00:00:00-07:00"
template: "post"
draft: false
slug: "ben-golub-explains-docker-inc"
category: "DevOps"
tags:
  - docker
  - dotcloud
  - bengolub
description: "Last week, I met with Ben Golub, Docker Inc's new CEO, to find out more about where they see things going."
---
_This post was originally published on [ActiveState's blog](https://www.activestate.com/blog/), but is longer available there._

Last June I met with [Solomon Hykes](/solomon-hykes-explains-docker) from dotCloud to talk about Docker. We discussed what Docker was, who was using it and the potential it had to change how things are done. A lot has changed since then for both the technology and for dotCloud.

With the obvious and immediate success of Docker, dotCloud changed its focus from public PaaS provider to focus on making Docker the best technology it can be. Their public PaaS is still alive and maintained, but they are very open about which direction they are heading.

In July, Ben Golub joined as CEO. This enabled Solomon Hykes, founder and then CEO, to become CTO and truly focus on the technology.

In October the company, dotCloud, was renamed to [Docker Inc](https://www.docker.com/) to clarify to anyone unsure where their focus was and to be clear focal point of the community. Although it already was.

Last week, I met with Ben Golub to find out more about Docker Inc and where they see things going.

## One Year Anniversary

In March, it will be Docker’s one year anniversary. Ben tells me that this first year has been about getting Docker stable, production ready and ubiquitous. This will be the continued direction right up to Docker 1.0.

As a company, Docker Inc wants to invest in the technology, invest in the community and provide commercial offerings around it.

In the past 6 months they have formed many partnerships with companies like Google, Red Hat and Yandex (the “Google of Russia”). PaaS solutions, such as ActiveState’s Stackato, have incorporated Docker. IaaS solutions, such as OpenStack, now have native support for Docker.

In June, just after I interviewed Solomon, Docker Inc became a part of the Linux Foundation. Docker relies on many core Linux technologies, such as LXC containers and cgroups. Docker is also driving adoption of these technologies.

## Hiring

In January, Docker Inc closed a $15 million round of funding. Ben tells me they are hiring lots of great developers and expanding their team. So far, James Turnbull has joined as VP of Services. He was previously VP of Technology Operations at Puppet Labs. Roger Egan, previously a VP at Red Hat, also joins Docker Inc as Senior VP of Sales and Channels.

From what Ben tells me, it seems that hiring developers to work on Docker and surrounding technologies is not a challenge. Development of Docker is out in the open. On GitHub and the mailing lists, they can see who is diving deep into the Docker code-base, who is enthusiastic about the technology, who is getting involved in the community and who is innovating around it.

## Commercialization

Docker Inc is focusing on centralized services for Docker. They want to make it easy to find and publish Docker images. Ben tells me that there is a lot that can be done around work flow and the meta data of Docker images. When you are using a Docker image, how do you know where it came from and how it was built?

Docker Inc also wants to help with tools for orchestration and migrations. For instance, how do you seamlessly move your Docker infrastructure from one cloud to another?

Ben also mentioned the potential for monitoring services and how monitoring may be integrated with Docker containers.

They are looking at launching some public cloud services in May. This follows the GitHub model of building public cloud solutions, then moving onto private solutions and eventually enterprise solutions.

Docker Inc has a good history of building multi-tenant scalable supported public cloud services, as they have done with the dotCloud public PaaS. This was the birth-place of Docker and so it makes sense to start commercial offerings in this space.

Ben tells me that they have a strong focus on Sysadmin tools.

## Matrix From Hell

I asked Ben how he sees Docker fitting into the relationship between Developers and Operations. What does this mean for DevOps?

Ben described what we have now as “the matrix from hell”. There are so many platforms, which range from mobile, to tablets, to laptops, desktops and servers. Each one of these is infinitely variable. This is one axis of matrix. On the other axis is software, the packages and solutions. Supporting and navigating this matrix is currently a nightmare. It is hard to quantify, manage, test and it is unsupportable.

Ben admits that we do have many great tools with which we try to automate away and simplify parts of the matrix, but says that these tools are brittle and do not scale well to the complexity of the matrix.

Docker redefines this matrix.

Docker separates the rows and columns into two distinct concerns. Developers care about what happens on the inside of the Docker container. Operations engineers care about what happens on the outside.

Ben said that DevOps defines two different people with different concerns. Developers like to experiment and build new features. Operations engineers care about reliability and uptime. By clearly separating these things with Docker, we enable them to work well together rather than stepping on each others toes as they stumble around the matrix from hell.

## Integrations

I asked Ben how Docker is integrating with tools like Chef, Puppet and others.

Docker is designed to integrate well with others.

The first round of integrations was driven by the enthusiastic community. Docker Inc have been involved in some integrations, but it does not scale. Round two will be first class official integrations driven by the owners of the integrating technologies.

One of the first integrations came from OpenStack. Docker containers can be managed and deployed on OpenStack via the [Docker virt driver](https://wiki.openstack.org/wiki/Docker). This is available in the Havana release of OpenStack and the Icehouse release will see inclusion of Docker 1.0.

## Community Outreach

There are numerous conferences and meet-ups around the world. Attendees of these are all keen to learn about Docker. I asked Ben how Docker Inc can scale to this demand in the same way Docker scales. “We can’t,” Ben told me. “Sending out lots of boxes of whale stickers and helping publicizing the events is as much as we can do.”

I am sure that they are secretly working on a technology to clone and deploy Solomon Hykes.

## Openness

Ben told me that Docker and Docker Inc have a very open mentality. Docker is provided with the Apache license, so that anyone can build upon it. They want open design, including the road-map for Docker. There will be no surprises and there are many maintainers of Docker who are not employees of Docker Inc. Discussion and code commits are all done in the open, on public mailing lists and public code repositories. Ben clarified that there are no secret repositories or code held back, as you sometimes see with companies that manage an open-source project.

## One Direction

There are many voices in such a large and growing community as is Docker’s. So far, Ben says they have seen very little contention for which direction Docker should go. He attributes this to putting extensibility as a core feature of Docker.

Docker is designed to be extended with a plug-in architecture, so if somebody wants to change any of the fundamental components of Docker, they can.

Solomon is the chief maintainer of the Docker project. It is his vision that will ultimately drive the direction of the project. He will do this in an open way.

## Releases

Docker aims for monthly releases. Their biggest priority now is 1.0. The current release is 0.8. In 1.0, they want a shrunken core, stable API and a coherent plug-in architecture.

Ben wants to see Docker running on all major platforms. He wants to see a low number of bugs and high quality documentation. Commercial support is important.

## Docker Index

They have already build a platform around Docker with the Docker Index. This is a public service that allows anyone to upload and share Docker images. The Docker Registry, that this is built on, is also available to anyone who wants to deploy their own registry.

Ben said they want to extend this to be able to trace the source code used to build the index and get more clarity on who the developers are. Anyone hosting their own Docker registry should be able to tap into this.

## LXC Core

I asked Ben about their relationship with the LXC core that Docker builds upon.

“We’re always looking for ways to contribute”, Ben told me. For instance, Greg Kroah-Hartman is an advisor to Docker Inc and contributor to Docker. Greg is the current Linux kernel maintainer for the -stable branch and a Fellow at The Linux Foundation. He also advises the CoreOS team.

Docker is focused on the layer above LXC. Ben states that they are “standing on the shoulders of giants”.

## Pluggability

Although, their hope is that the shoelaces of Docker will not be intimately tied to the shoulders of the giants on which they stand. With the plug-in architecture provided by Docker, it is intended to be possible to replace LXC with libvert, Solaris Zones or BSD jails.

Pluggability also applies to the file-system Docker uses, the security features (SE Linux) and they want to continue to enable this for all aspects of Docker.

## Service Discovery

Anyone playing with Docker at scale has probably started to look at service discovery. The developers of CoreOS quickly recognized this need when they started developing Etcd quite soon after Docker was announced.

Etcd seems to be the current best option for service discovery. Other options are ZooKeeper, which does not play well with virtual environments and Doozerd, which seems to have reached end-of-life.

Again, plug-ability is important. Service discovery should be just as pluggable. Ben stated that their “primary job is to build the hooks” into these external systems and not tightly couple Docker with any one solution. For instance, if somebody wants to use [Mesos](http://mesos.apache.org/) they can. If somebody is a Maestro or YARN fan then they should be able to use that.

## Host OS

I hear discussions around which operating system is best to run Docker inside. There are development environments, such as boot2docker and new operating systems, such as [CoreOS](/alex-polvi-explains-coreos), designed specifically for running Docker containers. Some people will still want to run on RHEL or Ubuntu LTS in production. I asked Ben what Docker Inc’s opinion is on this.

“Choosing the tools people use is not our job”, Ben told me. “Some people will want to use Docker in a revolutionary way, others in an evolutionary way. We want to remain agnostic.”

Ben said that organizations often choose RHEL, because it is reliable. Some like CoreOS, although it is still young. Different environments have different requirements. Do you want Docker inside a full OS or is being light-weight more important?

## Strong Opinions

I asked Ben how they deal with strong opinions about the direction that Docker should take or the priorities of features that should be implemented. Again, Ben pointed to the openness of the project.

“If you have an opinion, then send a pull request. If you have a strong opinion, become a maintainer.”

## Conclusion

In summary, I would say that the Ben Golub is helping building Docker Inc into a very open company to drive Docker forward with the technical guidance and vision of CTO Solomon Hykes. As Ben told me, they aim to do “less better, than more poorly.”

We look forward to Docker 1.0!