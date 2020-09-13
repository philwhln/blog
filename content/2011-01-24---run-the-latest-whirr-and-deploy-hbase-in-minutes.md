---
title: "Run The Latest Whirr And Deploy HBase In Minutes"
date: "2011-01-24T16:17:00-08:00"
template: "post"
draft: false
slug: "run-the-latest-whirr-and-deploy-hbase-in-minutes"
category: "DevOps"
tags:
  - amazon ec2
  - cloud-computing
  - hadoop
  - hbase
  - maven
  - nosql
  - ruby
  - whirr
  - zookeeper
description: "In a few of my recent posts I have covered the ease of deploying clusters of Hadoop and Cassandra using Whirr. With Whirr you can simply write a"
socialImage:
  publicURL: "/media/images/2011/01/whirr_hbase_sq.jpg"
---
<a href="https://www.flickr.com/photos/wheatfields/3938695154/" title="Fast, faster, faster! by net_efekt, on Flickr"><img align="left" alt="Fast, faster, faster!" height="240" src="/media/images/whirr_0_3_0/whirr_hbase.jpg" width="320"/></a>

<div style="min-height: 240px">
In a few of my recent posts I have covered the ease of deploying clusters of <a href="/map-reduce-with-ruby-using-hadoop">Hadoop</a> and <a href="/quickly-launch-a-cassandra-cluster-on-amazon-ec2">Cassandra using Whirr</a>. With Whirr you can simply write a configuration file specifying which cloud provider you are using, your credentials and the definition of the cluster you desire and it will build it for you. In this post, I am going to look at the latest Whirr, version 0.3.0, which is currently in <em>release candidate</em> status. I will show you how you can find and build the latest Whirr and live on the edge of what it possible.
</div>



<ul style="padding-bottom: 30px">
<li><a href="#grabbing-the-source-code">Grabbing The Source Code</a></li>
<li><a href="#make-sure-you-have-those-dependencies">Make Sure You Have Those Dependencies</a></li>
<li><a href="#building-whirr">Building Whirr</a></li>
<li><a href="#launch-a-hbase-cluster&gt;Launch A HBase Cluster&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a href=">Destroy!</a></li>
<li><a href="#conclusion">Conclusion</a></li>
</ul>

<a id="grabbing-the-source-code"></a>

## Grabbing The Source Code

