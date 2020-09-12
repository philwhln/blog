---
title: "Quickly Launch A Cassandra Cluster On Amazon EC2"
date: "2011-01-21T18:58:00-08:00"
template: "post"
draft: false
slug: "quickly-launch-a-cassandra-cluster-on-amazon-ec2"
category: "DevOps"
tags:
  - amazon ec2
  - cassandra
  - cluster
  - homebrew
  - mac os x
  - nosql
  - whirr
description: "If you have read my previous post, Map-Reduce With Ruby Using Hadoop, then you will know that firing up a Hadoop cluster is really simple when you use"
socialImage:
  publicURL: "/media/images/2011/01/launch_cassandra_amazon_ec2_sq.jpg"
---

If you have read my previous post, “[Map-Reduce With Ruby Using Hadoop](/map-reduce-with-ruby-using-hadoop)“, then you will know that firing up a Hadoop cluster is really simple when you use Whirr. Without even ssh’ing on the machines in the cloud you can start-up your cluster and interact with it. In this post I’ll show you that it is just as easy to fire up a Cassandra cluster on Amazon EC2.

## Install Whirr

I will fly through the setup of Whirr quite quickly. All the commands you need are here, but if you want a more thorough explanation then see my other post, “[Map-Reduce With Ruby Using Hadoop](/map-reduce-with-ruby-using-hadoop)“.

I am assuming that you have [Homebrew installed](/homebrew-intro-to-the-mac-os-x-package-installer).

```
sudo brew update
sudo brew install maven
mkdir ~/src/cloudera
cd ~/src/cloudera
wget https://archive.cloudera.com/cdh/3/whirr-0.1.0+23.tar.gz
tar -xvzf whirr-0.1.0+23.tar.gz
cd whirr-0.1.0+23
mvn clean install
mvn package -Ppackage

```

Be patient with the above. There is a lot to install, so it will take some time. Maven installs a lot of dependencies if it is your first time using it.

The good news is that from here on you are setup to easily fire-up your Amazon EC2 cluster for Cassandra, or [Hadoop](/map-reduce-with-ruby-using-hadoop) if you choose.

## Whirr Configuration File

We will need to make a configuration file for Whirr to tell it that we want to launch a Cassandra cluster with 3 nodes. If you are brave, patient and have the cash, then you could just as easily fire-up a 100 node cluster (leave a comment if you do – there may be prizes!).

You will need to create a _cassandra.properties_ file with the following contents…

```
whirr.service-name=cassandra
whirr.cluster-name=mycassandracluster
whirr.instance-templates=3 cassandra
whirr.provider=ec2
whirr.identity=<YOUR_AMAZON_EC2_ACCESS_KEY_ID_GOES_HERE>
whirr.credential=<YOUR_AMAZON_EC2_SECRET_ACCESS_KEY_GOES_HERE>
whirr.private-key-file=${sys:user.home}/.ssh/id_rsa
```

Replace the obvious fields with your Amazon EC2 Access Key ID and Amazon EC2 Secret Access Key.

## Launch Your Cluster

Now you are ready to fire-up your Cassandra cluster. Simply use the following command and then be prepared to wait 5-10 minutes while Amazon builds your machines. This time is variable. Sometimes Amazon is quick, sometimes not so quick.

```
bin/whirr launch-cluster --config cassandra.properties


Launching mycassandracluster cluster
Configuring template
Starting 3 node(s)
Nodes started: [[id=us-east-1/i-13f25e7f, providerId=i-13f25e7f, tag=mycassandracluster, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-74f0061d, os=[name=null, family=amzn-linux, version=2010.11.1-beta, arch=paravirtual, is64Bit=true, description=amazon/amzn-ami-2010.11.1-beta.x86_64-ebs], userMetadata={}, state=RUNNING, privateAddresses=[10.204.99.163], publicAddresses=[50.16.155.106], hardware=[id=t1.micro, providerId=t1.micro, name=t1.micro, processors=[[cores=1.0, speed=1.0]], ram=630, volumes=[[id=vol-1657d47e, type=SAN, size=null, device=/dev/sda1, durable=true, isBootDevice=true]], supportsImage=hasRootDeviceType(ebs)]], [id=us-east-1/i-17f25e7b, providerId=i-17f25e7b, tag=mycassandracluster, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-74f0061d, os=[name=null, family=amzn-linux, version=2010.11.1-beta, arch=paravirtual, is64Bit=true, description=amazon/amzn-ami-2010.11.1-beta.x86_64-ebs], userMetadata={}, state=RUNNING, privateAddresses=[10.117.43.129], publicAddresses=[50.16.85.79], hardware=[id=t1.micro, providerId=t1.micro, name=t1.micro, processors=[[cores=1.0, speed=1.0]], ram=630, volumes=[[id=vol-1457d47c, type=SAN, size=null, device=/dev/sda1, durable=true, isBootDevice=true]], supportsImage=hasRootDeviceType(ebs)]], [id=us-east-1/i-11f25e7d, providerId=i-11f25e7d, tag=mycassandracluster, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-74f0061d, os=[name=null, family=amzn-linux, version=2010.11.1-beta, arch=paravirtual, is64Bit=true, description=amazon/amzn-ami-2010.11.1-beta.x86_64-ebs], userMetadata={}, state=RUNNING, privateAddresses=[10.117.46.170], publicAddresses=[184.73.100.203], hardware=[id=t1.micro, providerId=t1.micro, name=t1.micro, processors=[[cores=1.0, speed=1.0]], ram=630, volumes=[[id=vol-e857d480, type=SAN, size=null, device=/dev/sda1, durable=true, isBootDevice=true]], supportsImage=hasRootDeviceType(ebs)]]]
Authorizing firewall
Running configuration script
Completed launch of mycassandracluster
Started cluster of 3 instances
Cluster{instances=[Instance{roles=[cassandra], publicAddress=/50.16.85.79, privateAddress=/10.117.43.129}, Instance{roles=[cassandra], publicAddress=/50.16.155.106, privateAddress=/10.204.99.163}, Instance{roles=[cassandra], publicAddress=/184.73.100.203, privateAddress=/10.117.46.170}], configuration={}}


```

