---
title: "De-volatile Your Memcached. Upgrade to Membase"
date: "2011-01-21T12:16:00-08:00"
template: "post"
draft: false
slug: "de-volatile-your-memcached-upgrade-to-membase"
category: "DevOps"
tags:
  - caching
  - distributed
  - key-value store
  - membase
  - memcached
  - moxi
  - nosql
  - review
  - web-development
description: "Membase's TCP interface is identical Memcached, so migrating your existing code-base will not be an issue at all."
socialImage:
  publicURL: "/media/images/2011/01/memcached_to_membase_sq.jpg"
---
<a href="https://www.flickr.com/photos/hojusaram/484764113/" title="20070505-135556.jpg by hojusaram, on Flickr"><img align="left" alt="20070505-135556.jpg" height="500" src="/media/images/memcached_to_membase/memcached_to_membase.jpg" style="padding-bottom: 20px" width="333"/></a>

## Migrate From Memcached

Memcached is usually your first stop when optimizing a website. It is a great tool. It’s fast and simple. You can build on this simplicity to do a lot of cool stuff. It is cool because it makes your website functionality so much faster and your boss will pat you on the back and say “Wow! You really did some magic on the site. Let’s talk about that promotion. Do you play golf?”. Yes, you managed to pull a rabbit out of the hat and your website has gone from zero to Matrix’s Nero. [Matt Ingenthron](https://twitter.com/#!/ingenthr) said internally at [Membase Inc](https://www.membase.com/company) they view Memcached as a rabbit. It is fast, but it is pretty dumb and procreates quickly. Before you know it, it will be running wild all over your system. Once you have sped up one area of the site, you will start crowbarring Memcached into every corner. And why not.

### It’s Easy

One of the first things you will find about Membase is that it’s TCP interface is identical Memcached, so migrating your existing code-base will not be an issue at all. Deployment seems simple enough. In fact, you could get away with not touching your application code or client configuration at all if the IPs and ports do not change. This is a big win for Membase, due to large usage of Memcached out there.

## What Is Wrong With Memcached?

Memcached has it’s limitations. It does not scale well. While, there is no reason you could not have a large cluster of machines, moving to large+1 machines is going to lose a lot of your data. There is no master to organize the re-balancing of the data. In effect, each client (each instance your application) is the master. Memcached servers are simple dumb beasts. You can tell any Memcached server to store any key-value and it will, even if the client will never find it there.

### Memcached’s Hashing Algorithm

Memcached servers know nothing of the other Memcached servers in the cluster. The client decides which Memcached server will store which key by using a hashing algorithm. The algorithm takes, as input, a _list_ of Memcached servers and the _key_ of data to store. It will always return the same server for a given key and does so in a way that scatters the keys evenly across the Memcached servers. The same hashing algorithm is built into all of the clients. Therefore, all of the clients need to being using the same algorithm and the same list of servers (configured in the same order). If one client has a list of 10 servers and another client has a list of 11 servers, then they will not be writing and reading the same keys in the same place, and things will get a tad confusing.

### It’s Fast!

On the plus side, using a hashing algorithm is incredibly fast and means you do not need to talk to any other service to find the data. The more you look at Memcached the more you can see these areas where speed has been made the priority. It has been this speed that has made Memcached such a powerful force as a caching solution and so successful.

### The Shuffle

With Memcached’s hashing algorithm, the resulting server for a given key is heavily dependent on the number of servers. Therefore, if the number of servers changes, then the locations of the keys will shuffle. Key “A” will move from server “X” to server “Y” while key “B” might move in the opposite direction, from server “Y” to server “X”. Since there is no migration of the data between Memcached servers, the data is effectively lost and this could be most of your data. I say “effectively lost” because the data is still in it’s original place, but the clients are now looking for it in a different place, due to changing the input (number of servers) to the hashing algorithm. The Memcached servers are blindly unaware that anything has changed.

## Membase’s Data Management

Knowing this weakness of the hashing algorithm, I was surprised that Membase was still using this. The main weakness is that it puts the onus on the client to decide where to put the data. While this is fast, it overly burdens the clients with making important decisions in cluster management.

### Re-routing

Membase has managed to lift this burden from the client by allowing the client to connect to any of the Membase servers and then _re-routing_ the request if necessary. Unlike Memcached, each Membase server knows which keys it should be storing data for. When one Membase server receives a request for a key that it is not responsible for, it re-routes the request to the appropriate Membase server. This is handled by a component of Membase called “Moxi” (“Memcached Proxy”).

### Buckets And Replication

Membase hashes keys to “buckets”, not servers, and the number of buckets remains fixed. Buckets are a way to divide up the entire dataset into equal chunks. The hashing algorithm ensures the even distribution of keys across the buckets. By default the number of buckets it 1024.

Since the number of buckets does not change, the assignment of a given key to a given bucket does not change, so keys remain in the same bucket as your cluster grows. The buckets still need to be assigned to Membase servers, but this is easier to manage with a known fixed number of items. It also allows for assigning the same buckets to multiple servers. Yes, Membase provides replication for your data.

### Cluster Size Limitations

It is important to note that using the default of 1024 buckets will prevent you from having more than 1024 nodes in your cluster. While this is a big cluster, there are many organisations out there managing their data with clusters of more that 1024 machines. In Membase’s current state this limitation is a mute point, since even 1024 nodes is not feasible due to way nodes communicate with each other. Having no master and the fact that “each node is created equal” means that each node is talking to all the other nodes in the cluster. This is a lot of network chatter and it seems that it would grow [factorially](https://en.wikipedia.org/wiki/Factorial). This is a focus for further development of Membase.

## Moxi – The Memcached Proxy

I mentioned above that _Moxi_ is the component of Membase that handles the re-routing of requests to other Membase servers when the server receiving the original request is not the one responsible for the data requested. Here I will explain a little more about what Moxi is, since it is a major part of Membase’s design.

Moxi was bulit for Memcached to get around the problem of maintaining configuration within the client. With Moxi, all clients can talk to a single Moxi and requests will be proxied through to the appropriate Memcached server. Additional Memcached servers can be added or removed with no change to the clients.

When new Memcached servers are added to or removed from the cluster, Moxi can be notified over the “management channel”.

Just like Memcached servers can run in parallel with no knowledge of each other, so can Moxi servers. Each Moxi server can talk to the same cluster of Memcached servers. Because we can run multiple Moxi servers in parallel, we can conveniently run one Moxi server on each of the client machines. These Moxi servers can provide caching for popular GET requests, and result in faster responses for these requests.

When a Memcached server fails, Moxi pushes messages out over the management channel. This allows some form of manager, listening to the management channel, to reconfigure the cluster and push configuration changes back to all the Moxi servers. Better still, a Moxi can listen to other Moxi’s management channels and they can work as a team.

Moxi is now a major component of Membase. In helps manage the cluster. It sits at the edge of each Membase server, listening for requests from clients and re-routes those requests to other Membase servers if necessary.

## There Is No Master. All Nodes Created Equal

Just as Memcached has no master, neither does Membase. In Memcached each client is effectively a master, as this is where all of the key management logic lies, using the hashing algorithm. In Membase this management logic lies within each Membase server, _not_ within the client. All nodes are created equal. Therefore there is no single point of failure. This is a very important benefit of Membase. A masterless system is a powerful thing. It can truly run wild – much like rabbits.

## “Smart Clients”

_Smart clients_ are clients that embed the _Moxi_ component. Without this, the client does not know where the data is stored in the cluster. It will arbitrarily ask one of the Membase servers. That Membase server may have the data and can return it, but more likely it will re-route the request to the Membase server that does have the data. This re-routing can be avoided if the client runs the Moxi component itself. As data is moved around within the cluster and Membase servers leave and join, the client’s Moxi will always have an up-to-date record of where to find the data for a given key. It can now request it directly from the Membase server responsible for the data and avoid this re-routing step.

A second benefit to smart clients is _persistent connections_. Moxi can more intelligently create connections to each Membase server and keep those connections open as long each server is part of the cluster. Not needing to establish a new TCP connection \[or re-routing\] each time you do a SET or GET from the client is a big win for performance.

## Streaming Changes

Having done some work with [messaging queues](https://en.wikipedia.org/wiki/Message_queue), I like the idea of being able to tap into a system and see the data that is flowing through it without affecting the system’s functionality. This allows for re-using the live data for other purposes without having the re-code or re-architecture your system. Membase uses streaming for replication. A new node can have the contents of a bucket streamed to it from another node and it can then continue listening on this stream for real-time updates to the data. Alternatively, an interested third-party can tap into the stream and watch for real-time changes as they flow through. This can be useful if you want to push some or all of those changes back up to the client or into another data-store, such as Hadoop, HBase or MySQL. While Membase’s focus is speed and simplicity, other tools might be more useful for structured (MySQL) or heavy-weight (Hadoop) manipulation of your data.

## Persistence

If you have used Memcached and have gotten carried away with how it can make everything faster, you may have felt some pain when that volatile memory disappeared into the ether due to a restart or a failure. Your house of cards comes tumbling down as suddenly nothing is in cache! Your application is scrambling around trying to fully serve your users and the database is screaming at the application, “what the \*%$@ is going on up there?”. All your lack-of-performance implementation sins are exposed in one fell swoop and your boss stops inviting you to play golf. It’s tough, but a few hours later Memcached is singing again with a full belly of cache and your application layer is putting the kettle on to make your exhausted database a nice cuppa tea. You start to wonder… how can I persist this “caching” data?

Membase persists data to disk. This may be the single thing you want to know. Even if you do not care about how it works, or you have no time to investigate and compare it to other NoSQL data-stores, you should be able to sleep better knowing there is “an upgrade” for Memcached that is less votatile.

Membase’s configuration specifies two conditions for writing data to the disk. It will be written when it reaches a certain age or if it has been updated frequently. Frequently updated data will never get old enough to reach the required age for persistence, so this alternative rule is required. 

## Conclusion

It seems the philosophy of Membase is similar Memcached, “Be fast, but know your limitations”. I liked this with Memcached and I like it with Membase, too. The choices they have made in the design of Membase are about performance and taking Memcached to the next level. While it is a huge leap from Memcached, it is only a small step for developers who want to deploy this as an alternative to replace, or “upgrade”, Memcached. There is no reason that it will be particularly slower than Memcached, since your data can still live in the RAM that you are currently using for Memcached. You have the additional benefit of that data being persisted to disk in the background and being immediately available after a rewrite. Volatility is has been replaced with replication and rebalancing.

Some other things that were not covered in this blog post are the authentication layer in Membase (via SASL or port), how Membase makes use of SQLite under the hood, the REST interface, how Membase uses vector clocks to handle collisions from multiple-writers and the very impressive user-interface to Membase, which I will cover in a future post.

While the focus of this blog post has been on Membase as a replacement to Memcached as a persistent caching layer, Membase is a distributed key-value \[NoSQL\] data-store in it’s own right and can be used for much more than just caching.

## Resources

*   [Membase](https://www.membase.com/)
*   [“Moxi – Memcached Proxy”. Good presentation on Slideshare](https://www.slideshare.net/northscale/moxi-memcached-proxy)
*   [Membase NoSQL Whitepaper (PDF)](https://www.membase.com/sites/default/files/uploads/all/whitepapers/Membase-NoSQL-Whitepaper.pdf)

<div id="comments">
  <h3 id="comments-number" class="comments-header">6 responses to “De-volatile Your Memcached. Upgrade to Membase”</h3>
  <ol class="comment-list">
    <li id="comment-829" class="comment even thread-even depth-1 comment reader">
      <img alt="Perry Krug" src="https://1.gravatar.com/avatar/70b38bbe52aff967313a7acf9dd8f43d?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.membase.com">Perry Krug</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, January 22nd, 2011, 6:31 pm">January 22, 2011</abbr> at <abbr class="comment-time" title="Saturday, January 22nd, 2011, 6:31 pm">6:31 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Phil, this is a fantastic blog…thank you very much.</p>
        <p>A few points of clarification. Please take these as constructive criticism, you have done an excellent job up to here and I simply want to add a bit more detail for your/our readers:</p>
        <p>- When talking about the hashing algorithm stuff, you are absolutely correct about how memcached handles things. You are also correct about how Membase handles the “re-routing” of client requests. However, we actually use “vBuckets” (not regular buckets…those are the logical keyspaces within a Membase cluster. Yes, I know it’s confusing…we apologize  .</p>
        <p>- Membase doesn’t actually have a cluster size limit. You are correct that you can only have 1024 servers holding “active” data, but if you have more than 1024 nodes in the cluster, some will only be handling replica data. The cluster itself will still be viable. We are also planning on raising this limit in the future when we run into deployments that need it. 1024 was appropriate from a testing and qualification perspective for now. You are still correct about the chattiness of the nodes and that we are continuing development in this area.</p>
        <p>- It’s worth pointing out that running Moxi on the client side is our best-practice for non-”smart” clients and will eliminate any extra network hops when retrieving data since Moxi can go directly to the Membase server for a particular key.</p>
        <p>- ”smart” clients don’t actually embed Moxi within them…they simply contain the same logic that Moxi does. Take a look at the Enyim (https://memcached.enyim.com/) client and our updated spymemcached client for Java (https://wiki.membase.org/display/membase/prerelease+spymemcached+vBucket). We’ve also begun the process of writing a libmembase C library (https://trondn.blogspot.com/2010/12/building-libmembase.html). You can really think of Moxi as the first “smart” client…though it can do much more.</p>
        <p>- It “may” be worthwhile calling out Membase’s features of replication and the TAP interface a bit more if you like…they’re rather key to the whole system (no pun intended)</p>
        <p>Thanks again and I really look forward to your continuing investigation of Membase. Please let me know if there’s anything I can do to help in your pursuit.</p>
        <p>Perry Krug</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-846" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, January 24th, 2011, 4:42 pm">January 24, 2011</abbr> at <abbr class="comment-time" title="Monday, January 24th, 2011, 4:42 pm">4:42 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Wow! Thanks for the excellent feedback on the post, Perry. I will definitely incorporate all those points in future Membase posts. I appreciate the clarifications and all the pointers to more resources.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-4104" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Peter Smith" src="https://0.gravatar.com/avatar/e8f1b679872435d5dd0d09d85f204395?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.linuxbox.co.uk">Peter Smith</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, August 6th, 2011, 11:56 am">August 6, 2011</abbr> at <abbr class="comment-time" title="Saturday, August 6th, 2011, 11:56 am">11:56 am</abbr>
      </div>
      <div class="comment-text">
        <p>Great article, but just a quick note on memcache limitations. Consistent hashing  (the PHP APIs support this, and some other languages too) solves the problems that occur when memcache servers are added/removed from the pool, so this isn’t really a reason not to use memcache (since there’s a simple solution).</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-4105" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, August 6th, 2011, 2:26 pm">August 6, 2011</abbr> at <abbr class="comment-time" title="Saturday, August 6th, 2011, 2:26 pm">2:26 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Peter. I had not come across this concept. </p>
            <p>Here’s a little explanation for other readers…<br />
https://code.google.com/p/memcached/wiki/FAQ</p>
            <p>Here is a Ruby implementation…<br />
https://www.mikeperham.com/2009/01/14/consistent-hashing-in-memcache-client/</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-4106" class="comment even thread-even depth-1 comment reader">
      <img alt="Perry Krug" src="https://0.gravatar.com/avatar/202cbbf282c581c26d97a79f0fcfec5f?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.couchbase.com">Perry Krug</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, August 6th, 2011, 4:01 pm">August 6, 2011</abbr> at <abbr class="comment-time" title="Saturday, August 6th, 2011, 4:01 pm">4:01 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Consistent hashing does make the addition/removal of servers “less” impactful, but it’s not perfect.  As a database, Membase (and now Couchbase Server) needed to be perfectly consistent and the vbucket concept has been extremely valuable to our project. </p>
        <p>As simply a memcached replacement, Membase does offer a number of other advantages like replication and failover, as well as persistence to disk (though some argue that a cache shouldn’t have to be persistent).  With the introduction of CouchDB underneath (creating the Couchbase Server 2.0 product) you also get robust indexing and map-reduce functionality (useful even in a “cache”) and cross datacenter replication is close on the horizon.</p>
        <p>We believe that memcached is still an extremely valuable technology and has many valid use cases, we even have customers running “standard” memcached within their Membase cluster.  Mostly for the enhanced monitoring and clustering that we bring to it, but also for much more transient data that they truly just need a volatile cache for.</p>
        <p>Full disclosure, I’m a bit biased having worked at Couchbase (formerly Membase, formerly NorthScale) for over a year now, but there is some seriously cool and useful technology being born before our eyes.</p>
        <p>Perry Krug<br />
Sr. Solutions Architect, Couchbase<br />
perry@couchbase.com</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