On [the source repository for Whirr](https://svn.apache.org/repos/asf/incubator/whirr/), we can find the latest and greatest source code for Whirr. 

[![](/media/images/whirr_0_3_0/twitter_post.png)](https://twitter.com/andreisavu/status/28815577430622208)

We will use the code from [trunk](https://svn.apache.org/repos/asf/incubator/whirr/trunk), which will give you the very latest version of the software.

Even though version 0.2.0 is currently the most recent stable version we can download on [the official Whirr site](https://incubator.apache.org/whirr/), 0.3.0 is available here under the tagged releases and hence the trunk. So let’s download the source for that. Version 0.3.0 will soon be made the official release, but by following this step-by-step guide, you will be ahead of the pack.

```

cd ~/src
svn co 
```

<a id="make-sure-you-have-those-dependencies"></a>

## Make Sure You Have Those Dependencies

```
cat BUILD.txt
```

Looking at the dependencies in [BUILD.txt](https://svn.apache.org/repos/asf/incubator/whirr/trunk/BUILD.txt), you can see there is a few things we need. 

<blockquote style="background: #eeeeee; padding: 15px; margin: 15px;"><p>Apache Whirr Build Instructions</p>
<p>REQUIREMENTS</p>
<p>- Java 1.6<br/>
- Apache Maven 2.2.1 or greater<br/>
- Ruby 1.8.7 or greater (to run build-tools/update-versions)</p>
<p>BUILDING</p>
<p>To run unit tests and install artifacts locally:</p>
<p>mvn clean install</p>
<p>To build a source package:</p>
<p>mvn package -Ppackage
</p></blockquote>

If you have followed through one of my [previous](/quickly-launch-a-cassandra-cluster-on-amazon-ec2) [posts](/map-reduce-with-ruby-using-hadoop), then you will likely have these dependencies all installed, but I will review them just encase.

Being on Mac OS X, I generally use [Homebrew](/homebrew-intro-to-the-mac-os-x-package-installer) so install my dependencies. This will install Maven on Mac OS X if you have [Homebrew installed](/homebrew-intro-to-the-mac-os-x-package-installer).

```
sudo brew install maven
```

If you are on Debian Linux (eg. Ubuntu) use apt-get to install Maven.

```
sudo apt-get install maven2
```

There are also instructions available for installing [Maven on Windows](https://maven.apache.org/download.html#Windows_2000XP) or [other platform](https://maven.apache.org/download.html#Installation).

If you are using the latest Mac OS X then you should have Ruby 1.8.7 and Java 1.6 installed. If you have to upgrade your Ruby, then I recommend checking out [Ruby Version Manager (RVM)](https://rvm.beginrescueend.com/).

Here’s how to check your versions and what I’m currently running…

```

# Ruby
ruby -v
ruby 1.9.2p94 (2010-12-08 revision 30140) [x86_64-darwin10.5.0]

# Java
java -version
java version "1.6.0_22"
Java(TM) SE Runtime Environment (build 1.6.0_22-b04-307-10M3261)
Java HotSpot(TM) 64-Bit Server VM (build 17.1-b03-307, mixed mode)

# Maven
mvn -v
Apache Maven 2.2.1 (r801777; 2009-08-06 12:16:01-0700)
Java version: 1.6.0_22
Java home: /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
Default locale: en_US, platform encoding: MacRoman
OS name: "mac os x" version: "10.6.6" arch: "x86_64" Family: "mac"
```

[![](/media/images/whirr_0_3_0/twitter_post2.png)](https://twitter.com/tom_e_white/status/27233840057548800)

<a id="building-whirr"></a>

## Building Whirr

Let’s build Whirr 0.3.0!

Run the following command from inside the source directory.

```
cd ~/src/whirr-trunk
mvn clean install
```

The first time I installed this, it took 17 minutes to build, most of which was downloading dependencies. It failed downloading one of the dependencies and so failed to build.

```
[WARNING] Unable to get resource 'org.apache.hadoop:hadoop-core:jar:0.20.2'
from repository central (https://repo1.maven.org/maven2):
GET request of:
org/apache/hadoop/hadoop-core/0.20.2/hadoop-core-0.20.2.jar
from central failed

[INFO] ------------------------------------------------------------------------
[ERROR] BUILD ERROR
[INFO] ------------------------------------------------------------------------
[INFO] Failed to resolve artifact.

Missing:
----------
1) org.apache.hadoop:hadoop-core:jar:0.20.2
```

The path to this dependency was ok, so it must have just been one of those unfortunate glitches in the magic workings of the Internet. Please let me know if you have a similar experience. The chances of this happening to you are very slim.

I ran “mvn clean install” once more. 

```
mvn clean install
```

This time is only took 2 minutes and built successfully.

```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] ------------------------------------------------------------------------
[INFO] Whirr ................................................. SUCCESS [6.779s]
[INFO] Apache Whirr Build Tools .............................. SUCCESS [2.017s]
[INFO] Apache Whirr Core ..................................... SUCCESS [11.104s]
[INFO] Apache Whirr Cassandra ................................ SUCCESS [3.332s]
[INFO] Apache Whirr Hadoop ................................... SUCCESS [9.983s]
[INFO] Apache Whirr ZooKeeper ................................ SUCCESS [7.584s]
[INFO] Apache Whirr HBase .................................... SUCCESS [57.760s]
[INFO] Apache Whirr CLI ...................................... SUCCESS [38.708s]
[INFO] Apache Whirr Hadoop ................................... SUCCESS [1.884s]
[INFO] ------------------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2 minutes 20 seconds
[INFO] Finished at: Mon Jan 24 12:49:58 PST 2011
[INFO] Final Memory: 90M/123M
[INFO] ------------------------------------------------------------------------
```

Now that our code is compiled, we can run the final build command.

```
mvn package -Ppackage

(reduced output)

[INFO] Scanning for projects...
[INFO] Reactor build order:
[INFO]   Whirr
[INFO]   Apache Whirr Build Tools
[INFO]   Apache Whirr Core
[INFO]   Apache Whirr Cassandra
[INFO]   Apache Whirr Hadoop
[INFO]   Apache Whirr ZooKeeper
[INFO]   Apache Whirr HBase
[INFO]   Apache Whirr CLI
[INFO]   Apache Whirr Hadoop
[INFO] ------------------------------------------------------------------------
[INFO] Building Whirr
[INFO]    task-segment: [package]
[INFO] ------------------------------------------------------------------------
```

<a id="launch-a-hbase-cluster"></a>

## Launch A HBase Cluster

Currently in trunk there is a _recipe_ under the “recipes” directory called “hbase-ec2″. We can copy that, although in this example we are not going to modify it.

```
cp recipes/hbase-ec2.properties .
```

There is many comments in there, so here is a summary. 

```
whirr.cluster-name=hbase
whirr.instance-templates=1 zk+nn+jt+hbase-master,5 dn+tt+hbase-regionserver
whirr.provider=ec2
whirr.identity=${env:AWS_ACCESS_KEY_ID}
whirr.credential=${env:AWS_SECRET_ACCESS_KEY}
whirr.hardware-id=c1.xlarge
whirr.image-id=us-east-1/ami-da0cf8b3
whirr.location-id=us-east-1
```

In the above, the line you will want to play with is “whirr.instance-templates”, as this defines the shape and size of your cluster. Increasing the value “5″ will give you a bigger cluster.

We have a total of 6 machines here. 1 machine runs all the master services and 5 more machines run all the worker services. Here is breakdown of the services running on those machines, as defined by _whirr.instance-templates_.

<table>
<tr>
<td>1 zk+nn+jt+hbase-master</td>
<td>1 = 1 instance of the following
<p>zk = Zookeeper<br/>nn = Hadoop Name-Node<br/>jt = Hadoop Job Tracker<br/>hbase-master = HBase Master</p></td>
</tr>
<td>5 dn+tt+hbase-regionserver</td>
<td>5 = 5 instances of the following
<p>dn = Hadoop Data-Node<br/>tt = Hadoop Task-Tracker<br/>hbase-regionserver = HBase Region Server</p></td>
</table>

If you are interested in how all these Hadoop and HBase components work together, see Lars George’s excellent posts “[HBase Architecture 101 – Storage](https://www.larsgeorge.com/2009/10/hbase-architecture-101-storage.html)” and [HBase Architecture 101 – Write-ahead-Log](https://www.larsgeorge.com/2010/01/hbase-architecture-101-write-ahead-log.html)” or check-out the [HBase wiki](https://wiki.apache.org/hadoop/Hbase/HbaseArchitecture).

In previous Whirr examples I have defined the Amazon EC2 credentials in this properties file, but the above will pick them up from the environment, which is a better way to go. Export you credentials into your environment (here I use dumby credentials as an example). 

```
export AWS_ACCESS_KEY_ID=123456789ABCDEFGHIJKLM
export AWS_SECRET_ACCESS_KEY=ABCDabcd1234/xyzXZY54321acbd

```

We can now launch our cluster.

```
bin/whirr launch-cluster --config hbase-ec2.properties

Bootstrapping cluster
Configuring template
Starting 1 node(s) with roles [jt, nn, zk, hbase-master]
Configuring template
Starting 5 node(s) with roles [tt, hbase-regionserver, dn]
Nodes started: [[id=us-east-1/i-e134808d, providerId=i-e134808d, tag=hbase, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-da0cf8b3, os=[name=null, family=ubuntu, version=10.04, arch=paravirtual, is64Bit=true, description=ubuntu-images-us/ubuntu-lucid-10.04-amd64-server-20101020.manifest.xml], userMetadata={}, state=RUNNING, privateAddresses=[10.112.205.48], publicAddresses=[184.72.159.249], hardware=[id=c1.xlarge, providerId=c1.xlarge, name=c1.xlarge, processors=[[cores=8.0, speed=2.5]], ram=7168, volumes=[[id=null, type=LOCAL, size=10.0, device=/dev/sda1, durable=false, isBootDevice=true], [id=null, type=LOCAL, size=420.0, device=/dev/sdb, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdc, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdd, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sde, durable=false, isBootDevice=false]], supportsImage=is64Bit()]]]
Nodes started: [[id=us-east-1/i-f734809b, providerId=i-f734809b, tag=hbase, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-da0cf8b3, os=[name=null, family=ubuntu, version=10.04, arch=paravirtual, is64Bit=true, description=ubuntu-images-us/ubuntu-lucid-10.04-amd64-server-20101020.manifest.xml], userMetadata={}, state=RUNNING, privateAddresses=[10.194.127.79], publicAddresses=[50.16.154.13], hardware=[id=c1.xlarge, providerId=c1.xlarge, name=c1.xlarge, processors=[[cores=8.0, speed=2.5]], ram=7168, volumes=[[id=null, type=LOCAL, size=10.0, device=/dev/sda1, durable=false, isBootDevice=true], [id=null, type=LOCAL, size=420.0, device=/dev/sdb, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdc, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdd, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sde, durable=false, isBootDevice=false]], supportsImage=is64Bit()]], [id=us-east-1/i-fb348097, providerId=i-fb348097, tag=hbase, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-da0cf8b3, os=[name=null, family=ubuntu, version=10.04, arch=paravirtual, is64Bit=true, description=ubuntu-images-us/ubuntu-lucid-10.04-amd64-server-20101020.manifest.xml], userMetadata={}, state=RUNNING, privateAddresses=[10.98.33.250], publicAddresses=[50.16.71.166], hardware=[id=c1.xlarge, providerId=c1.xlarge, name=c1.xlarge, processors=[[cores=8.0, speed=2.5]], ram=7168, volumes=[[id=null, type=LOCAL, size=10.0, device=/dev/sda1, durable=false, isBootDevice=true], [id=null, type=LOCAL, size=420.0, device=/dev/sdb, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdc, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdd, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sde, durable=false, isBootDevice=false]], supportsImage=is64Bit()]], [id=us-east-1/i-f5348099, providerId=i-f5348099, tag=hbase, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-da0cf8b3, os=[name=null, family=ubuntu, version=10.04, arch=paravirtual, is64Bit=true, description=ubuntu-images-us/ubuntu-lucid-10.04-amd64-server-20101020.manifest.xml], userMetadata={}, state=RUNNING, privateAddresses=[10.195.6.143], publicAddresses=[204.236.242.78], hardware=[id=c1.xlarge, providerId=c1.xlarge, name=c1.xlarge, processors=[[cores=8.0, speed=2.5]], ram=7168, volumes=[[id=null, type=LOCAL, size=10.0, device=/dev/sda1, durable=false, isBootDevice=true], [id=null, type=LOCAL, size=420.0, device=/dev/sdb, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdc, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdd, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sde, durable=false, isBootDevice=false]], supportsImage=is64Bit()]], [id=us-east-1/i-f9348095, providerId=i-f9348095, tag=hbase, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-da0cf8b3, os=[name=null, family=ubuntu, version=10.04, arch=paravirtual, is64Bit=true, description=ubuntu-images-us/ubuntu-lucid-10.04-amd64-server-20101020.manifest.xml], userMetadata={}, state=RUNNING, privateAddresses=[10.119.22.224], publicAddresses=[174.129.72.44], hardware=[id=c1.xlarge, providerId=c1.xlarge, name=c1.xlarge, processors=[[cores=8.0, speed=2.5]], ram=7168, volumes=[[id=null, type=LOCAL, size=10.0, device=/dev/sda1, durable=false, isBootDevice=true], [id=null, type=LOCAL, size=420.0, device=/dev/sdb, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdc, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdd, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sde, durable=false, isBootDevice=false]], supportsImage=is64Bit()]], [id=us-east-1/i-f134809d, providerId=i-f134809d, tag=hbase, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-da0cf8b3, os=[name=null, family=ubuntu, version=10.04, arch=paravirtual, is64Bit=true, description=ubuntu-images-us/ubuntu-lucid-10.04-amd64-server-20101020.manifest.xml], userMetadata={}, state=RUNNING, privateAddresses=[10.98.146.48], publicAddresses=[174.129.142.130], hardware=[id=c1.xlarge, providerId=c1.xlarge, name=c1.xlarge, processors=[[cores=8.0, speed=2.5]], ram=7168, volumes=[[id=null, type=LOCAL, size=10.0, device=/dev/sda1, durable=false, isBootDevice=true], [id=null, type=LOCAL, size=420.0, device=/dev/sdb, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdc, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sdd, durable=false, isBootDevice=false], [id=null, type=LOCAL, size=420.0, device=/dev/sde, durable=false, isBootDevice=false]], supportsImage=is64Bit()]]]
Authorizing firewall
Authorizing firewall
Authorizing firewall
Running configuration script
Configuration script run completed
Running configuration script
Configuration script run completed
Completed configuration of hbase
Web UI available at https://ec2-184-72-159-249.compute-1.amazonaws.com
Wrote Hadoop site file /Users/phil/.whirr/hbase/hadoop-site.xml
Wrote Hadoop proxy script /Users/phil/.whirr/hbase/hadoop-proxy.sh
Completed configuration of hbase
Hosts: ec2-184-72-159-249.compute-1.amazonaws.com:2181
Completed configuration of hbase
Web UI available at https://ec2-184-72-159-249.compute-1.amazonaws.com
Wrote HBase site file /Users/phil/.whirr/hbase/hbase-site.xml
Wrote HBase proxy script /Users/phil/.whirr/hbase/hbase-proxy.sh
Wrote instances file /Users/phil/.whirr/hbase/instances
Started cluster of 6 instances
Cluster{instances=[Instance{roles=[tt, hbase-regionserver, dn], publicAddress=/204.236.242.78, privateAddress=/10.195.6.143, id=us-east-1/i-f5348099}, Instance{roles=[tt, hbase-regionserver, dn], publicAddress=/174.129.72.44, privateAddress=/10.119.22.224, id=us-east-1/i-f9348095}, Instance{roles=[tt, hbase-regionserver, dn], publicAddress=/50.16.71.166, privateAddress=/10.98.33.250, id=us-east-1/i-fb348097}, Instance{roles=[jt, nn, zk, hbase-master], publicAddress=ec2-184-72-159-249.compute-1.amazonaws.com/184.72.159.249, privateAddress=/10.112.205.48, id=us-east-1/i-e134808d}, Instance{roles=[tt, hbase-regionserver, dn], publicAddress=/174.129.142.130, privateAddress=/10.98.146.48, id=us-east-1/i-f134809d}, Instance{roles=[tt, hbase-regionserver, dn], publicAddress=/50.16.154.13, privateAddress=/10.194.127.79, id=us-east-1/i-f734809b}], configuration={hbase.zookeeper.quorum=ec2-184-72-159-249.compute-1.amazonaws.com:2181}}
```

<a id="destroy"></a>

## Destroy!

At some point you will want to tear-down that cluster. Here is how you can do that.

```
bin/whirr destroy-cluster --config hbase-ec2.properties

Destroying hbase cluster
Cluster hbase destroyed
```

<a id="conclusion"></a>

## Conclusion

Congratulations! If you have followed through this example, then you now have your own HBase cluster running in the cloud. Now… what to do with that HBase cluster? [/](Stay tuned for a future posts).

## Comments

<div id="comments">
  <ol class="comment-list">
    <li id="comment-851" class="comment even thread-even depth-1 comment reader">
      <img alt="Lars George" src="https://1.gravatar.com/avatar/b66b4b8f23c553ed389edd73522d4cd1?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://cloudera.com">Lars George</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, January 25th, 2011, 6:09 am">January 25, 2011</abbr> at <abbr class="comment-time" title="Tuesday, January 25th, 2011, 6:09 am">6:09 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hi Phil, </p>
        <p>Great post! We are still working on a few kinks but once Whirr 0.3.0 is out and will be released with CDH it will be even easier to get going as no build is required (obviously).</p>
        <p>A few notes, you are missing the “tt” (i.e TaskTracker) in the explanation table for the cluster template. Also, just to reiterate, all of the above services share the same instance. So in your example you will start 1+5 EC2 server with the various services running on them. </p>
        <p>Finally, what is also cool is that Whirr creates a local “$HOME/.whirr//” directory, so in your example $HOME/.whirr/hbase/ which has various files to help you working with the cluster. One of those is the “hadoop-proxy.sh” (start it like “source $HOME/.whirr/hbase/hadoop-proxy.sh &amp;” to launch it in the background) which sets up the SOCKS proxy for you so that you can talk to the servers and access the web UI for the master and region servers.</p>
        <p>There is also a “instances” which lists nicely the servers in the cluster, their local and remote IPs and roles. It could be used by a script or program to parse the details and be able to communicate with the servers.</p>
        <p>Thanks again for writing this post!</p>
        <p>Regards,<br />
Lars</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-854" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Tuesday, January 25th, 2011, 9:16 am">January 25, 2011</abbr> at <abbr class="comment-time" title="Tuesday, January 25th, 2011, 9:16 am">9:16 am</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks for the comments Lars. Well spotted on that missing “tt”! I’ve also clarified the number of EC2 machines in use, as this was not clear.</p>
            <p>I’m looking forward to 0.3.0 of Whirr being released and the general progress of it. It is a fantastic tool for getting up and running in the cloud. Thanks to yourself and all the committers!</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-1017" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Lynn Monson" src="https://1.gravatar.com/avatar/33bbbf4053c9c66fb798c6314b524983?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://lmonson.com/">Lynn Monson</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, February 9th, 2011, 1:29 pm">February 9, 2011</abbr> at <abbr class="comment-time" title="Wednesday, February 9th, 2011, 1:29 pm">1:29 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Very helpful.  The only thing I tripped on was understanding how the local keypair file relates to Whirr.  In my case, I followed your tutorial from a clean machine, itself running on EC2.  Since that machine had no ~/.ssh/id_rsa file, Whirr failed.</p>
        <p>It’s still not completely clear to me how Whirr bootstraps the machines, but I did learn that eventually Whirr uploads your local public key to the bootstrapped instances.   So a simple invocation of ssh-keygen -t rsa got everything running.</p>
        <p>Thanks again for the time and effort spent on this tutorial.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-1046" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Eugene Koontz" src="https://0.gravatar.com/avatar/8c63eef99e2c33a10aed425610c37b97?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://cloudsecurity.trendmicro.com/">Eugene Koontz</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, February 11th, 2011, 12:00 pm">February 11, 2011</abbr> at <abbr class="comment-time" title="Friday, February 11th, 2011, 12:00 pm">12:00 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Thanks a lot for this post; I found that your instructions work great, but if you prefer to use git, you can do :</p>
        <p>git clone git://git.apache.org/whirr.git </p>
        <p>instead of:</p>
        <p>svn co https://svn.apache.org/repos/asf/incubator/whirr/trunk whirr-trunk</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-8467" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Tim" src="https://0.gravatar.com/avatar/065953e347e1601e26a0fbb310e93f61?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Tim</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, January 27th, 2012, 5:44 am">January 27, 2012</abbr> at <abbr class="comment-time" title="Friday, January 27th, 2012, 5:44 am">5:44 am</abbr>
      </div>
      <div class="comment-text">
        <p>Great instructions!<br />
I believe the /incubator is not needed in the svn URL</p>
        <p>When mine starts, it says:</p>
        <p>Completed configuration of hbase<br />
Web UI available at https://107.20.125.231<br />
Wrote HBase site file /Users/tim/.whirr/hbase/hbase-site.xml<br />
Wrote HBase proxy script /Users/tim/.whirr/hbase/hbase-proxy.sh<br />
Completed configuration of hbase role hadoop-datanode<br />
Completed configuration of hbase role hadoop-tasktracker<br />
Starting to run scripts on cluster for phase start on instances: us-east-1/i-116dab74<br />
Running start phase script on: us-east-1/i-116dab74<br />
start phase script run completed on: us-east-1/i-116dab74<br />
Successfully executed start script: [output=, error=, exitCode=0]<br />
Finished running start phase scripts on all cluster instances<br />
Started cluster of 4 instances</p>
        <p>but it seems only the zookeeper actually started up and there is no JT or HBase master:  </p>
        <p>tim@ip-10-79-37-92:~$ ps -ef | grep java<br />
root      3214     1  0 13:29 ?        00:00:00 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /usr/local/zookeeper-3.3.3/bin/../build/classes:/usr/local/zookeeper-3.3.3/bin/../build/lib/*.jar:/usr/local/zookeeper-3.3.3/bin/../zookeeper-3.3.3.jar:/usr/local/zookeeper-3.3.3/bin/../lib/log4j-1.2.15.jar:/usr/local/zookeeper-3.3.3/bin/../lib/jline-0.9.94.jar:/usr/local/zookeeper-3.3.3/bin/../src/java/lib/*.jar:/etc/zookeeper/conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /etc/zookeeper/conf/zoo.cfg</p>
        <p>All the slave node services started it seems.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-30563" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Kiko Aumond" src="https://0.gravatar.com/avatar/a329bdda562648b4d4a3fd8f679443e6?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Kiko Aumond</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, January 3rd, 2013, 9:20 am">January 3, 2013</abbr> at <abbr class="comment-time" title="Thursday, January 3rd, 2013, 9:20 am">9:20 am</abbr>
      </div>
      <div class="comment-text">
        <p>This is an extremely useful post, thank you so much for writing it!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