You now have your very own Cassandra cluster running in the cloud. Not so hard, hey!

## Connect From Ruby

I will be following this post with step-by-step guide on how you can interact with your new cluster from your Ruby On Rails application. I recommend [subscribing to the RSS feed](/feed) to get updates to the blog.

## Shutdown The Cluster

Here is how you can shutdown your cluster.

```
bin/whirr destroy-cluster --config cassandra.properties

Destroying mycassandracluster cluster
Cluster mycassandracluster destroyed
```

## Conclusion

Whirr makes it very easy to start and stop a Cassandra cluster in the cloud without leaving the comfort of your laptop. What you do with that cluster is up to you, but I will be give you some ideas of what you could do in future posts.

<div id="comments">
  <h3 id="comments-number" class="comments-header">4 responses to “Quickly Launch A Cassandra Cluster On Amazon EC2”</h3>
  <ol class="comment-list">
    <li id="comment-1141" class="comment even thread-even depth-1 comment reader">
      <img alt="Kevin Rood" src="https://0.gravatar.com/avatar/0aae2c417490c285d978499f04292a9c?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Kevin Rood</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, February 18th, 2011, 9:45 pm">February 18, 2011</abbr> at <abbr class="comment-time" title="Friday, February 18th, 2011, 9:45 pm">9:45 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Thanks for the excellent post.  I tried setting up the cluster and while it did create three ec2 instances, Cassandra was not started on any of them.  The version installed was 6.x as well.  Do you have any further details on how to make sure Cassandra starts on each node?  Also, I would like to completely control which version of Cassandra gets installed on the nodes.  Any thoughts about how to do that?  Ideally I would like to control what base version of Linux is used as well.</p>
        <p>Thanks!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-19471" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="rashm" src="https://0.gravatar.com/avatar/e0385e24dcd5867ac073a99d43c70ab2?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">rashm</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, July 21st, 2012, 3:50 pm">July 21, 2012</abbr> at <abbr class="comment-time" title="Saturday, July 21st, 2012, 3:50 pm">3:50 pm</abbr>
      </div>
      <div class="comment-text">
        <p>HI,</p>
        <p>Where do we specify hostname or ipaddress of cluster machine when installing cluster using whirr?</p>
        <p>Can we install hadoop stable version or hadoop-2.0.0-alpha version using whirr</p>
        <p>Should we use whirr for cluster installation at production?</p>
        <p>regards,<br />
rashmi</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-28187" class="comment even thread-even depth-1 comment reader">
      <img alt="Chandra Shekher Gupta" src="https://1.gravatar.com/avatar/b4afd8bb270abc9eb60379433fa91df3?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://bigdataconsultancy.wordpress.com/">Chandra Shekher Gupta</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, December 6th, 2012, 9:32 am">December 6, 2012</abbr> at <abbr class="comment-time" title="Thursday, December 6th, 2012, 9:32 am">9:32 am</abbr>
      </div>
      <div class="comment-text">
        <p>Few Question.</p>
        <p>* If i setup Cassandra in EC2 cloud, I will be charged for using the cloud?<br />
* I tried to register in Amazon cloud , they says it’s free but ask for credit card details, and i am sure  they would be charging for bandwidth usages.</p>
        <p>Thanks Chandra</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-28189" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, December 6th, 2012, 10:12 am">December 6, 2012</abbr> at <abbr class="comment-time" title="Thursday, December 6th, 2012, 10:12 am">10:12 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Chandra,</p>
            <p>You will be charged for any resources you use on Amazon EC2. They do have a “free tier”, but you will unlikely be able to set up a cluster on this. See https://aws.amazon.com/free/faqs/</p>
            <p>Thanks,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

