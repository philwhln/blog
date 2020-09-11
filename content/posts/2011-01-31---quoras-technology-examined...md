---
title: "Quora’s Technology Examined"
date: "2011-01-31T12:37:00-08:00"
template: "post"
draft: false
slug: "quoras-technology-examined.."
category: "Software Engineering"
tags:
  - adam d'angelo
  - amazon ec2
  - aws
  - charlie cheever
  - comet
  - git
  - haproxy
  - livenode
  - long-polling
  - memcached
  - mysql
  - nginx
  - nosql
  - paste
  - pylons
  - quora
  - quora.com
  - search-box
  - steve souders
  - technology
  - thrift
  - tornado
  - ubuntu linux
  - webnode2
  - webscale
description: "In this blog post I will delve into the snippets of information available on Quora and look at Quora from a technical perspective. What technical decisions have they made? What does their architecture look like? What languages and frameworks do they use? How do they make that search bar respond so quickly?"
---
<a href="https://www.flickr.com/photos/a_mason/4280360776/" title="Magnifying the Circuit Board by Andrew Mason, on Flickr"><img alt="Magnifying the Circuit Board" height="338" src="https://commondatastorage.googleapis.com/philwhln/blog/images/quora_microscope/quora_microscope.jpg" width="450"/></a>

Quora has taken the tech and entrepreneurial world by storm, providing a system that works so fluidly that it is sometimes hard to see what the big fuss is all about. This slick tool is powered, not only by an intelligent crowd of askers and answerers, but by a well-crafted backend created by co-founders who honed their skills at Facebook.

