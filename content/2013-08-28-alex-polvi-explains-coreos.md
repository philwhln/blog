---
title: "Alex Polvi Explains CoreOS"
date: "2013-08-28T00:00:00-07:00"
template: "post"
draft: false
slug: "alex-polvi-explains-coreos"
category: "DevOps"
tags:
  - docker
  - coreos
  - etcd
description: "CoreOS was announced a few weeks ago and seemed to answer this question. For this blog post I interviewed Alex Polvi, the CEO of CoreOS, to find out more."
---
_This post was originally published on [ActiveState's blog](https://www.activestate.com/blog/), but is longer available there._

A couple of months ago we interviewed [Solomon Hykes about Docker](/solomon-hykes-explains-docker), which is a way to build and manage Linux Containers with a lot of nice features. The next question was: if the full-stack can be provided by a Docker image and everything can be Dockerized, what is the minimum OS we need to run Docker images?

[CoreOS](https://coreos.com/) was announced a few weeks ago and seemed to answer this question. For this blog post I interviewed Alex Polvi, the CEO of CoreOS, to find out more.

I should note, that before talking to Alex, I imagined CoreOS to simply be a Linux distribution with everything stripped out, leaving only what is required to support Docker. As you will see, it is a lot more than that and the hopes Alex has for CoreOS go beyond just running Docker images.

## The Company

CoreOS is the name of the product and the company. CoreOS, the company, is a start-up with a strong team. They come from Rackspace, Google, Novell and Cisco. [Greg Kroah-Hartman](https://en.wikipedia.org/wiki/Greg_Kroah-Hartman), the current Linux kernel maintainer for the “-stable” branch, is an advisor to the team and has helped with some of the implementation details.

Alex, the CEO, previously built a company called Cloudkick, which was acquired by Rackspace at the beginning on 2011. Cloudkick’s office then became the San Francisco base for Rackspace.

## "Secure The Internet"

Prior to starting Cloudkick, Alex worked in the Operations team at Mozilla. With Firefox, Mozilla’s goal was to secure the Internet. Historically, the web-browser has been the best window into most of the computers of the planet and in the early days of the Internet it was easy to get a computer virus or malware through your web-browser.

Mozilla aimed to address this rampant lack of desktop security with the Firefox web-browser. Since Firefox was open-source, they were able to quickly iterate and rapidly find and fix known vulnerabilities. They quickly overtook other web-browsers, such as Internet Explorer, in terms of security. Security became the biggest selling point when convincing users to move to Firefox.

Alex noted that a more recent change in web-browsers, that has improved security of the Internet significantly, is *automatic updates*. After all, what it point in rapidly fixing vulnerabilities if there is a huge lag in users updating to a newer version.

Automatic updates was pioneered by the Chrome web-browser, rather than Mozilla. Mozilla knew that this would be a huge leap forward in terms of securing the Internet, but they were blocked by “online rights” and strong opinions that users should be in full control of everything that is installed on their system. Google, who were not so entrenched in the web-browser space, pushed forward with their new Chrome browser and paved the way for automatic updates. Mozilla were then able to quickly follow.

For many years now, we have seen a strong focus on securing the Internet from the front-end, through the browser. Alex believes that there has been much less progress on the backend, with servers. There have been no significant leaps in security, similar to what we have seen from web-browsers. Alex hopes that CoreOS can be such a progression and his hope is to “secure the Internet” from the server-side.

# Automatic Updates

CoreOS was originally based on ChromeOS. They liked the automatic updates model that, similar to the Chrome web-browser, ChromeOS implements. They used this as a starting point.

Since this starting point, things have changed a lot. CoreOS now uses a different Linux kernel and init system. Since ChromeOS was intended to be run on laptops, ChromeOS’s kernel provided a lot of support for “random hardware”, which they did not need on the server-side. CoreOS also makes heavy use of systemd.

With CoreOS, automatic updates happen in the background. CoreOS has two disk partitions, with only one active at a time. The inactive partition can be updated offline. You simply swap the disks and reboot to enable the updates. This reboot only takes between half a second to a second.

Alex points to this as a big win for security. With Debian or Yum updates you always have an inconsistent state during each package update, which is window of vulnerability. Also, packages may or may not restart processes for you. This is done incrementally as each package is updated. CoreOS aims to replace this incremental and volatile update mechanism with a whole machine “atomic upgrade”.

This atomic upgrade does involve the machine being rebooted, which Alex admits is not ideal, but should not be a problem if viewed as single piece of a larger distributed system. In a distributed system, machines going up and down should be tolerated.

## Read-only File System

It is important to understand that the active disk partition for the root filesystem not only does not change during an upgrade, but it never changes while it is in use. CoreOS uses a read-only partition for the root filesystem and this includes /etc. This is another security feature that CoreOS employs.

## etcd

A key component of CoreOS is etcd, which is a distributed configuration service. This is essentially a key-value store that provides an HTTP interface for retrieval and monitoring of configuration settings.

etcd is similar to Apache Zookeeper or Doozerd, but aims to fill a need that neither of these are fulfilling right now, according to Alex. To quantify this need, Alex noted that on Github etcd has received nearly 900 stars in the 3 weeks since it has been released.

At the core of etcd is the consensus algorithm, Raft. They have put a lot of work into creating consensus that actually works and provides fault tolerance. Alex admits that it is not perfect and they need to see more companies battle testing it, but they have been heavily focused on this area of their technology and he seems confident in its implementation.

etcd is essentially a shim around the Go libraries that they have developed. They hope these libraries, Go-raft, will be used by others outside of etcd.

In comparing etcd’s fault tolerance to Doozerd and Zookeeper, Alex said that it is better than Doozerd, but not as good as Zookeeper.

One concern he has with Zookeeper is that it is heavily focused on Java. The client bindings are based on Java, so while clients *have* been implemented in other languages, it is difficult to create good ZooKeeper clients in new languages.

## Distributed configuration for Docker

It is expected that Docker images will utilize etcd for their configuration, so that configuration can be shared across containers throughout the cluster. There is currently no defined way to do this, so it is hoped that developers will implement the best way relevant to the technologies they are using.

## Service Discovery

Another usage of etcd, which CoreOS hopes developers will adopt, is service discovery. Services running as processes inside Docker containers register themselves in etcd. These services are then known to all other processes in the CoreOS cluster, which can then find and utilize those services.

## Open-Source

Everything that CoreOS (the company) is doing is open-source. Similar to dotCloud with Docker, right now they are focused on the building technology above all else.

## Google Parallels

Alex notes that much of what CoreOS is bringing to open source is already practiced at Google. Google uses distributed locking, as provided by etcd. They already deploy applications packaged with all their dependencies, similar to the model provided by Docker and similar the atomic updates of CoreOS’s root filesystem.

Google also contributed a lot of the functionality behind Linux Containers, such as cgroups, to the Linux Kernel. These Linux Containers are the core technology behind Docker.

## Conclusion

CoreOS and etcd are very new technologies. It is about 3 weeks since they were announced, but like Docker there is a lot of buzz around them. The team behind CoreOS seems strong and they do already have some customers, which is key to the success of any startup.

I like ethos behind CoreOS. “Secure the Internet”. I am interested to see other ways that CoreOS plans to drive innovation in this space.

As it stands, CoreOS seems like a good solution for hosting Docker images, and etcd could be a great solution for distributed configuration as it matures.