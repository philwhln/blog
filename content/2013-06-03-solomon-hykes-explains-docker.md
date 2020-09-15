---
title: "Solomon Hykes Explains Docker"
date: "2013-06-03T00:00:00-07:00"
template: "post"
draft: true
slug: "solomon-hykes-explains-docker"
category: "DevOps"
tags:
  - docker
  - solomon hykes
  - dotcloud
description: "I just had a call with Solomon Hykes from dotCloud about Docker, who gave me the low-down about Docker."
---
_This post was originally published on [ActiveState's blog](https://www.activestate.com/blog/), but is longer available there._

I just had a call with Solomon Hykes from dotCloud about Docker, who gave me the low-down about Docker.

## What is Docker?

Docker brings the power of [Linux containers (LXC)](https://en.wikipedia.org/wiki/LXC) and [aufs](https://en.wikipedia.org/wiki/Aufs) (Another Union File System) to create a new level of packaging and process isolation that really is something you should sit up and take note of.

The first thing to mention about Docker is that it is a single executable daemon. That means it is easy to download and start running docker images on your Linux-based operating system. This docker daemon manages the docker processes, which run inside of LXC containers.

A docker process is an instance of a docker image, and a docker image has one primary process that will run when you instantiate it.

Have a spare 100 milliseconds and you want to run CouchDB? “docker run couchdb”. A second later CouchDB will be running in an isolated container. It will be unaware that it is running inside of Docker.

Want to run a headless Firefox with a VNC interface under Suse Linux? Just run the docker image for that. Want to run 20 instances of that docker image on the same machine? Just run “docker run” 20 times. Each process thinks it is binding to the same port. Suddenly you are testing a Cassandra or Hadoop cluster on one machine.

## Docker Images

Docker images are essentially a collection of files which include everything needed to run that process. This is everything from the OS packages and up.

A docker image has a default process it runs when it is instantiated. This could be bash, to drop you into the terminal, or a web server so you can access it from the browser.

To construct a docker image you use “docker build” which uses a docker configuration file. This file is usually about 5-6 lines of unix commands, such as apt-get. You can think of it as a bash script, but it is much simpler, and it is targeted towards the construction of image layers. Therefore, you might have a line for the base Operating System, then for installing Apache web-server, then a line for installing your programming language runtimes and then your code, or the process, that you want to run.

The simplest docker image is “busybox,” which is 5Mb linux distribution, so it is very small.

## Layers

Docker images are built up in layers. So, for instance, if you need to run WordPress, you would build the Ubuntu layer, add a layer for Apache2 web server, add a PHP layer and then a layer for the WordPress files. Lower layers can be re-used. We might take the PHP layer and layer on Drupal instead of WordPress, or update our WordPress layer with a newer version or WordPress.

Because we can re-use layers, we can make new docker images very cheaply. We can create a new docker image by changing just a single line of one file and we do not have to rebuild the whole stack.

The beauty of docker images being “just files” means that the difference between two docker images is just a diff of the files they contain.

## Docker Push and Pull

Docker has the commands “docker pull” and “docker push” which are used to fetch docker images from either the public docker repository or your private repository.

The re-use of layers means that “docker pull” only needs to download the additional layers. If all your docker images have the same basic stack (e.g. Ubuntu 12.04 + some base packages) then this will be very quick, since you only require the files for the top-level layers.

The file diff-ing also gives the push and pull a git-like optimization when pulling and pushing image updates.

## It’s Fast!

Docker uses copy-on-write, which essentially means that every instance of your docker image uses the same files until one of them needs to change a file. At that point, it copies the files and creates its own version. This means that often a docker image will not need to write anything to disk to spawn its process. That makes docker fast! We’re talking “100 milliseconds” fast.

Let’s rewind. When you run “docker run,” it’s firing up everything in the stack needed to run that process.

In the words of Solomon, it “allocates a new LXC container, allocates the filesystem for it, mounts a read-write layer, allocates a network interface, sets an IP and then executes the process.”

## Portability

“The whole point is portability. I can create a docker image in then push it out to run on Amazon EC2 or on any Linux machine running docker.”

## What Docker Means For Virtual Machines?

Solomon and I discussed virtual appliances and whether docker images would replace virtual appliances that are packaged as virtual machines. Solomon thought that in the short-term it is unlikely, since most IaaS providers have a marketplace that hosts appliances in the form of a virtual machine, such as Amazon’s AMIs.

What you could do though, is easily deploy a docker image inside a simple Linux-based virtual machine running docker. It would be trivial to strip away the docker layer if it was considered redundant overhead, but there are some advantages to keeping it.

## Upgrading Virtual Appliances

Upgrading a docker image is a simple “docker push” or “docker pull,” which only transfers the files that have changed. This is much simpler than an apt-get install or code compilation that might be otherwise required.

Yes, you can blow away your virtual machines and replace them with a new virtual machine image, but there is cost to doing that. The IaaS APIs sometimes fail, sometimes new machines do not come up, and sometimes machines come up but perform badly. It is not uncommon for high-scale users, such Netflix, to have to cull poorly performing machines and have a continuous process to find good machines. This should not be how it is, but unfortunately it is the current reality we live in. A simple upgrade process via docker, would mean keeping good virtual machines running and upgrades that happen in seconds rather than minutes or longer.

The second important point to remember is that one virtual machine can run more than one docker image than can be identical. Therefore, you can have instant switch-over from the older version and the new version of the docker image and even keep the older docker process running until you are happy that the upgrade is successful.

Solomon pointed out that while the ideal of virtual machines was to bring us closer to the software layer, current virtual machines are still close to the hardware layer, and we still have to deal with issues that the hardware layer brings. Docker, on the other hand, is completely at the software layer.

## Common Use-Cases

“Developer experiments” is one use-case. Solomon gave the example of “hey, I’ve never used Suse linux before. I can fire up a docker image and immediately I’m in the shell. I can play around and then throw away that instance when I’m done”.

The use-cases seems to unbounded. Consistency, portability, speed, and the ephemeral nature of docker images seem to be the key drivers in creating new docker images. It is early days, so “can I do this?” seems to be a key theme. The answer is often yes.

## Interesting Use-Cases

Solomon thought it would be cool to run headless Firefox with a VNC interface. You can find this on the public docker repository and run it yourself.

Another interesting use-case is a company called Veezio who does a lot of image processing, such as picking text out from within the images. Their stack is quite complex, because it requires many Python modules, which in turn requires several system packages which need to compiled. Building this stack with docker images has made it easy to do this once and re-use and deploy the architecture easily. New docker images only need to worry about the high-level code changes.

## Limited to a Single Process?

While a docker image only has a single primary process, that process can be anything. This includes a process manager, such as supervisord. Therefore, depending on your architecture, you can have multiple docker images running individual processes or include a process manager within your docker image to manage those processes together in one docker image.

## Limitations and Requirements

The key requirement is a Linux Kernel with appropriate LXC support. This means docker will not run on your Macbook Air or Window XP machine.

## Data Volumes

Solomon mentioned the concept of “data volumes.” This is a way to ensure that certain files are not bundled with the docker image. Examples would be file uploads for WordPress or database files from PostgreSQL.

As Solomon pointed out, sometimes the initial database files — such as the PostgreSQL table structure — are required for setup. This can be included in the docker image, but as you start to populate the database you want to keep that separate from the docker image. You might also want to share that data volume between docker images. Docker images can include this information in the configuration.

## Public Registry of Docker Images

Docker development has been very community-focused, and the public registry of docker images has enabled people to get involved and share their docker images. You can find the “Docker Index” at [https://index.docker.io/](https://index.docker.io/)

It is interesting to browse through and see what you can already run on docker. One docker image that I found was “Doozerd,” which is used in Stackato. I can see it would be a good way to write system tests against our doozerd integration points and have the test suite fire up a docker image for Doozerd. There would be no worries about ensuring the machine had all the dependencies or worry about cleaning up afterwards. Doozerd (in binary form), much like docker, actually does not have any dependencies, but you get the idea.

## Private Registry of Docker Images

One thing Solomon pointed out is that while the public repository of docker images is very important, it is limited when enterprises begin to start using docker and need somewhere to host their own private docker images. Therefore, they have released a private version of the Docker Index as open source.

## Conclusion

Docker is here to stay, and this space is only going to get more interesting as the number of public docker images increases, and Docker is more widely adopted. Docker is not part of your stack, but rather it is going to be a key tool for DevOps, alongside virtual images, vagrant, Puppet, Chef, and IaaS integrations.

Docker is currently at 0.3, with 0.4 coming out soon. The key milestone is 1.0, which is scheduled for the end of July and is geared towards stability and production quality Docker.

With spring ending, summer about to start, and an end of July deadline, I asked Solomon, “So nobody gets a summer holiday?” His response echoed his obvious enthusiasm and passion for this new technology: “Working on this is my holiday!”
