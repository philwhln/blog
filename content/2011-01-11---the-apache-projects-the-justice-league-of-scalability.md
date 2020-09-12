---
title: "The Apache Projects – The Justice League Of Scalability"
date: "2011-01-11T15:48:00-08:00"
template: "post"
draft: false
slug: "the-apache-projects-the-justice-league-of-scalability"
category: "Data processing"
tags:
  - activemq
  - apache projects
  - apache software foundation
  - asf
  - cassandra
  - hadoop
  - hbase
  - high scalability
  - lucene
  - mahout
  - rabbitmq
  - solr
  - zeromq
  - zookeeper
description: "In this post I will define what I believe to be the most important projects within the Apache Projects for building scalable web sites and generally managing large volumes of data."
socialImage:
  publicURL: "/media/images/2011/01/apache_justice_league_sq.jpg"
---
<a href="https://www.flickr.com/photos/deskhiker/2636488860/" title="Captain America by DuckBrown, on Flickr"><img alt="Captain Apache" class="alignleft size-full wp-image-777" height="233" src="/media/images/2011/01/captain_apache.jpg" title="captain_apache" width="350"/></a>

In this post I will define what I believe to be the most important projects within the [Apache Projects](https://projects.apache.org/) for building scalable web sites and generally managing large volumes of data.

If you are not aware of the Apache Projects, then you should be. Do not mistake this for [The Apache Web (HTTPD) Server](https://httpd.apache.org/), which is just one of the many projects in the Apache Software Foundation. Each project is it’s own open-source movement and, while Java is often the language of choice, it may have been created in any language. 

I often see developers working on solutions that can be easily solved by using one the tools in the Apache Projects toolbox. The goal of this post is to raise awareness of these great pieces of open-source software. If this is not new to you, then hopefully it will be a good recap of some of the key projects for building scalable solutions.

>  
> __The Apache Software Foundation__  
> provides support for the Apache community of open-source software projects.
> 

You have probably heard of many of these projects, such as Cassandra or Hadoop, but maybe you did not realize that they all come under the same umbrella. This umbrella is known as the Apache Projects and is a great place to keep tabs on existing and new projects. It is also a gold seal of approval in the open-source world. I think of these projects, or tools, as the superheroes of building modern scalable websites. By day they are just a list of open-source projects listed on a rather dry and hard to navigate website at [https://www.apache.org](https://www.apache.org/). But by night… they do battle with some of the world’s most gnarly datasets. Terabytes and even petabytes are nothing to these guys. They are the nemesis of high throughput, the digg effect, and the dreaded query-per-second upper limit. They laugh in the face of limited resources. Ok, maybe I am getting carried away here. Let’s move on…

## Joining The Justice League

Prior to a project joining the Apache Projects it will first be accepted into the [Apache Incubator](https://incubator.apache.org/). For instance, [Deltacloud has recently been accepted into the Apache Incubator](https://watzmann.net/blog/2010/07/deltacloud-apache-incubator.html).

>  
> The Apache Incubator has two primary goals:
> 
> *   Ensure all donations are in accordance with the ASF legal standards
> *   Develop new communities that adhere to our guiding principles
> 
>  – [ Apache Incubator Website](https://incubator.apache.org)
> 

You can find a list all the projects currently in the Apache Incubator [here](https://incubator.apache.org/projects/index.html). 

## Our Heroes

Here is a list, in no particular order, of tools you will find in the Apache Software Foundation Projects.

*   [Apache Cassandra](/the-apache-projects-the-justice-league-of-scalability#apache-cassandra)
*   [Apache Hadoop](/the-apache-projects-the-justice-league-of-scalability#apache-hadoop)
*   [Apache HBase](/the-apache-projects-the-justice-league-of-scalability#apache-hbase)
*   [Apache ZooKeeper](/the-apache-projects-the-justice-league-of-scalability#apache-zookeeper)
*   [Apache Solr](/the-apache-projects-the-justice-league-of-scalability#apache-solr)
*   [Apache ActiveMQ](/the-apache-projects-the-justice-league-of-scalability#apache-activemq)
*   [Apache Mahout](/the-apache-projects-the-justice-league-of-scalability#apache-mahout)

### Kryptonite

While each have their major benefits, none is a “silver bullet” or a “golden hammer”. When you design software that scales or does any job very well, then you have to make certain choices. If you are not using the software for the task it is was designed for then you will obviously find it’s weakness. Understanding and balancing the strengths and weaknesses of various solutions will enable you to better design your scalable architecture. Just because Facebook uses Cassandra to do battle with it’s petabytes of data, does not necessarily mean it will be a good solution for your “simple” terabyte-wrestling architecture. What you do with the data is often more important than how much data you have. For instance, Facebook has decided that HBase is now a better solution for many of their needs. \[This is the cue for everyone to run to the other side of the boat\].

>  
> Kryptonite is one of the few things that can kill Superman.
> 
> The word kryptonite is also used in modern speech as a synonym for Achilles’ heel, the one weakness of an otherwise invulnerable hero.
> 
>  – [“Krytonite” by Wikipedia](https://en.wikipedia.org/wiki/Kryptonite)
> 

Now, let’s look in more detail at some of the projects that can fly faster than a speeding bullet and leap tall datasets is a single \[multi-server, distributed, parallelized, fault-tolerant, load-balanced and adequately redundant\] bound.

### Apache Cassandra

Cassandra was built by Facebook to hold it’s ridiculous volumes of data within it’s email system. It’s much a like distributed key-value store, but with a hierarchy. The model is very similar to most NoSQL databases. The [data-model](https://wiki.apache.org/cassandra/DataModel) consists of columns, column-families and super-columns. I will not go into detail here about the data-model, but there is a great intro (see “[WTF is a SuperColumn? An Intro to the Cassandra Data Model](https://arin.me/blog/wtf-is-a-supercolumn-cassandra-data-model)“) that you can read. 

Cassandra can handle fast writes and reads, but its Kryptonite is _consistency_. It takes time to make sure all the nodes serving the data have the same value. For this reason Facebook is now [moving away from Cassandra](https://www.facebook.com/note.php?note_id=454991608919#) for its new messaging system, to HBase. HBase is a NoSQL database built on-top of Hadoop. More on this below.

### Apache Hadoop

Apache Hadoop, son of [Apache Nutch](https://nutch.apache.org/) and later raised under the watchful eye of Yahoo, has since become an Apache Project. Nutch is an open-source web search project built on Lucene. The component of Nutch that became Hadoop gave Nutch it’s “web-scale”.

Hadoop’s goal was to manage large volumes of data on commodity hardware, such as thousands of desktop harddrives. Hadoop takes much of it’s design from a paper published by Google on their [Bigtable](https://labs.google.com/papers/bigtable.html). It stores data on the Hadoop Distributed File-System (a.k.a. “HDFS”) and manages the running of distributed Map-Reduce jobs. In a previous post I gave an [example using Ruby with Hadoop](/map-reduce-with-ruby-using-hadoop) to perform Map-Reduce jobs.

_Map-Reduce_ is a way to crunch large datasets using two simple algorithms (“map” and “reduce”). You write these algorithms specific to the data you are processing. Although the your map and reduce code can be [extremely simple](/map-reduce-with-ruby-using-hadoop#coding-your-map-and-reduce-scripts-in-ruby), it scales across the entire dataset using Hadoop. This applies even if you have petabytes of data across thousands of machines. Your resulting data can be found in a directory on you HDFS disk when the Map-Reduce job completes. Hadoop provides some great web-based tools for visualizing your cluster and monitoring the progress of any running jobs.

Hadoop deals with very large chunks of data. You can tell Hadoop to Map-Reduce across everything it finds under a specific directory within HDFS and then output the results to another directory within HDFS. Hadoop likes \[really really\] large files (many gigabytes) made from large chunks (eg. 128Mb), so it can stream through them quickly, without many disk-seeks and manage distributing the chunks effectively.

Hadoop’s Kryptonite would be it’s brute force. It is designed for churning through large volumes of data, rather than being real-time. A common use-case is to spool up data for a period of time (an hour) and then run your map-reduce jobs on that data. By doing this you can very efficiently process vasts amounts of data, but you would not have real-time results.

__Recommended Reading__  
[Hadoop: The Definitive Guide by Tom White](https://www.amazon.com/gp/product/1449389732?ie=UTF8&amp;tag=getafil-20&amp;linkCode=as2&amp;camp=1789&amp;creative=9325&amp;creativeASIN=1449389732)

<img alt="" border="0" height="1" src="https://www.assoc-amazon.com/e/ir?t=getafil-20&amp;l=as2&amp;o=1&amp;a=1449389732" style="border:none !important; margin:0px !important;" width="1"/>

### Apache HBase

Apache HBase is a NoSQL layer on-top of Hadoop that adds structure to your data. HBase uses [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead_logging) to manage writes, which are then merged down to HDFS. A client request is responded to as soon as the update is written to the write-ahead log and the change is made in memory. This means that updates are very fast. The read side is also fast, since data is stored on disk in key order. Subsequently, because data is stored on disk in key-order, scans across sequential keys are fast, due to the low number disk seeks required. Larger scans are not currently possible.

HBase’s Krytonite would be similar to most NoSQL databases out there. Many use-cases still benefit from using relational database and HBase is not a relational database.

__Look Out For This Book Coming Soon__  
[HBase: The Definitive Guide by Lars George (May 2011)](https://www.amazon.com/gp/product/1449396100?ie=UTF8&amp;tag=getafil-20&amp;linkCode=as2&amp;camp=1789&amp;creative=9325&amp;creativeASIN=1449396100)

<img alt="" border="0" height="1" src="https://www.assoc-amazon.com/e/ir?t=getafil-20&amp;l=as2&amp;o=1&amp;a=1449396100" style="border:none !important; margin:0px !important;" width="1"/>

  
Lars has an [excellent blog](https://www.larsgeorge.com/) that covers Hadoop and HBase thoroughly.

### Apache ZooKeeper

[Apache ZooKeeper](https://hadoop.apache.org/zookeeper/) is the janitor of our Justice League. It is being used more and more in scalable applications such as Apache HBase, Apache Solr (see below) and [Katta](https://katta.sourceforge.net/). It manages an application’s distributed needs, such as configuration, naming and synchronization. All these tasks are important when you have a large cluster with constantly failing disks, failing servers, replication and shifting roles between your nodes.

### Apache Solr

[Apache Solr](https://lucene.apache.org/solr/) is built on-top of one of my favorite Apache Projects, [Apache Lucene \[Java\]](https://lucene.apache.org/java/docs/index.html). Lucene is a powerful search engine API written in Java. I have built large distributed search engines with Lucene and have been very happy with the results.

_Solr_ packages up Lucene as a product that can be used stand-alone. It provides various ways to interface with the search engine, such as via XML or JSON requests. Therefore, Java knowledge is not a requirement for using it. It adds a layer to Lucene that makes it more easily scale across a cluster of machines.

### Apache ActiveMQ

[Apache ActiveMQ](https://activemq.apache.org/) is a messaging queue much like [RabbitMQ](https://www.rabbitmq.com/) or [ZeroMQ](https://www.zeromq.org/). I mention ActiveMQ because I has used it, but it is definitely worth checking out the alternatives.

A _message queue_ is way to quickly collect data, funnel the data through your system and use the same information for multiple services. This provides separation, within your architecture, between collecting data and using it. Data can be entered into different queues (data streams). Different clients can subscribe to these queues and use the data as they wish.

ActiveMQ has two types of queue, “queue” and “topic”.

The queue type “queue” means that each piece of data on the queue can only be read once. If client “A” reads a piece of data off the queue then client “B” cannot read it, but can read the next item on the queue. This is a good way of dividing up data across a cluster. All the clients in the cluster will take a share of the data and process it, but the whole dataset will only be processed once. Faster clients will take a larger share of the data and slow clients will not hold-up the queue.

A “topic” means that each client subscribed will see all the data, regardless of what the other clients do. This is useful if you have different services all requiring the same dataset. It can be collected and managed once by ActiveMQ, but utilized by multiple processors. Slow clients can cause this type of queue to back-up.

If you are interested in messaging queues then I suggest checking out these [Message Queue Evaluation Notes](https://wiki.secondlife.com/wiki/Message_Queue_Evaluation_Notes/) by [SecondLife](https://secondlife.com/), who are heavy users of messaging queues.

### Apache Mahout

The son of Lucene and now a Hadoop side-kick, [Apache Mahout](https://mahout.apache.org/) was born to be an intelligence engine. From the Hindi word for “elephant driver” (Hadoop being the elephant), Mahout has grown into a top-level Apache Project in it’s own right, mastering the art of Artificial Intelligence on large datasets. While Hadoop can tackle the more heavy-weight datasets on it’s own, more cunning datasets require a little more algorithmic manipulation. Much of the focus of Mahout is on large datasets using Map-Reduce on-top of Hadoop, but the code-base is optimized to run well on non-distributed datasets as well.

## We appreciate your comments

If you found this blog post useful then please leave a comment below. I would like to hear which other Apache Projects you think deserve more attention and if you have ever been saved, like Lois Lane, by one of the above.

## Resources

*   [Apache Software Foundation](https://www.apache.org/)
*   [Apache Projects](https://projects.apache.org/)
*   [Apache Incubator](https://incubator.apache.org/)
*   [Justice League on Wikipedia](https://en.wikipedia.org/wiki/Justice_League)

<div id="comments">
  <h3 id="comments-number" class="comments-header">8 responses to “The Apache Projects – The Justice League Of Scalability”</h3>
  <ol class="comment-list">
    <li id="comment-671" class="comment even thread-even depth-1 comment reader">
      <img alt="Allen Wittenauer" src="https://1.gravatar.com/avatar/f9ba9fc53b19911b7c27822084eb3ffb?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Allen Wittenauer</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, January 11th, 2011, 6:55 pm">January 11, 2011</abbr> at <abbr class="comment-time" title="Tuesday, January 11th, 2011, 6:55 pm">6:55 pm</abbr>
      </div>
      <div class="comment-text">
        <p>A slight correction.</p>
        <p>Apache Hadoop came from Apache Lucene.  Yahoo! joined the already-in-progress project, adding some much needed know-how around scale.  So great are their contributions, that it is a common misconception that Yahoo! started the project.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-677" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, January 12th, 2011, 10:33 am">January 12, 2011</abbr> at <abbr class="comment-time" title="Wednesday, January 12th, 2011, 10:33 am">10:33 am</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks for the clarification, Allen. Based on your comments and Doug Cutting’s “Hadoop: a brief history” https://research.yahoo.com/files/cutting.pdf I have updated the text.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-673" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Hector" src="https://0.gravatar.com/avatar/88946afe1cfd908886ea1f44673b5cb0?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Hector</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, January 12th, 2011, 1:56 am">January 12, 2011</abbr> at <abbr class="comment-time" title="Wednesday, January 12th, 2011, 1:56 am">1:56 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hi. Facebook only uses cassandra in the inbox feature, so whatever the effect of eventual consistency on cassandra, it would only be noticeable there. Also, they are not moving away from it (as it is still being used). They decided it wasn’t the right fit for the new functionality they were planning.</p>
        <p>Regards</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-678" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, January 12th, 2011, 10:39 am">January 12, 2011</abbr> at <abbr class="comment-time" title="Wednesday, January 12th, 2011, 10:39 am">10:39 am</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks for those two corrections, Hector. Do you have any more information on what Facebook is now using Cassandra for?</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-687" class="comment even thread-even depth-1 comment reader">
      <img alt="Karthik Shiraly" src="https://0.gravatar.com/avatar/ca30d3abf6659128e83af2c8a8ec70c3?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Karthik Shiraly</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, January 13th, 2011, 9:17 am">January 13, 2011</abbr> at <abbr class="comment-time" title="Thursday, January 13th, 2011, 9:17 am">9:17 am</abbr>
      </div>
      <div class="comment-text">
        <p>Thanks for a great article. I’m very interested in high scalability technologies, and your article gives a real useful overview.</p>
        <p>Both Apache Solr and ActiveMQ have helped reduce development time on my projects. I’d especially like to highlight the fantastic faceted search feature of Solr – that’s thousands of DB ‘select count’ queries saved there!</p>
        <p>The Apache Hive project also deserves a mention in your list – it adds a query language layer of abstraction over hadoop, that makes writing map reduce jobs for data analytics easier and intuitive.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-692" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, January 13th, 2011, 10:55 am">January 13, 2011</abbr> at <abbr class="comment-time" title="Thursday, January 13th, 2011, 10:55 am">10:55 am</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks for the nice comments, Karthik. I missed Hive. I’ll have to give Hive some love in a future post. I agree with you on Solr / Lucene. It can be a great alternative (or compliment) to traditional databases and even (yes <em>even</em>) “NoSQL” technologies. Use-case dependent obviously.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-689" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Otis Gospodnetic" src="https://0.gravatar.com/avatar/a07c686baf62e1dfd53a16a1096a2256?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://blog.sematext.com/">Otis Gospodnetic</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, January 13th, 2011, 10:18 am">January 13, 2011</abbr> at <abbr class="comment-time" title="Thursday, January 13th, 2011, 10:18 am">10:18 am</abbr>
      </div>
      <div class="comment-text">
        <p>Actually, I’d say Hadoop came from Apache Nutch, where the HDFS predecessor was first created.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-691" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, January 13th, 2011, 10:48 am">January 13, 2011</abbr> at <abbr class="comment-time" title="Thursday, January 13th, 2011, 10:48 am">10:48 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Otis,</p>
            <p>Thanks for the comment!</p>
            <p>I updated the text yesterday for Hadoop. Does this cover roughly what you were saying?</p>
            <blockquote>
              <p>Apache Hadoop, son of Apache Nutch and later raised under the watchful eye of Yahoo, has since become an Apache Project. Nutch is an open-source web search project built on Lucene. The component of Nutch that became Hadoop gave Nutch it’s “web-scale”.</p>
            </blockquote>
            <p>I’ll try and squeeze a mention of HDFS there too.</p>
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