It is not surprising that, with all the smart people using this smart tool, there are many pondering on how it works so well. The NoSQL boffins scratch their heads and ponder such questions as, “[Why does Quora use MySQL as the data store rather than NoSQLs such as Cassandra, MongoDB, CouchDB, etc?](https://www.quora.com/Why-does-Quora-use-MySQL-as-the-data-store-rather-than-NoSQLs-such-as-Cassandra-MongoDB-CouchDB-etc)“.

In this blog post I will delve into the snippets of information available on Quora and look at Quora from a technical perspective. What technical decisions have they made? What does their architecture look like? What languages and frameworks do they use? How do they make that search bar respond so quickly?

*   [Components Of Quora](/quoras-technology-examined#components-of-quora)
*   [What’s Cooking Under That Hood?](/quoras-technology-examined#whats-cooking-under-that-hood)

*   [The Search-Box](/quoras-technology-examined#the-search-box)
*   [Webnode2 And LiveNode](/quoras-technology-examined#webnode2-and-livenode)
*   [Amazon Web Services]()

*   [Ubuntu Linux](/quoras-technology-examined#ubuntu-linux)
*   [Static Content](/quoras-technology-examined#static-content)

*   [HAProxy Load-Balancing](/quoras-technology-examined#haproxy-load-balancing)
*   [Nginx](/quoras-technology-examined#nginx)
*   [Pylons And Paste](/quoras-technology-examined#pylons-and-paste)
*   [Python](/quoras-technology-examined#python)
*   [Thrift](/quoras-technology-examined#thrift)
*   [Tornado](/quoras-technology-examined#tornado)
*   [Long Polling (Comet)](/quoras-technology-examined#long-polling-comet)
*   [MySQL](/quoras-technology-examined#mysql)
*   [Memcached](/quoras-technology-examined#memcached)
*   [Git](/quoras-technology-examined#git)

*   [Charlie Cheever Follows “14 Rules for Faster-Loading Web Sites”](/quoras-technology-examined#charlie-cheever-follows-14-rules-for-faster-loading-web-sites)
*   [Conclusion](/quoras-technology-examined#conclusion)
*   [Recommended Reading](/quoras-technology-examined#recommended-reading)
*   [Resources](/quoras-technology-examined#resources)

<h2 id="components-of-quora">Components Of Quora</h2>

The general components that make up Quora are…

*   You can ask __questions__
*   You can __answer__ questions (anonymously if you desire)
*   You can __comment__ on answered questions
*   You can __vote__-up or vote-down answers to questions
*   Questions can be assigned to __topics__
*   You can write a __post__ (a informative statement, rather like a orphaned answer or blog post)
*   You can __follow__ questions, topics or other users
*   A super-fast auto-complete __search__-box at the top, which doubles as the method for entering new questions

The last point, the super-fast auto-complete search-box, is one of the defining features of Quora. You can see immediately, as you begin to enter a question, whether somebody else has already asked that question or if there is a topic or post on the subject. Let’s start there…

<h2 id="whats-cooking-under-that-hood">What’s Cooking Under That Hood?</h2>

<h3 id="the-search-box">The Search-Box</h3>

Only the questions, topic labels, user names or post titles are indexed and served up to the search-box. There is no full-text search, so searching the content of questions and answers will not work. The text that is indexed is tokenized so that words in a different order will still be matched. Prefix matching enables best matches to be shown before the entire word is entered. For instance, typing “mi” might immediately show “Microsoft” in the results.

There is some simple stemming of words, since “nears” matches “near”, but “pony” does not match “ponies”. “Topic-aliases” allow for similar matches on topic names, such as “startup” and “start-up”. These topic-aliases have been manually entered by users. Otherwise these would not match.

If a duplicate question is redirected to another question (a feature of Quora), then that original question will still appear in the search results, since it increases the chances of a match. There is no [n-gram](https://www.lucidimagination.com/blog/2009/09/08/auto-suggest-from-popular-queries-using-edgengrams/) indexing, so slight mis-spellings will not match. For instance, “gooogle” (with an extra “o”) finds nothing.

Previously, they did use an open source search server, called [Sphinx](https://sphinxsearch.com/). It supports the features they are using above, but they have since moved from this [due to real-time constraints](https://www.quora.com/What-is-the-best-open-source-solution-for-implementing-fast-auto-complete). Their new solution is built in-house and allows them better prefix indexing and control over the matching algorithms. They built this in Python.

<blockquote><p><a href="https://www.quora.com/What-libraries-does-Quora-use-for-search"><img height="16" src="https://www.quora.com/favicon.ico" style="padding-right: 10px" width="16"/>What libraries does Quora use for search?</a><br/>
<small>Adam D’Angelo, Quora Founder (Nov 13, 2010)</small><br/>
Our search is custom-written. It doesn’t use any libraries aside from Thrift, and Python’s unicode library, which we use for unicode normalization.
</p></blockquote>

<h4 id="speedy-queries">Speedy Queries</h4>

Did I mention that the search-box is fast? I did some tests and found the responses to be impressive. Queries are sent over AJAX as a GET request. Responses come back as JSON with the rendered HTML embedded inside the JSON. Rendering of the results on the server-side, as opposed to rendering them in JavaScript, seems to be due to the need to highlight matching words in the text. This is sometimes too complex for JavaScript. For instance, typing “categories” might highlight the world “category” in the result text.

I was seeing responses of roughly 50 milliseconds per query from my [Linode](https://www.linode.com/) machine. Quora does not short-change you when sending requests. From within the browser, I found typing “Microsoft” (9 characters) would result in nine requests to the Quora search server, no matter how fast you type. As you will [see later](/quoras-technology-examined#long-polling-comet), the server is in control, so if it did become over-loaded, then it could update the results less frequently without changing the JavaScript.

Quora uses persistent connections. A HTTP connection is established with the server when you start typing the search query. This connection is kept open and further requests are made on this same open connection. The connection will terminate (times-out) if not used for 60 seconds. If a connection times-out then a new connection is established when typing begins.

To simulate the typing of a word into the search-box, I sent the following requests, character-by-character, across a persistent connection. For instance “butler” is six requests (“b”, “bu”, “but” … “butler”).

<pre>"butler" (6 chars) duration: 0.393 secs 0.065 secs per query
"butler monkeys" (14 chars) duration: 0.672 secs 0.048 secs per query
"fasdisajfosdffsa" (16 chars) duration: 0.746 secs 0.046 secs per query</pre>

That last query was used to test if there was a slow-down for a word that would obviously not be in a caching layer. I saw no slow-down. This means that they are not caching, caching is only used to take the load off the backend search engine or they are doing something smarter (e.g. if there is no match for “fasd” then there will be no match for “fasdi”, so abort).

<blockquote><p><a href="https://www.quora.com/Quora-product/Is-Quora-going-to-implement-full-text-search"><img height="16" src="https://www.quora.com/favicon.ico" style="padding-right: 10px" width="16"/>Is Quora going to implement full-text search?</a><br/>
<small>Adam D’Angelo, I made a lot of the early Quora … (Sep 1, 2010)</small><br/>
Yes, eventually. We haven’t implemented this yet because we’ve prioritized other things, but we will definitely do it in the future.
</p></blockquote>

<h3 id="webnode2-and-livenode">Webnode2 And LiveNode</h3>

Webnode2 and LiveNode are some of Quora’s internal systems, which were built for managing the content. Webnode2 generates HTML, CSS and JavaScript and is tightly coupled with LiveNode, which is responsible for managing the display of the content on the webpage. Charlie Cheever says that he were to start a similar project without LiveNode, then the [first thing he would do is rebuild it](https://www.quora.com/What-limitations-has-Quora-encountered-due-to-LiveNode-WebNode#answers).

They seem very pleased with the technology they have built and [struggled to find its weaknesses](https://www.quora.com/What-limitations-has-Quora-encountered-due-to-LiveNode-WebNode#answers). One weakness is that it is tricky for LiveNode to keep track of what is happening within the browser as it pushes changes from the server. If users A and B are viewing the same question then ones interactions will affect the other. For instance, if user A up-votes an answer then that answer will be promoted and will visibly move up the page. This display change will be pushed over AJAX to user B’s browser. Any prior browser-side change that user B made, such as expanding a comments section, might be lost.

[LiveNode is written in](https://www.quora.com/What-is-LiveNode-written-in) Python, C++, and JavaScript. [jQuery](https://jquery.com/) and [Cython](https://cython.org/) is also used.

While they [would like to open-source LiveNode](https://www.quora.com/Is-Quora-planning-on-open-sourcing-LiveNode) and have tried to keep code separation, doing so right now would be too much work and would take time away from their main goal, which is making Quora better.

Charlie Cheever points out that webnode2 is [unrelated to the “free and easy website builder” called Webnode at webnode.com](https://www.quora.com/Quora-Infrastructure/What-is-webnode2).

<h3 id="amazon-web-services">Amazon Web Services</h3>

Amazon EC2 and S3 is used for their hosting. While this is not as cost-effective in the long-term as running your own servers, it is perfectly designed for fast growing companies like Quora.

<h4 id="ubuntu-linux">Ubuntu Linux</h4>

Quora uses Ubuntu Linux as its OS of choice. No major surprises there. It is easy to deploy and manage on Amazon EC2. Adam D’Angelo [points out](https://www.quora.com/Quora-Infrastructure/Which-Linux-flavor-does-Quora-use-Why) that he used Debian Linux at high school and college and stuck with it because [“it works and there hasn’t been a compelling reason to switch”](https://www.quora.com/Quora-Infrastructure/Which-Linux-flavor-does-Quora-use-Why).

<h4 id="static-content">Static Content</h4>

You only need to look at the source HTML of any Quora webpage to see that they are using Amazon’s distributed content delivery network, [Cloudfront](https://aws.amazon.com/cloudfront/). URLs are in the form…

<pre>https://d2o7bfz2il9cb7.<strong>cloudfront</strong>.net/main-thumb-670336-25-7kmigSSkkdusoE6gHRkdQsXfjuTCaxQs.jpeg</pre>

CloudFront is used for all static images, CSS and JavaScript (except for Google’s Analytics JavaScript, which is hosted by Google). [Images are uploaded to the EC2 machine, then resized and uploaded to S3](https://www.quora.com/How-is-Quora-doing-image-uploads-to-Amazon-S3). This is managed using the [Python S3 API](https://aws.amazon.com/code/134).

<h3 id="haproxy-load-balancing">HAProxy Load-Balancing</h3>

Quora uses [HAProxy](https://haproxy.1wt.eu/) at the front-line, which load-balances onto the distributed Nginx servers behind them.

<h3 id="nginx">Nginx</h3>

Behind the load-balancer, [Nginx](https://nginx.org/en/) is used as a reverse-proxy server onto the web-servers.

To understand more about this setup I recommend reading “[Using Nginx As Reverse-Proxy Server On High-Loaded Sites](https://kovyrin.net/2006/05/18/nginx-as-reverse-proxy/)“.

<h3 id="pylons-and-paste">Pylons And Paste</h3>

[Pylons](https://pylonshq.com/), a lightweight web framework, is used as their main web-server behind Nginx. They use the default [Pylon + Paste stack](https://spacepants.org/blog/pylons-paste-stack).

[Pylons](https://pylonshq.com/) was selected much like you would select a pumpkin for Haloween. They scooped out the insides, such as templates and the ORM, and replaced it with their own technology, written in Python. This is where [LiveNode and webnode2 reside](https://www.quora.com/What-languages-and-frameworks-were-used-to-code-Quora).

[MochiMedia](https://www.mochimedia.com/) was also one of the inspirations for using Pylons, since they were using it themselves.

<h3 id="python">Python</h3>

Coming from Facebook, it was a good bet that Charlie and Adam would choose PHP for their development language. As Adam points out, “[Facebook is stuck on that for legacy reasons, not because it is the best choice right now](https://www.quora.com/Why-did-Quora-choose-Python-for-its-development)“. From this experience they knew that choosing technologies, especially programming languages, for the long-run was very important. They also looked at C\#, Java, and Scala. [Discounting Mono](https://www.quora.com/Why-did-Quora-choose-Python-for-its-development), C\# would be a choice of more than just the language. It would require them to build on-top of a Microsoft stack. Python won over Java because it is more expressive and quicker to write code than Java. Scala was too new. Adam mentions [speed and the lack of type-checking](https://www.quora.com/Why-did-Quora-choose-Python-for-its-development) as drawbacks with Python, but they both already knew the language reasonably well. Where Python lacks speed for performance critical backend components, they opt to write them in C++. They saw Ruby as a close match to Python, but their experience with Python and lack of experience in Ruby, made Python the winner. Python _2.6_, to be precise.

Additional benefits for using Python are the fact that data-structures that map well to JSON, code readability, there is a large collection of libraries and the availability of good debuggers and reloaders. Browser-server communication using JSON is major component of what Quora does, so this was an important factor.

[No IDEs](https://www.quora.com/Adam-DAngelo/What-version-of-Python-are-you-programming-in-and-what-IDE-do-you-use) are used for development as most use the [Emacs text editor](https://www.gnu.org/software/emacs/). Obviously this is personal choice, and would change as the team grows.

[PyPy](https://codespeak.net/pypy/dist/pypy/doc/), a project that aims to produce a flexible and fast Python implementation, was also mentioned as something that might give them a speed-boost.

<h3 id="thrift">Thrift</h3>

[Thrift](https://incubator.apache.org/thrift/) is used for communications between backend systems. The Thrift service is written in C++.

<blockquote><p><a href="https://www.quora.com/Why-would-you-write-a-Thrift-service-in-C"><img height="16" src="https://www.quora.com/favicon.ico" style="padding-right: 10px" width="16"/>Why would you write a Thrift service in C++?</a><br/>
<small>Adam D’Angelo, I’ve written a lot of Python, in… (Sep 4, 2010)</small><br/>
Mainly if you want to keep data in memory between requests, and want to keep your Python code stateless. Writing a Python wrapper around a C library involves some memory management with reference counting that requires some understanding of the Python internals, but writing a thrift interface is simple. You also isolate failures this way – if the service goes down it won’t take the Python code down with it.
</p></blockquote>

<h3 id="tornado">Tornado</h3>

The [Tornado](https://www.tornadoweb.org/) web framework is used for live updating. This is their Comet server, which handles the large volumes of open connections used for long-polling and pushes updates to the browsers.

<h3 id="long-polling-comet">Long Polling (Comet)</h3>

Quora does not display just static web pages. Each page will update will new content as questions, answers and comments are submitted by other others. As Adam D’Angelo points out, one of the best ways to do this currently is with “long polling”. This is different to “polling”. With _polling_ the browser will repeatedly send requests to the server saying “Any updates?” and the server will respond with “No”. A few seconds later it will ask again, “How about now?”. “No”. “How about now?”, “No, already!”. This puts the client (web-browser) in the driver’s seat. This is backwards because the client does not how long to wait before asking again. If the client asks the server too frequently then it will unduly overload the server. If it pings the server too infrequently, then server will be sitting on updates while it waits for the client to request them and the end-user will not see updates immediately.

Long polling, also known as [Comet](https://en.wikipedia.org/wiki/Comet_(programming)), puts the server in control, by making the client wait for responses. The conversation between the client and server is the same, but instead of the client waiting before making another request, the server waits before it makes the response. The server can keep the connection open for a long period of time (e.g. 60 seconds) while it waits to see if any updates come in. When updates do come in it can respond immediately to the client. On receiving the update, the client then immediately sends a new request for more updates. The server, once again, delays responding until it knows something worth telling the client or enough time has past that it would be rude not to respond.

The benefit to long-polling is that there is less back-and-forth between the client and server. The server is in control of the timing, so updates to the browser can be made within milliseconds. This makes it ideal for chat applications or applications that want really snappy updates for their users.

The down-side is that you are going have lots of open connections between the clients and your servers. If you have a million users (Quora will soon) and, if only 10% of them are online on your site, you will need an architecture that can hold open at least 100,000 concurrent connections. This assumes they only have one tab open to your site. Right now I have 7 tabs open for quora.com in my browser. Each tab usually has multiple connections open to quora.com. In short, Quora must maintain _a lot_ of open connections.

The good news is that there are technologies specifically designed for this. It costs very little to hold open connections in memory if you free up all the resources used for that connection. For instance, Nginx (Quora uses this for proxying requests) is a single-threaded event-based application and uses very little memory for each connection. Each Nginx process is actively dealing with only one connection at a time. This means it can scale to tens of thousands of concurrent connections.

<blockquote><p><a href="https://www.quora.com/How-do-you-push-messages-back-to-a-web-browser-client-through-AJAX-Is-there-any-way-to-do-this-without-having-the-client-constantly-polling-the-server-for-updates"><img height="16" src="https://www.quora.com/favicon.ico" style="padding-right: 10px" width="16"/>How do you push messages back to a web-browser client through AJAX?  Is there any way to do this without having the client constantly polling the server for updates?</a><br/>
<small>Adam D’Angelo, Quora (Sep 29, 2010)</small><br/>
There is no reliable way to do this without having the client polling the server. However, you can make the server stall its responses (50 seconds is a safe bet) and then complete them when a message is ready for the client. This is called “long polling” and it’s how Quora, Gmail, Meebo, etc all handle the problem.</p><p>If you have a specialized server that uses epoll or kqueue, you should be able to hold on the order of 100k users per server (depending on how many messages are going). This is called the “c10k” problem. https://www.kegel.com/c10k.html
</p></blockquote>

<h3 id="mysql">MySQL</h3>

Just like Facebook, where co-founder Adam D’Angelo previously worked, Quora heavily uses MySQL. In answer to the Quora question “[When Adam D’Angelo says “partition your data at the application level”, what exactly does he mean?](https://www.quora.com/When-Adam-DAngelo-says-partition-your-data-at-the-application-level-what-exactly-does-he-mean)“, D’Angelo goes into the details of how to use MySQL (or relational-databases generally) as a distributed data-store.

The basic advice is to only partition data if necessary, keep data on one machine if possible and use a hash of the primary key to partition larger datasets across multiple databases. Joins must be avoided. He sites FriendFeed’s architecture as a good example of this. FriendFeed’s architecture is described by Bret Taylor in his post “[How FriendFeed uses MySQL to store schema-less data](https://bret.appspot.com/entry/how-friendfeed-uses-mysql)“. D’Angelo also [states](https://www.quora.com/NoSQL/In-what-parts-of-a-social-site-with-concert-listings-should-one-use-a-NoSQL-DB-versus-a-SQL-DB) that you should not use a NoSQL database for a social site until you have millions of users.

It is not only Quora and FriendFeed who are heavily using MySQL. Ever heard of “Google”? It is hard to imagine, since everything Google does has to scale so well, but in [the words of Google](https://googlecode.blogspot.com/2007/04/google-releases-patches-that-enhance.html), “Google uses MySQL \[...\] in some of the applications that we build that are not search related”. Google has released [patches](https://code.google.com/p/google-mysql-tools/wiki/Mysql4Patches) for MySQL related to replication, syncing, monitoring and faster master promotion.

<blockquote><p><a href="https://www.quora.com/How-does-one-evaluate-if-a-database-is-efficient-enough-to-not-crash-as-its-put-under-increasing-load"><img height="16" src="https://www.quora.com/favicon.ico" style="padding-right: 10px" width="16"/>How does one evaluate if a database is efficient enough to not crash as it’s put under increasing load?</a><br/>
<small>Adam D’Angelo, Quora (Oct 10, 2010)</small><br/>
One option is to simulate some load. Write a script that mimics the kinds of queries your application will be doing, and make sure it can handle the amount of load you want it to be ready for (especially as the size of the dataset changes).
</p></blockquote>

<h3 id="memcached">Memcached</h3>

[Memcached](https://memcached.org/) is used as a caching layer in front of MySQL.

<h3 id="git">Git</h3>

[Git](https://git-scm.com/) is [used for version control](https://www.quora.com/What-languages-and-frameworks-were-used-to-code-Quora).

<h3 id="javascript-placement">JavaScript Placement</h3>

If you look at Quora’s source you will see that the JavaScript comes at the end of the page. Charlie Cheever [suggests](https://www.quora.com/Why-is-the-Quora-website-so-fast) that this give the feeling of a quicker loading page, since the browser has content to display before the JavaScript has be seen.

<h2 id="charlie-cheever-follows-14-rules-for-faster-loading-web-sites">Charlie Cheever Follows “14 Rules for Faster-Loading Web Sites”</h2>

Steve Souders, author of High Performance Web Sites and Even Faster Web Sites, lists the following [rules for making websites faster](https://stevesouders.com/hpws/rules.php). This list is mentioned by Charlie Cheever, the co-founder of Quora, as one of the reasons for Quora’s speed.

>  
> “One resource we used as a guide is Steve Souders’ list of rules for high performance websites: <https://stevesouders.com/hpws/rules.php>”  
> <small>[ – Charlie Cheever, Quora](https://www.quora.com/Why-is-the-Quora-website-so-fast)</small>
> 

<div>
<div style="float: left">
Steve Souders’ 14 rules are…
<ul>
<li>Make Fewer HTTP Requests</li>
<li>Use a Content Delivery Network</li>
<li>Add an Expires Header</li>
<li>Gzip Components</li>
<li>Put Stylesheets at the Top</li>
<li>Put Scripts at the Bottom</li>
<li>Avoid CSS Expressions</li>
<li>Make JavaScript and CSS External</li>
<li>Reduce DNS Lookups</li>
<li>Minify JavaScript</li>
<li>Avoid Redirects</li>
<li>Remove Duplicate Scripts</li>
<li>Configure ETags</li>
<li>Make AJAX Cacheable</li>
</ul>
</div>
<div style="float: left; padding-left: 80px;">
<a href="https://www.amazon.com/gp/product/0596529309?ie=UTF8&amp;tag=getafil-20&amp;linkCode=as2&amp;camp=1789&amp;creative=9325&amp;creativeASIN=0596529309"><img border="0" src="https://images-na.ssl-images-amazon.com/images/I/41COtT-V1UL._SL160_.jpg"/></a><img alt="" border="0" height="1" src="https://www.assoc-amazon.com/e/ir?t=getafil-20&amp;l=as2&amp;o=1&amp;a=0596529309" style="border:none !important; margin:0px !important;" width="1"/>
<p><a href="https://www.amazon.com/gp/product/0596522304?ie=UTF8&amp;tag=getafil-20&amp;linkCode=as2&amp;camp=1789&amp;creative=9325&amp;creativeASIN=0596522304"><img border="0" src="https://images-na.ssl-images-amazon.com/images/I/41vfOvQugoL._SL160_.jpg"/></a><img alt="" border="0" height="1" src="https://www.assoc-amazon.com/e/ir?t=getafil-20&amp;l=as2&amp;o=1&amp;a=0596522304" style="border:none !important; margin:0px !important;" width="1"/></p>
</div>
<div style="clear:both;"></div>
</div>

<h2 id="conclusion">Conclusion</h2>

Quora is a great example of a modern tech start-up. They are very small team who understand the technologies they are using very well. They have made considered choices in the technology they have selected and have a good vision of which components would be better written from scratch. They seem keen to share these in-house technologies with the open-source community and I look forward to when they have the time to make this a reality.

I intend to keep following Quora and writing about them [__more in future blog posts__](/feed).

If you found this post useful then please [__leave a comment__](/quoras-technology-examined#comments-template) or [__follow this blog__](/feed).

<h2 id="resources">Resources</h2>

<ul style="padding-bottom: 40px;">
<li><a href="https://www.quora.com">Quora.com</a></li>
<li><a href="https://bret.appspot.com/entry/how-friendfeed-uses-mysql">How FriendFeed uses MySQL to store schema-less data</a></li>
<li><a href="https://stevesouders.com/hpws/rules.php">Steve Souders’ 14 Rules for Faster-Loading Web Sites</a></li>
</ul>

<div class="hentry post publish post-1 odd author-admin category-start-ups category-web-development post_tag-algorithms post_tag-amazon-ec2 post_tag-django post_tag-gunicorn post_tag-guy-kawasaki post_tag-iphone-api post_tag-jquery-template post_tag-json-api post_tag-mongodb post_tag-mysql post_tag-nginx post_tag-python post_tag-redis post_tag-slow-clients post_tag-summify post_tag-the-long-tail post_tag-tornado post_tag-tornado-swirl" id="post-1248">
<h2 class="entry-title"><a href="/summifys-technology-examined" rel="bookmark" title="Summify’s Technology Examined">Summify’s Technology Examined</a></h2>
<p class="byline"><span class="byline-prep byline-prep-author">By</span> <span class="author vcard"><span class="fn">Phil Whelan</span></span> <span class="byline-prep byline-prep-published">on</span> <abbr class="published" title="Monday, March 7th, 2011, 2:11 am">March 7, 2011</abbr> </p>
<div style="float: left;"><img alt="summify_technology_sq" class="attachment-post-thumbnail wp-post-image" height="180" src="/wp-content/uploads/2011/03/summify_technology_sq.jpg" title="summify_technology_sq" width="180"/></div>
<div class="entry-content">
<p>In this post I will look at the technology infrastructure behind <a href="https://summify.com/" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','summify.com/']);">Summify.com</a>, a website that strives to make our lives easier and helps us deal with the information overload we all experience every time we sit down at our computers. Summify has aggregated over 200 million stories from the web and serves them up on-demand through a series of different mediums. The website uses Tornado to push real-time updates out to the users and they have developed over a dozen backend systems, some of which I will cover in this blog post.</p>
</div>
<p><!-- .entry-content --></p>
<div class="continue-reading"><a href="/summifys-technology-examined"><strong>Continue reading…</strong></a></div>
</div>

In this post I will look at the technology infrastructure behind <a href="https://summify.com/" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','summify.com/']);">Summify.com</a>, a website that strives to make our lives easier and helps us deal with the information overload we all experience every time we sit down at our computers. Summify has aggregated over 200 million stories from the web and serves them up on-demand through a series of different mediums. The website uses Tornado to push real-time updates out to the users and they have developed over a dozen backend systems, some of which I will cover in this blog post.