---
title: "Summify’s Technology Examined"
date: "2011-03-07T02:11:00-08:00"
template: "post"
draft: false
slug: "summifys-technology-examined"
category: "Startups"
tags:
  - algorithms
  - amazon ec2
  - django
  - gunicorn
  - guy kawasaki
  - iphone api
  - jquery template
  - json api
  - mongodb
  - mysql
  - nginx
  - python
  - redis
  - slow clients
  - summify
  - the long tail
  - tornado
  - tornado swirl
description: "In this post I will look at the technology infrastructure behind Summify.com, a website that strives to make our lives easier and helps us deal with the information overload we all experience every time we sit down at our computers. Summify has aggregated over 200 million stories from the web and serves them up on-demand through a series of different mediums. The website uses Tornado to push real-time updates out to the users and they have developed over a dozen backend systems, some of which I will cover in this blog post."
socialImage:
  publicURL: "/wp-content/uploads/2011/03/summify_technology_sq.jpg"
---
<a href="https://www.flickr.com/photos/zipckr/3925513417/" title="Web 2.0 icons by zipckr, on Flickr"><img alt="Web 2.0 icons" height="338" src="https://commondatastorage.googleapis.com/philwhln/blog/images/summify_technology/summify_technology.jpg?1" width="450"/></a>

