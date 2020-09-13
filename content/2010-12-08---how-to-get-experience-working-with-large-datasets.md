---
title: "How To Get Experience Working With Large Datasets"
date: "2010-12-08T11:50:00-08:00"
template: "post"
draft: false
slug: "how-to-get-experience-working-with-large-datasets"
category: "Data processing"
tags:
  - amazon ec2
  - bbc
  - cassandra
  - data processing
  - dbpedia
  - ebs
  - freebase
  - hadoop
  - hdfs
  - large datasets
  - livedoor
  - lucene
  - mongodb
  - nosql
  - rackspace
  - solr
  - wikipedia
  - working with data
description: "There are data sources out there, but which data source you choose depends on which technology you wish to get experience working with. The experience should be of the technologies you are using, rather than what the data is. Certain datasets pair better with certain technologies. Simulating the data can be another approach. You just need a clever way of generating and randomizing your fake data. Thirdly, you can use a hybrid approach. Take real data and replay it on a loop, randomizing it as it goes through. Simulating the Twitter fire-hose should not be too hard, should it?"
socialImage:
  publicURL: "/media/images/2010/12/big_dataset_experience_sq.jpg"
---
I think I have been lucky that several of the projects I been worked on have exposed me to having to manage large volumes of data. The largest dataset was probably at [MailChannels](https://www.mailchannels.com), though [Livedoor.com](https://www.livedoor.com) also had some sizeable data for their books store and department store. Most of the pain with Livedoor’s data was from it being in Japanese. Other than that, it was pretty static. This was similar to the data I worked with at the BBC. You would be surprised at how much data can be involved with a single episode of a TV show. With any in-house generated data the update size and frequency is much less dramatic, even if the data is being regularly pumped in from 3rd parties.

__Those Humans__  
The real fun comes when the public (that’s you guys) are generating data to be pumped into the system. MailChannels’ work with email, which is human generated (lies! 95% is actually from spambots). Humans are unpredictable. They suddenly all get excited about the same thing at the same time, they are demanding, impatient, and smell funny. The latter will not be so much of problem to you, but the other points will if you intend to open your doors to user generated information.

__Need More Humans__  
Opening your doors will not necessarily bring you the large volumes of data you require. In the beginning it will be a bit draughty, and you might consider closing those doors. What you need is more of those human eyeballs looking through their glowing screens at your data collectors. Making that a reality is the hard part. If you do get to that point, then you will probably not have “getting experience with large datasets” at the fore-front of your mind. Like sex, working with large datasets is most important to those not working with large datasets. So we will look elsewhere.

__Data Hunting__  
The Internet is really just one giant bucket of data soup. Sure, we all just see the blonde, brunette or redhead, but it’s actually just green squiggly codes raining down in uniform vertical lines behind the scenes. What you need to figure out is how to get those streams of data that are flying around in the tubes of the Internet to be directed through your tube, into your MongoDB NoSQL database, Solr search engine, Hadoop distributed file-system or Cassandra cluster. 

>  
> “All I see now is blonde, brunette, redhead”  
> <small> – Cypher, The Matrix</small>
> 

__Data Sources__  
The trouble with most sources of data is that they are owned and the data is copyrighted or proprietary. You can scrape websites and, if you fly under the radar, you will get a dataset. Although, if you want a large dataset, then it will take a lot of scraping. Instead, you should look for data that you can acquire more efficiently and, hopefully, legally. If nothing else, collecting the data in a legal manner will help you sleep better at night and you have a chance at going on to use that data to build something useful.

<a href="https://www.flickr.com/photos/donshall/4660443389/" title="teenie • zach by origamidon, on Flickr"><img align="right" alt="teenie • zach" height="240" src="https://farm5.static.flickr.com/4030/4660443389_88514b09b3_m.jpg" width="240"/></a>

Here’s a list of places that have data available, provided by my good friend, [Geoff Webb](https://ca.linkedin.com/in/geoffreywebb).

*   <https://data.vancouver.ca>
*   <https://stats.oecd.org/index.aspx>
*   <https://data.un.org/Explorer.aspx>
*   <https://mdgs.un.org/unsd/mdg/Data.aspx>
*   <https://robjhyndman.com/TSDL>
*   <https://www.ngdc.noaa.gov/ngdc.html>
*   <https://www.data.gov>
*   <https://www.data.gov.uk>
*   <https://data.worldbank.org>

Some others I think are worth looking at are [Wikipedia](https://www.wikipedia.org/), [Freebase](https://www.freebase.com/) and [DBpedia](https://dbpedia.org/). Freebase pulls its data from Wikipedia on a regular basis, as well as from TVRage, Metacritic, Bloomberg and CorpWatch. DBpedia also pulls data from Wikipedia, as well as YAGO, Wordnet and other sources.

You can download a dump of the Freebase dataset [here](https://download.freebase.com/datadumps/). More information on the these dump files can be found [here](https://wiki.freebase.com/wiki/Data_dumps).

__Make It Yourself__  
The quickest way I’ve found to get a good feed of data is to generate it. Create an algorithm that simulates mass user sign-up, 10,000 tweets per second, or vast amounts of logging data. Generating timestamps is easy enough. Use a dictionary of words for the content. Download War And Peace, Alice In Wonderland or any other book that is now out of copyright if you need real strings of words for your fake tweets.

__Get Loopy__  
I have built a load-testing system, in which I recorded one days worth of data coming into the production system, then played it back at high-speed on a loop into my load-testing system. I could simulate a hundred times throughput or more. This could also be done with a smaller amount of data. If you can randomize the data a little so it’s slightly different each time around the loop, then even better.

__Play In The Clouds__  
I would recommend you do not download the data to your home or work machine. Fire up an Amazon EC2 machine and store you data on a EBS volume. You’ll be able to turn it on and off so that you are only paying for the machine when you have time to play with it. The bandwidth and speed you can download that data will be much quickly. If you want to scale the data across a small cluster of machines it will be easy to setup and tear-down. The data will be in the cloud, so if you do get an idea of what to do with it then it will not take you two weeks to upload that data from your home machine to the cloud. It will already be there. On Amazon EC2 you can play with the same data on a cluster of small machines with limited RAM, or on a “extra large” machine with up to 64Gb of RAM. You can try all these different configurations very quickly and inexpensively. I mention Amazon EC2, because that is where my experience is, but the same applies for Rackspace and other cloud infrastructure providers.

__Conclusion__  
There are data sources out there, but which data source you choose depends on which technology you wish to get experience working with. The experience should be of the technologies you are using, rather than what the data is. Certain datasets pair better with certain technologies. Simulating the data can be another approach. You just need a clever way of generating and randomizing your fake data. Thirdly, you can use a hybrid approach. Take real data and replay it on a loop, randomizing it as it goes through. Simulating the Twitter fire-hose should not be too hard, should it?

__Please Leave A Comment__  
If you have other links to sources of good datasets, have any code for simulating datasets or any ideas on this topic, then please leave a comment on this blog post. I will answer any questions as best as I can and intend to write much more on this topic in future blogposts. Your feedback will help guide the content of these posts.

__Related Posts On This Blog__

*   [Working With Large Data Sets](/?p=149)
*   [Data Mining Without Hadoop](/?p=238)

## Comments

<div id="comments">
  <ol class="comment-list">
    <li id="comment-245" class="comment even thread-even depth-1 comment reader">
      <img alt="Chris Hemedinger" src="https://0.gravatar.com/avatar/2f0476d7b3be190241c3537b3948d2fa?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://blogs.sas.com/sasdummy">Chris Hemedinger</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, December 10th, 2010, 5:47 am">December 10, 2010</abbr> at <abbr class="comment-time" title="Friday, December 10th, 2010, 5:47 am">5:47 am</abbr>
      </div>
      <div class="comment-text">
        <p>Another great source is Kiva.org, with tons of loan and lender data related to microfinance transactions.</p>
        <p>I’ve written about this at my blog for World Statistics Day (last month):<br />
https://blogs.sas.com/sasdummy/index.php?/archives/208-World-Statistics,-FTW!.html</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-247" class="comment byuser comment-author-admin bypostauthor odd alt thread-odd thread-alt depth-1 comment role-administrator user-admin entry-author">
      <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, December 10th, 2010, 10:33 am">December 10, 2010</abbr> at <abbr class="comment-time" title="Friday, December 10th, 2010, 10:33 am">10:33 am</abbr>
      </div>
      <div class="comment-text">
        <p>BTW, Pavan Yara left this comment on highscalability.com, where this post is also featured, and gave some really great data sources.</p>
        <p>The Physics arXiv Blog from Technology Review lists 70 online database sources:<br />
https://www.technologyreview.com/blog/arxiv/26097/</p>
        <p>Stanford’s SNAP library also offers a large collection of network datasets useful for variety of purposes:<br />
https://snap.stanford.edu/data/index.html</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-253" class="comment even thread-even depth-1 comment reader">
      <img alt="Jeremy Weiss" src="https://0.gravatar.com/avatar/e616898907a7c78714cd3abc0eda1a7a?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Jeremy Weiss</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, December 10th, 2010, 2:13 pm">December 10, 2010</abbr> at <abbr class="comment-time" title="Friday, December 10th, 2010, 2:13 pm">2:13 pm</abbr>
      </div>
      <div class="comment-text">
        <p>How large does a dataset have to be, before it’s considered ‘large’?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-255" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Friday, December 10th, 2010, 2:36 pm">December 10, 2010</abbr> at <abbr class="comment-time" title="Friday, December 10th, 2010, 2:36 pm">2:36 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Jeremy,</p>
            <p>Great question.</p>
            <p>While there is definitely no official standard on what a “large” dataset is and everything being relative, I’d say 1Tb is probably a good size.</p>
            <p>Generally, if you find it’s going to hard to process the data you have on a single machine and you start scratching your head and looking at technologies such as Hadoop, Cassandra, MongoDB and other NoSQL technologies then I’d call it “large”.</p>
            <p>For me, it’s generally about the throughput of data. I like playing with data that’s large, but also data that is coming into the system in serious volumes. The difficulties in handling all those requests and inability to write to disk quickly enough becomes interesting and leads you to have to understand clustering, NoSQL technologies, queuing (such as ActiveMQ and RabbitMQ) and make hard decisions about exactly what it is your trying to do with the data and how intend to access it.</p>
            <p>Obviously it’s difficult to find really large datasets and possibly hard justify where you’re realistically going encounter those scenarios in your career.</p>
            <p>Phil</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <li id="comment-338" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Brad" src="https://0.gravatar.com/avatar/ecb6e867287ff5b52ee44259faee63fc?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Brad</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, December 15th, 2010, 3:04 pm">December 15, 2010</abbr> at <abbr class="comment-time" title="Wednesday, December 15th, 2010, 3:04 pm">3:04 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Here’s another source:<br />
https://data.stackexchange.com/</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-339" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, December 15th, 2010, 3:19 pm">December 15, 2010</abbr> at <abbr class="comment-time" title="Wednesday, December 15th, 2010, 3:19 pm">3:19 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Brad. That’s a great resource. I see you can get a 2Gb torrent from ClearBits of this data.</p>
            <p>“Stack Overflow trilogy creative commons data dump, to start of Nov 2010. Includes – https://stackoverflow.com – https://serverfault.com – https://superuser.com – https://meta.stackoverflow.com – https://meta.serverfault.com – https://meta.superuser.com – https://stackapps.com And any other public (non-beta) website and its corresponding meta site at https://stackexchange.com/sites“</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-424" class="comment byuser comment-author-admin bypostauthor even thread-even depth-1 comment role-administrator user-admin entry-author">
      <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, December 22nd, 2010, 10:23 pm">December 22, 2010</abbr> at <abbr class="comment-time" title="Wednesday, December 22nd, 2010, 10:23 pm">10:23 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Andrew Clegg posted this comment on highscalability.com, where this blog post is also featured. Thank Andrew!<br />
https://highscalability.com/blog/2010/12/8/how-to-get-experience-working-with-large-datasets.html</p>
        <p>There’s dozens and dozens of life-sciences databases listed here, most of which can be downloaded for free:</p>
        <p>https://en.wikipedia.org/wiki/List_of_biological_databases</p>
        <p>Gene sequences, protein structures, interactions, functional annotations etc.</p>
        <p>If biology isn’t your thing, try:</p>
        <p>https://infochimps.com/</p>
        <p>https://mldata.org/</p>
        <p>https://www.kaggle.com/</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-1240" class="comment byuser comment-author-kencochrane odd alt thread-odd thread-alt depth-1 comment role-subscriber user-kencochrane">
      <img alt="kencochrane" src="https://1.gravatar.com/avatar/5b5661a3a443a2a4bece8674006bd6ef?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.google.com/profiles/103710366821159191618">kencochrane</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, March 2nd, 2011, 4:44 am">March 2, 2011</abbr> at <abbr class="comment-time" title="Wednesday, March 2nd, 2011, 4:44 am">4:44 am</abbr>
      </div>
      <div class="comment-text">
        <p>Another source I’m surprised someone didn’t mention yet is Amazon’s Public data sets: https://aws.amazon.com/datasets</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-6245" class="comment even thread-even depth-1 comment reader">
      <img alt="Yildirim Mungan" src="https://0.gravatar.com/avatar/aac670327a1f36a2de3cb6f37444298f?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Yildirim Mungan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, December 7th, 2011, 7:36 am">December 7, 2011</abbr> at <abbr class="comment-time" title="Wednesday, December 7th, 2011, 7:36 am">7:36 am</abbr>
      </div>
      <div class="comment-text">
        <p>Great blog Phil! I’m reading everything you put. Thank you very much indeed.<br />
Although I’ve searched google, I could not find a big sample data for telco CDR (Call Detail Records).<br />
I am planning to apply some analysis on CDR files, and provide a solution for telco big data analysis problem.<br />
Can you recommend a web address where I may find what I am looking for?</p>
        <p>Cheers!</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-6385" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, December 10th, 2011, 10:49 am">December 10, 2011</abbr> at <abbr class="comment-time" title="Saturday, December 10th, 2011, 10:49 am">10:49 am</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks for the nice comment on my blog, Yildirim!</p>
            <p>I do not have any info on the Telco CDR data that you’re looking for. I’d just Google it myself. You could try posting your question to some forums on the topic e.g. https://www.linkedin.com/groups/Big-Data-Low-Latency-3638279</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-9489" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="dodgy_coder" src="https://0.gravatar.com/avatar/67258db0b6551dbe7d020e58c9812017?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://dodgycoder.net">dodgy_coder</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, February 20th, 2012, 9:53 pm">February 20, 2012</abbr> at <abbr class="comment-time" title="Monday, February 20th, 2012, 9:53 pm">9:53 pm</abbr>
      </div>
      <div class="comment-text">
        <p>&gt; “Download War And Peace, Alice In Wonderland or any other book that is now out of copyright if you need real strings of words for your fake tweets” </p>
        <p>Nice post, and this above is a good idea but in doing so don’t forget about non-english languages such as Chinese and Japanese which have different character sets…</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-9490" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Travis Reeder" src="https://0.gravatar.com/avatar/ae5aa3978c0258532cbda24a495459d4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.iron.io">Travis Reeder</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, February 20th, 2012, 9:59 pm">February 20, 2012</abbr> at <abbr class="comment-time" title="Monday, February 20th, 2012, 9:59 pm">9:59 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Great post Phil. I hope this will encourage people that want to get into the big data space to just get out there and do it. The best experience someone can get is to make something, whether it’s for work or just a project for fun/learning. When I’m hiring, I put a heavy weight on people that have done their own projects just for the sake of learning. </p>
        <p>Subscribed.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-9504" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Patrick Dobson" src="https://1.gravatar.com/avatar/7ff76dfe425304e6fbab155269111cb1?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://pdobson.com">Patrick Dobson</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, February 21st, 2012, 4:42 am">February 21, 2012</abbr> at <abbr class="comment-time" title="Tuesday, February 21st, 2012, 4:42 am">4:42 am</abbr>
      </div>
      <div class="comment-text">
        <p>Voter files are also freely available. You could get millions of real records from every state. Here is Ohio:</p>
        <p>https://www2.sos.state.oh.us/pls/voter/f?p=111:1:3229553117069219</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-9505" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Aquecedor" src="https://1.gravatar.com/avatar/99d17f0d19f9567f9c0b82e2dc646b5e?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.solartec.com.br">Aquecedor</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, February 21st, 2012, 4:45 am">February 21, 2012</abbr> at <abbr class="comment-time" title="Tuesday, February 21st, 2012, 4:45 am">4:45 am</abbr>
      </div>
      <div class="comment-text">
        <p>I have always been fascinated by large datasets. I hope to play with this as soon as I have time. Great and inspiring post.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-9510" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Seamus Abshere" src="https://1.gravatar.com/avatar/139ef2e3a8dd85aa6a9ea5c82b2fe034?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://brighterplanet.com/">Seamus Abshere</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, February 21st, 2012, 9:14 am">February 21, 2012</abbr> at <abbr class="comment-time" title="Tuesday, February 21st, 2012, 9:14 am">9:14 am</abbr>
      </div>
      <div class="comment-text">
        <p>We’ve got some curated datasets here:</p>
        <p>https://data.brighterplanet.com/</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-18052" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="JJ" src="https://1.gravatar.com/avatar/1d23b2e052699c2118f355c6a15235fe?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">JJ</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, June 20th, 2012, 8:17 pm">June 20, 2012</abbr> at <abbr class="comment-time" title="Wednesday, June 20th, 2012, 8:17 pm">8:17 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Thanks for the great post.  I have been spending some time trying to get creative with data set sources.  If you could have access to any data set out there, what would choose?  Its fun to dream.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-286590" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Sudhakar Singh" src="https://0.gravatar.com/avatar/c88c1549d683bc87c6b87272e7d48a8d?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Sudhakar Singh</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, May 5th, 2014, 1:14 am">May 5, 2014</abbr> at <abbr class="comment-time" title="Monday, May 5th, 2014, 1:14 am">1:14 am</abbr>
      </div>
      <div class="comment-text">
        <p>From where can I find Transnational Datasets?  I want to apply Association Rule Mining algorithms on it.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

