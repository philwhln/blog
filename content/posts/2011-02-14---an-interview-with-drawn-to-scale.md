---
title: "Other People’s Data – An Interview With “Drawn To Scale”"
date: "2011-02-14T11:13:00-08:00"
template: "post"
draft: false
slug: "an-interview-with-drawn-to-scale"
category: "Interview"
tags:
  - big data
  - bradford stephens
  - business intelligence
  - cloud-computing
  - data management
  - data processing
  - drawn to scale
  - ec2
  - gaming
  - hadoop
  - hbase
  - high scalability
  - iaas
  - media
  - paas
  - rackspace
  - social networks
  - spire
  - startup
description: "In this blog-post Bradford Stephens, Drawn To Scale's founder, answers a series of technical, business and personal questions to give an overview of what Drawn To Scale is and where it is going. Who are the founders? What is their background, technology and business model? How were they going to manage other people's big data? Can one tool fit the demands from a broad range of data challenges that different businesses are seeing?"
socialImage:
  publicURL: "/media/images/2011/02/drawn_to_scale_sq.jpg"
---
<img alt="Drawn To Scale" height="211" src="https://commondatastorage.googleapis.com/philwhln/blog/media/images/drawn_to_scale/drawn_to_scale.png" style="border: none" width="408"/>

Early this year I came across a new start-up called “[Drawn To Scale](https://drawntoscalehq.com/)“. Their [website](https://drawntoscalehq.com/) had just enough information to peak my interest, but not enough to answer my questions of who, what and how. Who are the founders? What is their background, technology and business model? How were they going to manage other people’s big data? Can one tool fit the demands from a broad range of data challenges that different businesses are seeing? 

In this blog-post [Bradford Stephens](https://twitter.com/#!/lusciouspear), Drawn To Scale’s founder, answers a series of technical, business and personal questions to give an overview of what Drawn To Scale is and where it is going.

## The Interview

<div class="interview">
<h4>What is “Drawn To Scale”?</h4>
<p><a href="https://drawntoscalehq.com/index.html">Drawn to Scale</a> is the company who created <a href="https://drawntoscalehq.com/whyspire.html">Spire</a>, a distributed database with real-time queries and fulltext search, that’s also simple to manage. Our unique distributed indexing technology allows far more fast and efficient scalability than any other database for this purpose.</p>
<p>Since we’re built for the web application era, we have native <a href="https://www.json.org/">JSON</a> serialization on top of our columnar store, so integration into things like <a href="https://jquery.com/">jQuery</a> is effortless. Users don’t need a large complex <a href="https://en.wikipedia.org/wiki/Object-relational_mapping">ORM</a> to use Spire.</p>
<p>Databases such as <a href="https://hbase.apache.org/">HBase</a> or <a href="https://cassandra.apache.org/">Cassandra</a> only offer rudimentary primary-key access to data, whereas projects like <a href="https://couchdb.apache.org/">CouchDB</a> and <a href="https://www.mongodb.org/">MongoDB</a> have rich functionality but are limited in scalability. We combine the best of both worlds.</p>
<p>Currently, Spire is available for direct use on-premises or on top of cloud infrastructure, but we’re investigating other ways of making our platform available as the technology evolves.</p>
<h4>I see that your tagline is “Big data for all”. How would you describe what Drawn To Scale does for its users?</h4>
<p>Spire allows users to build applications (mostly web applications) on top of a database platform without worrying about scaling to the amount of users or data. To handle larger amounts of data or more requests, one can simply add nodes without worrying about all the nightmares of distributed computing management. Companies won’t need to hunt for a team of distributed computing experts (of whom there aren’t very many) in order to <a href="/how-to-get-experience-working-with-large-datasets">work with Big Data</a>.</p>
<h4>What stage is the business at today?</h4>
<p>We have been around about a year, and have a paying beta customer, with more on the way. We have several employees from well-respected backgrounds in distributed systems and infrastructure. Drawn to Scale is currently self-funded.</p>
<h4>When will Drawn To Scale be production ready?</h4>
<p>End of <a href="https://en.wikipedia.org/wiki/Second_quarter_of_a_calendar_year#Subdivisions">Q2</a>, this year.</p>
<h4>Who do you see as your target customers?</h4>
<p>There’s a few interesting verticals:</p>
<p><strong>Social Networks / Media / Gaming / <a href="https://en.wikipedia.org/wiki/Business_intelligence">BI (Business Intelligence)</a> / Analytics</strong><br/>
Anything in the social space usually has a combination of Big Data and Big Users. This is perfect for Spire.</p>
<p><strong>Mobile Phones / Sensor Networks</strong><br/>
Again — lots of users, lots of data. From mobile applications to monitoring the network infrastructure itself. Even power meter monitoring on the smartgrid is a “big data” problem.</p>
<p><strong>Advertising Networks</strong><br/>
Better targeting and faster analytics in the advertising world has always been needed, and we accelerate this.</p>
<h4>Are you doing or planning to do a beta sign-up or any trials with select customers?</h4>
<p>Yes, we are in a private beta with select customers right now. We have space for a few more, so we’d love to <a href="https://drawntoscalehq.com/contact.html">talk to anyone</a> interested!</p>
<h4>Who do you see as your biggest competitors in this space?</h4>
<p>By far, the biggest competitor is companies trying to build their own purpose-driven database in-house. This usually has many unexpected problems. Distributed systems are not straightforward, nor easy.</p>
<p>What’s interesting is that we usually see 3 things happen to companies when they hit “big data” problems:</p>
<p>1. They attempt to build a sharded SQL or Document database, with all the usual problems (querying every node, failover, imperfect data balancing, etc.)</p>
<p>2. Try to build their own distributed DB from scratch or force functionality on top of <a href="https://en.wikipedia.org/wiki/NoSQL#Key.2Fvalue_store_on_disk">KV-stores</a> like <a href="https://cassandra.apache.org/">Cassandra</a>, or <a href="https://labs.google.com/papers/bigtable.html">BigTable</a> clones like <a href="https://hbase.apache.org/">HBase</a>.</p>
<p>3. The scariest thing we’ve seen is that companies have to change their business model because their database infrastructure can’t handle the amount of data or users they’re getting. This is not as rare as you think.</p>
<h4>What do you see as your key differentiator or unique-selling-point from other solutions?</h4>
<p>Besides some internal Google tools (and Megastore), we’re the only DB with a <a href="https://en.wikipedia.org/wiki/Index_(search_engine)#Challenges_in_Parallelism">distributed index</a> and <a href="https://en.wikipedia.org/wiki/Full_text_search">fulltext search</a>. This means you can keep query latency to “real-time” while adding more data and users. We also are an end-to-end database platform. You can do everything from batch processing to real-time storage without having to glue dozens of open source projects together.</p>
<h4>How are your products priced?</h4>
<p>We’re pretty flexible. We do standard, straightforward per-cluster licences, or OEM shared-revenue licenses.</p>
<h4>How are you ensuring that customers trust in your solution enough to commit to building their own products on top of it?</h4>
<p>One of the best things about Spire is that we use <a href="https://hadoop.apache.org/">Hadoop</a> and <a href="https://hbase.apache.org/">HBase</a> as a storage layer, which are very well <a href="https://www.cloudera.com/products-services/enterprise/">enterprise-proven</a>. So customers have little risk of losing data from an unproven platform.</p>
<h4>Tell me about the founders and any other people behind the scenes at Drawn-To-Scale?</h4>
<p>I, <a href="https://twitter.com/#!/lusciouspear">Bradford Stephens</a>, am the primary founder of Drawn to Scale. I have a background in Computer Science and Politics. I worked on SQL Server at Microsoft and did consulting, before landing at a Social Media BI startup, Visible Technologies. As the lead platform engineer at <a href="https://www.visibletechnologies.com/">Visible Technologies</a>, I was responsible for creating a “Google-Scale”<br/>
infrastructure for real-time querying and analytics on top of Social Media such as blogs and tweets. Besides being the author of popular scalability blog <a href="https://www.roadtofailure.com/">roadtofailure.com</a>, I am the co-chair of the <a href="https://www.oscon.com/oscon2011">OSCON Data conference</a>, and <a href="https://events.gigaom.com/bigdata/">GigaOm Structure Big Data</a> [conference].</p>
<p>We have a fascinating advisory team as well: <a href="https://twitter.com/#!/ryanobjc">Ryan Rawson</a> (Amazon, Google, Stumbleupon), <a href="https://www.crunchbase.com/person/bradford-cross">Bradford Cross</a> (Flightcaster, Woven), <a href="https://twitter.com/#!/rhm2k">Rich Miller</a> (well-known cloud / tech CEO), and a few others on the way.</p>
<h4>How fast are your “low-latency queries”?</h4>
<p>Depends on size of data, number of requests, hardware, etc. We aim for milliseconds to seconds.</p>
<h4>How did you achieve this performance and what technical challenges did you encounter?</h4>
<p><strong>Peformance</strong><br/>
The Spire Distributed Indexing Engine is one key to scalability and performance. A lot of research has gone into this. Spire only needs to query the few machines that actually have relevant data. If you try to shard MySQL or document stores, what you end up with is a partitioned set of data on many machines. These systems all have non-distributed indexes, so doing queries is almost always a worst-case scenario. If you wanted to find “<em>All documents with AuthorName = Bob</em>“, and you have 20 nodes, you’ll need to query <em>every one</em> for each query, which completely kills throughput and latency. Our index allows a “universal” view, so finding <em>“All documents with AuthorName = Bob”</em> is a simple request to a single node.</p>
<p><strong>Distributed index</strong><br/>
Major challenges (that will always need tuning) are the <a href="https://redstack.wordpress.com/2011/01/06/visualising-garbage-collection-in-the-jvm/">JVM and Garbage Collection</a>, as well as building very strong fault handling mechanisms. <a href="https://hadoop.apache.org/">Hadoop</a> especially has a history of vague failure modes. We feel it is far more important to be bulletproof in reality than elegant in theory, which is why we’re fans of the very well-proven <a href="https://labs.google.com/papers/bigtable.html">BigTable</a> model from Google.</p>
<h4>You mention “full-text search” in your white-paper. Can you go into more detail about what kinds of searches you can do?</h4>
<p>We can do faceted, multi-lingual searches, because we implement <a href="https://lucene.apache.org/java/3_0_1/api/core/org/apache/lucene/analysis/Analyzer.html">Lucene analyzers</a> (but we don’t use <a href="https://wiki.apache.org/lucene-java/FrontPage">Lucene</a> as the index itself). This makes replacing existing <a href="https://lucene.apache.org/solr/">Solr</a> / Lucene clusters straightforward.</p>
<h4>I see you are using <a href="https://hbase.apache.org/">HBase</a>, <a href="https://lucene.apache.org/">Lucene</a> and <a href="https://www.opscode.com/chef/">Chef</a>. Are there any other critical components to you system?</h4>
<p>These are the major components, along with open-source serialization libraries, performance testers (<a href="https://grinder.sourceforge.net/">Grinder</a>), and minor OSS (open-source software) stuff.</p>
<h4>Configuration of the cluster is managed by configuration server. Is that right?</h4>
<p>Yes, configuration is managed by a replication <a href="https://www.opscode.com/chef/">Chef</a> configuration server (backed by <a href="https://couchdb.apache.org/">CouchDB</a>). This makes the job of Operations much easier, because we feel managing dozens of nodes manually is a massive barrier to adoption for distributed databases.</p>
<h4>Does each customer have it own cluster of machines (or virtual machines) and their own configuration server?</h4>
<p>Yes, we’re not a <a href="https://en.wikipedia.org/wiki/Platform_as_a_service">Platform-As-A-Service</a>, so right now we focus on running in datacenters or on <a href="https://en.wikipedia.org/wiki/Infrastructure_as_a_service#Infrastructure">IaaS</a> (Infrastructure-As-A-Service) vendors (EC2, Rackspace Cloud, etc.)</p>
<h4> Is this a hosted-only solution? Can customers install your product within their own network?</h4>
<p>We run in both the datacenter and the cloud, we are not self-hosted.</p>
<h4>Is there an amusing stories of how you can up with the name Drawn-To-Scale?</h4>
<p>Bradford’s wife, Myra, is the Lead Chemist at Nintendo (and does product safety testing.) She’s the creative one of the family, and basically came up with it after a brainstorming session. She’s named all of my projects, musical compositions, etc (except for the database itself, Spire). The previous name for the project was “BigSearch” or “BigQuery”, which were rather insipid.</p>
<p>Other humorous stories are classified for a few years <img alt=";)" class="wp-smiley" src="/media/images/smilies/icon_wink.gif"/> </p>
</div>

<script type="text/javascript">&lt;!--
google_ad_client = "pub-5260338827095335";
/* philwhln post end ldrbrd 728x90 */
google_ad_slot = "7456241966";
google_ad_width = 728;
google_ad_height = 90;
//--&gt;
</script>

  

<script src="https://pagead2.googlesyndication.com/pagead/show_ads.js" type="text/javascript">
</script>

## Conclusion

Since I first contacted Drawn To Scale, they have been [featured on the O’Reilly Radar](https://radar.oreilly.com/2011/01/bradford-stephens-drawn-to-scale.html) and their website now provides more information on what Drawn To Scale is.

Whilst they are still at the early stages and are finding their feet with their first customers, Bradford’s background and understanding of the technology show good promise for a successful startup and a good solution in this big data space.

## Resources

*   [Video interview on the O’Reilly Radar](https://radar.oreilly.com/2011/01/bradford-stephens-drawn-to-scale.html)
*   [“How to Decrease the Pain in Building Distributed Systems” – presentation by Bradford Stephens]()
*   [Drawn To Scale on Crunchbase](https://www.crunchbase.com/company/drawn-to-scale)
*   [Bradford Stephens on Twitter](https://twitter.com/#!/lusciouspear)
*   [Nick Dimiduk on Twitter](https://twitter.com/#!/xefyr)
*   [“Pig, Making Hadoop Easy” – uploaded to Slideshare by Nick Dimiduk](https://www.slideshare.net/xefyr/pig-making-hadoop-easy)

<div class="hentry post publish post-2 even alt author-admin category-start-ups category-web-development post_tag-adam-dangelo post_tag-amazon-ec2 post_tag-aws post_tag-charlie-cheever post_tag-comet post_tag-git post_tag-haproxy post_tag-livenode post_tag-long-polling post_tag-memcached post_tag-mysql post_tag-nginx post_tag-nosql post_tag-paste post_tag-pylons post_tag-quora post_tag-quora-com post_tag-search-box post_tag-steve-souders post_tag-technology post_tag-thrift post_tag-tornado post_tag-ubuntu-linux post_tag-webnode2 post_tag-webscale" id="post-1120">
<h2 class="entry-title"><a href="/quoras-technology-examined" rel="bookmark" title="Quora’s Technology Examined">Quora’s Technology Examined</a></h2>
<p class="byline"><span class="byline-prep byline-prep-author">By</span> <span class="author vcard"><span class="fn">Phil Whelan</span></span> <span class="byline-prep byline-prep-published">on</span> <abbr class="published" title="Monday, January 31st, 2011, 12:37 pm">January 31, 2011</abbr> </p>
<div style="float: left;"><img alt="quora_microscope_sq" class="attachment-post-thumbnail wp-post-image" height="180" src="/media/images/2011/01/quora_microscope_sq.jpg" title="quora_microscope_sq" width="180"/></div>
<div class="entry-content">
<p>In this blog post I will delve into the snippets of information available on Quora and look at Quora from a technical perspective. What technical decisions have they made? What does their architecture look like? What languages and frameworks do they use? How do they make that search bar respond so quickly?</p>
</div>
<p><!-- .entry-content --></p>
<div class="continue-reading"><a href="/quoras-technology-examined"><strong>Continue reading…</strong></a></div>
</div>

In this blog post I will delve into the snippets of information available on Quora and look at Quora from a technical perspective. What technical decisions have they made? What does their architecture look like? What languages and frameworks do they use? How do they make that search bar respond so quickly?