Following on from examining [Quora’s technology](/quoras-technology-examined), I thought I would look at a tech company closer to home. Home being [Vancouver, BC](https://www.tourismvancouver.com/visitors/). While the tech scene is much smaller here than in [the valley](https://en.wikipedia.org/wiki/Silicon_Valley), [it is here](https://www.readwriteweb.com/start/2011/01/never-mind-the-valley-heres-va.php). In fact, Vancouver boasts the largest number of [entrepreneurs per capita](https://www.readwriteweb.com/start/2011/01/never-mind-the-valley-heres-va.php).

[Summify.com](https://summify.com/) is a website that strives to make our lives easier and helps us deal with the information overload we all experience every time we sit down at our computers. The founders of this start-up, [Cristian Strat](https://cristianstrat.com/) and [Mircea Paşoi](https://mirceapasoi.com/), seem to have all the right ingredients for success. This is their biggest venture so far, but not their first. They have previously built [Infoarena.ro](https://infoarena.ro) and [Balaur.ro](https://balaur.ro/), which are both focused on their home country of Romania.

>  
> “We’re a team of two Romanian hackers and entrepreneurs, passionate about technology and Internet startups. We’ve interned at Google and Microsoft and we’ve kicked ass in programming contests like the [International Olympiad in Informatics](https://en.wikipedia.org/wiki/International_Olympiad_in_Informatics) and [TopCoder](https://topcoder.com/).”  
> <small>[ – Summify Team. “Our Story”](https://summify.com/team/)</small>
> 

In this post I will look at the technology infrastructure they have built for Summify.com, the details of which they were kind enough to share with me.

## Jump In

*   [The Components Of Summify](/summifys-technology-examined#the-components-of-summify)
*   [Technical Challenges](/summifys-technology-examined#technical-challenges)
*   [What’s Cooking Under That Hood?](/summifys-technology-examined#whats-cooking-under-that-hood)

*   [Amazon EC2](/summifys-technology-examined#amazon-ec2)
*   [Nginx](/summifys-technology-examined#nginx)

*   [Multi-line Headers And Malformed Input](/summifys-technology-examined#multi-line-headers-and-malformed-input)
*   [Handling Slow Clients](/summifys-technology-examined#handling-slow-clients)
*   [Shift Your Paradigms](/summifys-technology-examined#shift-your-paradigms)

*   [Gunicorn](/summifys-technology-examined#gunicorn)
*   [Django](/summifys-technology-examined#django)
*   [Tornado](/summifys-technology-examined#tornado)

*   [Tornado Clients](/summifys-technology-examined#tornado-clients)
*   [Swirl](/summifys-technology-examined#swirl)

*   [MongoDB](/summifys-technology-examined#mongodb)
*   [Redis](/summifys-technology-examined#redis)

*   [URL Redirects](/summifys-technology-examined#url-redirects)

*   [MySQL](/summifys-technology-examined#mysql)
*   [jQuery Template](/summifys-technology-examined#jquery-template)
*   [Algorithms](/summifys-technology-examined#algorithms)

*   [Beware Of The Kawasaki](/summifys-technology-examined#beware-of-the-kawasaki)
*   [The Dark-Side Of The Long-Tail](/summifys-technology-examined#the-dark-side-of-the-long-tail)
*   [JSON API](/summifys-technology-examined#json-api)

*   [API Request via HTTP GET](/summifys-technology-examined#api-request-via-http-get)
*   [API Response HTTP Header](/summifys-technology-examined#api-response-http-header)
*   [API Response Body – JSON](/summifys-technology-examined#api-response-body-json)

*   [Next Stop – iPhone](/summifys-technology-examined#next-stop-iphone)
*   [Conclusion](/summifys-technology-examined#conclusion)

<h2 id="the-components-of-summify">The Components Of Summify</h2>

*   It retrieves your __feeds__ from Twitter, Google Reader and Facebook
*   It __crawls__ the __content__ from these feeds, converting Twitter links into full webpages
*   It __cleans__ up the content
*   It __displays__ your content via a streaming webpage
*   A __daily digest__ of your feeds is sent to you by __e-mail__
*   Soon there will be an __iPhone app__

<h2 id="technical-challenges">Technical Challenges</h2>

I see two technical challenges that Summify face…

*   Crawling a large volume of feeds and web pages
*   Live streaming updates to the website

To date Summify has crawled over 200 million web pages. This is fair amount. Even if you are _not_ impressed with this count, then you should understand that they are not in control of this number. They crawl [on-demand](https://en.wikipedia.org/wiki/Print_on_demand). It is not unlikely that their user-base would suddenly spike and they would have to ramp up their volume of crawling quickly. Crawling is easy, but crawling on-demand and being ready to display a re-rendered webpage in near real-time is much harder.

If you read [my post on Quora’s technology](/quoras-technology-examined), then you will have some understanding of the problems faced when displaying live updates to the user’s browser. Summify have similar challenges and they are using many of the same technologies as Quora does, such as Tornado for dealing with the large number of concurrent connections you see when you implement [long-polling](/quoras-technology-examined#long-polling-comet).

<h2 id="whats-cooking-under-that-hood">What’s Cooking Under That Hood?</h2>

<h3 id="amazon-ec2">Amazon EC2</h3>

Summify has approximately 15 different services running behind the scenes. These services are spread across 5 [Amazon EC2](https://aws.amazon.com/ec2/) machines.

<h3 id="nginx">Nginx</h3>

[Nginx](https://nginx.org/en/) is used as the load-balancer and is the first line of defense.

One of Nginx’s jobs is as a [reverse-proxy](https://en.wikipedia.org/wiki/Reverse_proxy) onto [Django](/summifys-technology-examined#django) / [G-Unicorn](/summifys-technology-examined#gunicorn) and [Tornado](/summifys-technology-examined#tornado). As a reverse-proxy, Nginx takes on the job of dealing with multi-line headers, malformed input and handling slow clients. Essentially, Nginx is protecting the application servers from the outside world.

<h4 id="multi-line-headers-and-malformed-input">Multi-line Headers And Malformed Input</h4>

In Tornado’s own [admission](https://www.tornadoweb.org/documentation#caveats-and-support), it does not handle multi-line headers and malformed input very well. Therefore, it is recommended to put Nginx in-front of it to protect it from the gritty outside world. Thanks to Nginx, Tornado only sees clean well-formed HTTP requests.

<h4 id="handling-slow-clients">Handling Slow Clients</h4>

One benefit of using a reverse-proxy, such as Nginx, is that application servers, such as [Gunicorn](https://gunicorn.org/) (or [Unicorn](https://unicorn.bogomips.org/)), are not always designed to handle _slow clients_. If a client, in this case a web-browser, has a slow connection to the Internet then it will take up a lot of the application server’s valuable time. Nginx is designed to handle a huge number of connections concurrently and does not care how fast or slow these connections are. Nginx can buffer the requests and the responses between the client and application server. Only when a request from a client is fully received does the Nginx concern the application server. It can pass this full request quickly back to the application server and can fully receive its response just as quickly. Nginx then begins sending the response back to client. Again, this may take some time if the client’s connection is slow. As far as the application server is concerned it is only ever dealing with fast clients and it can concentrate on efficiently serving those fast clients (again, this is just Nginx). Nginx deals with the real-world, simplifying it for the application server.

Without ensuring that your architecture can handle slow clients you will be wide-open to simple [denial-of-service attacks](https://en.wikipedia.org/wiki/Denial-of-service_attack). [Gunicorn’s website](https://gunicorn.org/deploy.html) recommends using [slowloris](https://ha.ckers.org/slowloris/) for checking that you are handling slow clients correctly.

<h4 id="shift-your-paradigms">Shift Your Paradigms</h4>

If you are wondering why the application server cannot handle slow clients itself, then the answer is… you. Most developers are _not_ accustomed to writing highly performant non-blocking event-driven code. This is how Nginx is built and if you were to [write an Nginx module](https://www.evanmiller.org/nginx-modules-guide.html) then you would have to understand this paradigm. This type of code is harder to develop and maintain. Off-loading the task of handling slow-clients to Nginx is one way that you can make your code simpler and continue to use the more powerful web application frameworks that are not built this way.

One way to make your code easier to read, and not look so _event_ful, is to use an abstraction layer such as [Swirl](https://code.naeseth.com/swirl/) (“some sugar for Tornado”). Summify use Swirl with their Tornado code.

Event-driven frameworks are become more and more popular and it is likely that more web application frameworks will be built on-top these technologies. Some technologies to look at are Tornado, which we will discuss [below](/summifys-technology-examined#tornado), [Node.js](https://nodejs.org/) (Evented I/O for V8 JavaScript) and [Apache Mina](https://mina.apache.org/) (network application framework).

<h3 id="gunicorn">Gunicorn</h3>

[Gunicorn](https://gunicorn.org/) (a.k.a. “Green Unicorn”) is a Python HTTP server in which Summify run the Django web framework. It manages the worker processes. 

>  
> Gunicorn ‘Green Unicorn’ is a Python WSGI HTTP Server for UNIX. It’s a pre-fork worker model ported from Ruby’s Unicorn project. The Gunicorn server is broadly compatible with various web frameworks, simply implemented, light on server resources, and fairly speedy.  
> <small> – gunicorn.org</small>
> 

Gunicorn’s website has some great information on how to [deploy](https://gunicorn.org/deploy.html) Gunicorn with Nginx, in a similar way to how Summify is using this. Cristian points out that the main difference is that they are using TCP sockets, not UNIX sockets.

<del datetime="2011-02-27T18:10:53+00:00">By night the Green Unicorn fights crime with his trusty sidekick Kato.</del>

<h3 id="django">Django</h3>

[Django](https://www.djangoproject.com/) is a Python web application framework. This is used to serve up much of the more trivial aspects of Summify’s website, such as sign-up, login, navigation and serving up the initial Summify-specific HTML pages you see as you navigate around the site.

In Summify’s architecture Django works closely with MySQL.

<h3 id="tornado">Tornado</h3>

[Tornado](https://www.tornadoweb.org/) is an open-source version of [FriendFeed](https://friendfeed.com/)‘s web server and is one of the technologies that powers [Facebook](https://developers.facebook.com/opensource/), [Quora](/quoras-technology-examined) and [many others](https://tornado.poweredsites.org/top).

Tornado is a web server that is ideally suited to handling the dramatic up-tick in concurrent connections you will see when you start playing with “[long polling](/quoras-technology-examined#long-polling-comet)” (comet).

Tornado is able to handle this large volume of connections by being event-driven and non-blocking. It can use the same thread for maintaining many thousands of connections at the same time, since most of these connections are idle at any given time. It multiplexes between the connections, switching to handling the next connection in the queue each time a blocking event occurs (generally disk and network reads and writes). Using [epoll](https://www.kernel.org/doc/man-pages/online/pages/man4/epoll.4.html), the operating system will notify Tornado when it has returned from a blocking event at which point the Tornado server will add an event for the related connection to the queue of events to process.

If you are interested installing Tornado behind Nginx, in the way that Summify have, then [this is good reference](https://www.tornadoweb.org/documentation#running-tornado-in-production).

<h4 id="tornado-clients">Tornado Clients</h4>

Just as Tornado _the server_ is great at keeping connections open for a long time and concurrently handling thousands of clients, Tornado _the client_ is capable of connecting to many servers concurrently.

Just like the server, this is done with non-blocking calls. Your HTTP request code provides a callback to the code to call once the response is received. This way, you can fire-off multiple requests asynchronously.

Here is an example…

```
class MainHandler(tornado.web.RequestHandler):
    @tornado.web.asynchronous
    def get(self):
        http = tornado.httpclient.AsyncHTTPClient()
        http.fetch("https://friendfeed-api.com/v2/feed/bret",
                   callback=self.async_callback(self.on_response))

    def on_response(self, response):
        if response.error: raise tornado.web.HTTPError(500)
        json = tornado.escape.json_decode(response.body)
        self.write("Fetched %d entries from the FriendFeed API" %
            len(json['entries']))
        self.finish()

```

<h4 id="swirl">Swirl</h4>

The above Tornado client example is from [Eric Naeseth’s website](https://code.naeseth.com/swirl/). Eric has created several MIT-licensed open-source projects, such as [Swirl](https://code.naeseth.com/swirl/). Swirl builds on the above concepts to allow you to write asynchronous code using Tornado without using a callback function.

Cristian and Mircea are using Swirl in Summify’s Tornado client code, which they use to crawl the web asynchronously.

<h3 id="mongodb">MongoDB</h3>

[MongoDB](https://www.mongodb.org/) is used to store the news articles after they have been retrieved and reduced. The Summify co-founders have great praise for the technology and the great support they have received from MongoDB’s developers at [10gen](https://www.10gen.com/).

Unfortunately, they have found limitations with the way they are using it. Similar to the problems FourSquare faced with memory page fragmentation, Summify is suffering from ever growing storage issues. Deleting data does not delete a whole page, so the page is not released and the storage size remains the same. Constantly adding and removing data to MongoDB is causing more and more of these fragments that never become removed up. This is similar to the disk fragmentation you will have seen on Windows. Unlike Windows, MongoDB does not come with disk defragmenter.

For this reason they are looking at retiring MongoDB and promoting Redis, which is already part of their architecture, to handle more of their data needs.

>  
> “We have a history of doing great things together. Starting from high-school and later in college we’ve built what is now the biggest Romanian community of Computer Science geeks: [Infoarena](https://infoarena.ro/) is where you train for and compete in programming contests.”  
> <small>[ – Summify Team. “Our Story”](https://summify.com/team/)</small>
> 

<h3 id="redis">Redis</h3>

Redis is used for storing key-values, such as the URL redirects.

<h4 id="url-redirects">URL Redirects</h4>

Nearly every link used on Twitter is a _redirect_ to a longer URL. Different tweeters use different URL-shorteners for the original URL, such as bit.ly, goo.gl, tinyurl, ow.ly or Twitter’s own t.co. Knowing that two URLs ultimately resolve to the same URL is very important for efficiency. Cristian and Mircea have found Redis very good at efficiently handling this data.

<h3 id="mysql">MySQL</h3>

MySQL is used for those situations where [\[database table\] joins](https://en.wikipedia.org/wiki/Join_(SQL)) are required. For instance, _users_ and related user information, such as their feeds (Twitter, Google Reader and Facebook), are stored here.

The list of user feeds in MySQL are used by the _feed retrieval service_ that treats this as queue of jobs to process. Once it reaches the end of the list, it restarts at the beginning in a round-robin manner.

<h3 id="jquery-template">jQuery Template</h3>

Much of site is dynamic. For instance, the news feed is very interactive. Updates are made via AJAX which retrieves new content in the form of [JSON](https://www.json.org/). The JSON is used by JavaScript in the browser to render HTML using [jQuery Templates](https://plugins.jquery.com/project/jquerytemplate) that are pre-loaded.

If you look at the source of a Summify page you will see these templates included there. Here is a snippet of what you might see…

```

<script id="user-tmpl" type="text/x-jquery-tmpl" class="spaceless">
  <div class="top">
    <div class="badge">
      {{if avatar}}
        <span class="avatar"><img src="${avatar}" alt="That's you!" width="32" height="32"/></span>
      {{/if}}

```

I asked Mircea if they had considered any other JavaScript templating languages, such as [Mustache](https://github.com/defunkt/mustache). jQuery Template was the right solution for them, since they wanted the flexibility to use JavaScript in the templates. Mustache is more static in its approach and would limit them, for instance, in formatting of a date-time field.

<h3 id="algorithms">Algorithms</h3>

For Summify to summarize your daily feeds into a single email containing only a handful of stories, they have to algorithmically boil down from the hundreds of stories contained in your feeds. This is a key component to their success and, like Google’s search algorithm, would be constantly evolving.

>  
> The most important inputs we’re using are how many and which of your friends shared an article, (not all your friends are equal) a time decay factor, and popularity across the web.  
> <small> – Cristian Strat, Summify</small>
> 

<h2 id="beware-of-the-kawasaki">Beware Of The Kawasaki</h2>

Cristian and Mircea contacted [Guy Kawasaki](https://www.guykawasaki.com/) to see if he would be interested in testing Summify. As tests go, it was good one. Guy managed to immediately break their system.

Guy Kawasaki’s Twitter feed is unlike most others. As I type this he is currently following [306,014](https://twitter.com/#!/GuyKawasaki/following) other tweeters. Even if each of these tweeters only tweeted once a day, then that is still a lot of tweets and a lot of URLs for Summify to crawl. I am sure Guy’s eyes must get sore from trying to keep up with his Twitter feed.

While they were humbled in finding Summify’s limitations, Cristian and Mircea decided it would be best to cap the number of URLs to 5000 per user. Not all tweets contain URLs, so this is not the limit on the number of tweets. This limit also covers content from Google Reader and the Facebook feed. It is unlikely that anyone is capable of reading more than 5000 articles per day.

<h2 id="the-dark-side-of-the-long-tail">The Dark-Side Of The Long-Tail</h2>

Everyone loves the [long-tail](https://www.longtail.com/about.html). That is were the majority the users hang-out and if we can tap into the long-tail’s raw potential then we will find untold wealth. The pot of gold is no longer at the end of the rainbow. Instead, that pot of gold has relocated to the end of the long-tail.

>  
> __[“The Long Tail, in a nutshell”](https://www.longtail.com/about.html)__
> 
> The theory of the Long Tail is that our culture and economy is increasingly shifting away from a focus on a relatively small number of “hits” (mainstream products and markets) at the head of the demand curve and toward a huge number of niches in the tail. As the costs of production and distribution fall, especially online, there is now less need to lump products and consumers into one-size-fits-all containers. In an era without the constraints of physical shelf space and other bottlenecks of distribution, narrowly-targeted goods and services can be as economically attractive as mainstream fare.
> 
>  – [Chris Anderson](https://en.wikipedia.org/wiki/Chris_Anderson_(writer))
> 

For Summify, and similar companies, the long-tail means more work and more processing power. 80% of the RSS feeds that Summify crawls only have a one or two subscribers. While this can partially be put down to the fact that Summify is still far from _[critical mass](https://en.wikipedia.org/wiki/Critical_mass_(sociodynamics))_, if the theory of the long-tail holds true, then this will be an ongoing problem. It is the way in which start-ups creatively address problems like this that defines the successful start-ups and ensures their technology remains stable as they reach [escape velocity](https://en.wikipedia.org/wiki/Escape_velocity).

<h2 id="json-api">JSON API</h2>

I will not go into details of the complete API, 1) because I do not have it and 2) because this is just for Summify’s internal use and Cristian and Mircea might know where I live. Instead this is intended as a look at how a JSON API for a dynamically updating website might be implemented, rather than a reference to the API.

<h3 id="api-request-via-http-get">API Request via HTTP GET</h3>

This request is made when I click the 

<img height="162" src="https://commondatastorage.googleapis.com/philwhln/blog/images/summify_technology/summify_click_more.png" width="500"/>

```
GET /api/recent.json?count=20&order_lt=1298761351.0000%2C1328%2C0000%2C3f4d HTTP/1.1
Host: summify.com
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.6; en-US; rv:1.9.2.13) Gecko/20101203 Firefox/3.6.13
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-us,en;q=0.5
Accept-Encoding: *;q=0
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 115
Connection: keep-alive
X-Requested-With: XMLHttpRequest
Referer: https://summify.com/client
Cookie: csrftoken=b781c99a21bfa6ec66f73576ece73b3b; sessionid=bdb49da344d0be4659534fd2d8e11872; __utma=133505548.1812117490.1298762364.1298762364.1298762923.2; __utmb=133505548.2.10.1298762923; __utmz=133505548.1298762364.1.1.utmcsr=(direct)|utmccn=(direct)|utmcmd=(none); __utmc=133505548
```

You will notice that the request basically just comprises of the following…

```
count=20
```

“Get 20 more articles”.

```
order_lt=1298761351.0000,1328,0000,3f4d    (URL encoded)
```

This is order id associated with oldest article we have on the screen. So we are requesting to get any newer articles.

_1298761351_ is the current date-time as an [Unix time](https://en.wikipedia.org/wiki/Unix_time) (number of seconds elapsed since January 1, 1970). The server can use this to check if there are new updates to send back to the browser.

Note that I have disabled response encoding in my request. Generally your browser will request “gzip,deflate” encoding using the “Accept-Encoding” HTTP header. I have [disabled this in Firefox](https://forgetmenotes.blogspot.com/2009/05/how-to-disable-gzip-compression-in.html), since I want to be able to more easily see the content returned.

<h3 id="api-response-http-header">API Response HTTP Header</h3>

```
HTTP/1.1 200 OK
Server: nginx/0.7.67
Date: Sat, 26 Feb 2011 23:29:21 GMT
Content-Type: application/json
Connection: keep-alive
Vary: Accept-Encoding
Content-Length: 23289
Etag: "4928161e5bbc814b22c37e82b83820b4df611de7"
```

The response header is pretty straight forward. It specifies “keep-alive” to maintain the HTTP connection for future requests. We can also see that 23Kb of content is returned to us in the body of response. This is actually larger than would be transferred to you since I have disabled content encoding.

<h3 id="api-response-body-json">API Response Body – JSON</h3>

The content of the response is [JSON encoded](https://www.json.org/). I have prettified the JSON here, since this data is actually sent as a single line of JSON and is not so easy on the eyes. Also, your browser will request gzip encoding. If you can read un-prettified gzip-encoded JSON then I take my hat off to [you](https://matrix.wikia.com/wiki/Tank).

```
[
   {
      "article_id" : "4d69875eaf55ef771200304d",
      "published_at" : "2011-02-26T23:02:31Z",
      "marked_read" : false,
      "keep_unread" : false,
      "global_reactions" : {
         "twitter" : {
            "tweet_count" : 0
         },
         "facebook" : {
            "click_count" : 0,
            "comment_count" : 0,
            "like_count" : 0,
            "share_count" : 0,
            "total_count" : 0
         }
      },
      "image" : {
         "width" : 250,
         "url" : "https://www.linglom.com/images/Linux/ChangeIPAddress/_1.png",
         "height" : 233
      },
      "sources" : [
         {
            "user_avatar" : "https://a2.twimg.com/profile_images/1153016992/180_warau_normal.jpg",
            "published_at" : "2011-02-26T23:02:31Z",
            "tweet_id" : "41634242127470592",
            "user_name" : "junichi_y",
            "user_screen_name" : "junysb3",
            "generator" : "twitter",
            "service_username" : [
               "philwhln"
            ],
            "text" : "How to change IP Address on Linux Redhat | Linglom.com:  https://bit.ly/dZpYOg",
            "user_id" : 81482943
         }
      ],
      "order" : "1298761351.0000,0592,0000,b52a",
      "article" : "4d69875eaf55ef771200304d",
      "url" : {
         "domain" : "www.linglom.com",
         "full" : "https://www.linglom.com/2008/04/20/how-to-change-ip-address-on-linux-redhat/"
      },
      "title" : "How to change IP Address on Linux Redhat"
   },

```

This shows the first of 20 records returned and comprises of the various attributes that are displayed on the screen for this article. You can see the title, url, image, sources, article id (unique to Summify) and “order” attribute. The last “order” id is used as the next request to the API. For instance, if this was the last article returned then the next request for “More” would be…

```
GET /api/recent.json?count=20&order_lt=1298761351.0000%2C0592%2C0000%2Cb52a HTTP/1.1
```

<h2 id="next-stop-iphone">Next Stop – iPhone</h2>

Next on the cards is the iPhone application. As you saw above, JSON is used to update Summify’s website interface. The JSON API they have built can easily be utilized from within an iPhone application.

Reading blog posts on-the-go is difficult. This is mainly due to the way in which each blog is formatted differently. Blogs are often not formatted for use on mobile devices. Since Summify strips out the formatting and leaves just the content they can easily re-format that content to better fit the mobile device in a consistent way. 

>  
> “We’ve turned down some cozy job offers from companies like Google and Facebook and we moved half-way across the world for this incredible opportunity.”  
> <small>[ – Summify Team. “Our Story”](https://summify.com/team/)</small>
> 

<h2 id="conclusion">Conclusion</h2>

Summify is not scared to try different technologies within their architecture and they appear very agile in their approach, while building a solid foundation to take Summify to the next level.

When I wrote the previous post on [Quora’s technology](/quoras-technology-examined), I had a strong sense that Quora knew exactly what they doing. Summify is using many of the same technologies. In fact, they showed interest in Quora’s internally developed technologies, such as [LiveNode](/quoras-technology-examined#webnode2-and-livenode). Hopefully LiveNode will become released as open-source, benefiting many more companies like Summify.

I think we will be hearing much more from Summify’s founders, Cristian and Mircea. They have a great long-standing partnership and both understand the technology they are using very well.

At the time of writing this Summify has aggregated more than 200 million stories.

I intend to keep following Summify’s progress. I will be writing more about them and [the technology of other similar tech companies in future blog posts](/feed).

If you found this post useful then please [__leave a comment below__](/summifys-technology-examined#comments-template) or [__follow this blog__](/feed).

<h2 id="recommended-reading">Recommended Reading</h2>

<object classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" codebase="https://fpdownload.macromedia.com/get/flashplayer/current/swflash.cab" height="200px" id="Player_fbe332d7-592b-4c06-975a-967ba64d98db" width="600px"> 

<param name="movie" value="https://ws.amazon.com/widgets/q?ServiceVersion=20070822&amp;MarketPlace=US&amp;ID=V20070822%2FUS%2Fgetafil-20%2F8010%2Ffbe332d7-592b-4c06-975a-967ba64d98db&amp;Operation=GetDisplayTemplate"/>

<param name="quality" value="high"/>

<param name="bgcolor" value="#FFFFFF"/>

<param name="allowscriptaccess" value="always"/>

<embed align="middle" allowscriptaccess="always" bgcolor="#ffffff" height="200px" id="Player_fbe332d7-592b-4c06-975a-967ba64d98db" name="Player_fbe332d7-592b-4c06-975a-967ba64d98db" quality="high" src="https://ws.amazon.com/widgets/q?ServiceVersion=20070822&amp;MarketPlace=US&amp;ID=V20070822%2FUS%2Fgetafil-20%2F8010%2Ffbe332d7-592b-4c06-975a-967ba64d98db&amp;Operation=GetDisplayTemplate" type="application/x-shockwave-flash" width="600px"/>

</object> 

<noscript><a href="https://ws.amazon.com/widgets/q?ServiceVersion=20070822&amp;MarketPlace=US&amp;ID=V20070822%2FUS%2Fgetafil-20%2F8010%2Ffbe332d7-592b-4c06-975a-967ba64d98db&amp;Operation=NoScript">Amazon.com Widgets</a></noscript>

In this blog post I will delve into the snippets of information available on Quora and look at Quora from a technical perspective. What technical decisions have they made? What does their architecture look like? What languages and frameworks do they use? How do they make that search bar respond so quickly?

<div id="comments">
  <h3 id="comments-number" class="comments-header">9 responses to “Summify’s Technology Examined”</h3>
  <ol class="comment-list">
    <li id="comment-1317" class="comment byuser comment-author-mirceapasoi even thread-even depth-1 comment role-subscriber user-mirceapasoi">
      <img alt="Mircea Paşoi" src="https://1.gravatar.com/avatar/9fda40028c6fd432f278c5d2aa2e9daf?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.facebook.com/mirceapasoi">Mircea Paşoi</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, March 7th, 2011, 6:11 pm">March 7, 2011</abbr> at <abbr class="comment-time" title="Monday, March 7th, 2011, 6:11 pm">6:11 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Great article, thanks for the review!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-1338" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Jerry Tian" src="https://0.gravatar.com/avatar/454101ac9da0a02fd2b95725bb24483f?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://learningfy.com">Jerry Tian</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, March 7th, 2011, 8:59 pm">March 7, 2011</abbr> at <abbr class="comment-time" title="Monday, March 7th, 2011, 8:59 pm">8:59 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Phil, thanks a lot for the great article.<br />
To the Summify guys, interesting use of Redis. How much data is Redis storing currently? It sounds like a lot data to keep for a in memory data store. Is Redis coping well? How many Redis clusters do you have?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-1348" class="comment byuser comment-author-mirceapasoi even depth-2 comment role-subscriber user-mirceapasoi">
          <img alt="Mircea Paşoi" src="https://1.gravatar.com/avatar/9fda40028c6fd432f278c5d2aa2e9daf?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.facebook.com/mirceapasoi">Mircea Paşoi</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, March 7th, 2011, 11:09 pm">March 7, 2011</abbr> at <abbr class="comment-time" title="Monday, March 7th, 2011, 11:09 pm">11:09 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Jerry,</p>
            <p>We’re storing about 8-9GB of data in Redis currently and it’s coping well so far. We have 4 Redis instances on the same machine, each with different data sets and with different access patterns (between 10 and 1000 requests/second)</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-1346" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Jesse Heaslip" src="https://1.gravatar.com/avatar/3a7c07f7086cd879e8886586fe2a95b9?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Jesse Heaslip</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, March 7th, 2011, 10:56 pm">March 7, 2011</abbr> at <abbr class="comment-time" title="Monday, March 7th, 2011, 10:56 pm">10:56 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Great read Phil! Always interesting to have a peek under the hood of a product like this. Thanks for a thorough post!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-1552" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="XU Qiang" src="https://1.gravatar.com/avatar/70e2c152a34c9a1ed77076fa1bf12341?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">XU Qiang</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, March 15th, 2011, 8:40 am">March 15, 2011</abbr> at <abbr class="comment-time" title="Tuesday, March 15th, 2011, 8:40 am">8:40 am</abbr>
      </div>
      <div class="comment-text">
        <p>i find ur blog really helpful, thanks a lot~~~</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-3556" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Alex Toulemonde" src="https://0.gravatar.com/avatar/6b02975aaf1cab1a97d696fa5b822c88?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://hockey-community.com">Alex Toulemonde</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, July 2nd, 2011, 12:02 am">July 2, 2011</abbr> at <abbr class="comment-time" title="Saturday, July 2nd, 2011, 12:02 am">12:02 am</abbr>
      </div>
      <div class="comment-text">
        <p>Wow I miss this article. Great one. I knew Cris and Mircea were awesome but not as much <img src="/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />  Thanks for taking the time to break their technology down. Great learning. </p>
        <p>PS: Go Vancouver Startups Go!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-3942" class="pingback even thread-odd thread-alt depth-1 pingback reader">
      <img alt="Downtime Postmortem | Summify" src="https://0.gravatar.com/avatar/?d=https://www.bigfastblog.com/css/library/images/pingback.png&amp;s=80" class="avatar avatar-80 photo avatar-default" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://newblog.summify.com/2011/03/19/downtime-postmortem/">Downtime Postmortem | Summify</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, July 28th, 2011, 12:25 pm">July 28, 2011</abbr> at <abbr class="comment-time" title="Thursday, July 28th, 2011, 12:25 pm">12:25 pm</abbr>
      </div>
      <div class="comment-text">
        <p>[...] Yesterday, in order to cope with the increased volume of links we added more crawler processes on one of our machines. (more details about our infrastructure) [...]</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-5297" class="pingback odd alt thread-even depth-1 pingback reader">
      <img alt="Summify’s Technology Examined « Another Word For It" src="https://0.gravatar.com/avatar/?d=https://www.bigfastblog.com/css/library/images/pingback.png&amp;s=80" class="avatar avatar-80 photo avatar-default" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://tm.durusau.net/?p=17356">Summify’s Technology Examined « Another Word For It</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, November 2nd, 2011, 4:25 pm">November 2, 2011</abbr> at <abbr class="comment-time" title="Wednesday, November 2nd, 2011, 4:25 pm">4:25 pm</abbr>
      </div>
      <div class="comment-text">
        <blockquote>
          <p>May just be me but I would think the semantics of the feeds would rank pretty high. Both in terms of recognition of items of interest in terminology familiar to the user as well as new terminology. For example, what if I say I wants feeds on P2P systems, an information overload reducing application would also give me distributed network entries.</p>
          <p>That’s an easy example but you get the idea. And the system should do that across different interests of users and update its recognition of relevant items to include new terminology as it emerges.</p>
          <p>BTW, you might want to check out the Summify FAQ on how they determine your interests.</p>
        </blockquote>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-12229" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Matt Smith" src="https://1.gravatar.com/avatar/bc748c8ae72f5f4e6874186d504ff1f3?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.thinkific.om">Matt Smith</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, April 6th, 2012, 6:08 pm">April 6, 2012</abbr> at <abbr class="comment-time" title="Friday, April 6th, 2012, 6:08 pm">6:08 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Super thorough article, thanks!  Couldn’t stop laughing at the reference to Tank (https://matrix.wikia.com/wiki/Tank).</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

