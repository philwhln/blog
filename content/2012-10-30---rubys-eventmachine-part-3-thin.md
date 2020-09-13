---
title: "Ruby’s EventMachine – Part 3 : Thin"
date: "2012-10-30T14:26:00-07:00"
template: "post"
draft: false
slug: "rubys-eventmachine-part-3-thin"
category: "EventMachine"
tags:
  - event-based
  - event-loop
  - eventmachine
  - rack
  - ruby
  - ruby on rails
  - thin
  - web server
  - web-development
description: "As mentioned in part 1 and part 2 of this series on Ruby's EventMachine, Thin is where most folks encounter EventMachine for the first time, even if they do not realize it. EventMachine is at the core of Thin and allows for the high concurrency that Thin provides to your Rails application. In this post I will look at Thin's usage of EventMachine."
socialImage:
  publicURL: "/media/images/photo.jpg"
---
As mentioned in [part 1](/rubys-eventmachine-part-1-event-based-programming) and [part 2](/ruby-eventmachine-part-2-asynchronous-not-equal-faster) of this [series on Ruby’s EventMachine](/tag/eventmachine), Thin is where most folks encounter EventMachine for the first time, even if they do not realize it. EventMachine is at the core of Thin and allows for the high concurrency that Thin provides to your Rails application. In this post I will look at Thin’s usage of EventMachine.

## Thin?

Let’s start at the basics.

>  
> Thin is a Ruby web server that glues together 3 of the best Ruby libraries in web history.  
> <small> – Thin homepage</small>
> 

The above quote refers to Mongrel, EventMachine and Rack. I am not going cover Mongrel and Rack.

## Up And Running With Thin

Quickly, let’s see what it takes to get up and running with Thin.

Create your Rails application.

```
rails new thinapp
cd thinapp
```

Add the “thin” gem to your application.

```
echo "gem 'thin'" >> Gemfile
bundle update
```

Start your application, running under Thin.

```
thin start
>> Using rack adapter
>> Thin web server (v1.5.0 codename Knife)
>> Maximum connections set to 1024
>> Listening on 0.0.0.0:3000, CTRL+C to stop
```

And just like that you are now up and running with Thin and your code is running inside the EventMachine event-loop.

## Concurrent Requests

Even if you make no changes to your application, it will receive several benefits.

You can now handle slow clients and large uploads without blocking your Rails application. This is huge! As discussed in part 1 of this EventMachine series, one slow client will hog your whole process if you do not have a proxy server in front of your application for processing slow clients.

Without Thin, your Rails application process will sit there patiently waiting for that one user to upload a 4Gb video using his [1964 antique modem](https://www.youtube.com/watch?v=X9dpXHnJXaE). All your other users will either have to use another Rails process or wait patiently for their turn at dedicated usage of your Rails process.

With Thin, the HTTP requests are multi-plexed. Our single process ignores a request until a chunk of data comes in on that socket. When a chunk of data is received, Thin receives a notification from EventMachine and Thin adds the current chunk of data to the accumulated request. If the request is complete, then your application is passed the complete request. If the request is not complete then we simply wait until another chunk of data to come in from this or other request sockets

This handling of chunks of data from client sockets is very fast. The data is already there when the socket event fires, so the data just needs to be processed. This means that Thin / EventMachine can process a large number chunks of data from many different HTTP request sockets in a small amount of time. We are only CPU-bound here.

## Many Connections == Much Memory

You might have realized that this multiplexing could become expensive in terms of RAM.

It is great that one guy with a slow modem can upload a large video without affecting your other user requests, but what happens when he uploads 10 videos concurrently? What happens when you have a thousand users all uploading 4Gb videos concurrently?

It is an extreme example, but receiving (1000 x 4Gb) 4 terabytes of data concurrently is going cause you a few headaches. You might have 64Gb of RAM and 500 terabytes of disk space, but if your application does not see those 4Gb requests until we have the full request from Thin, then how are we going to fit all those partially complete requests in RAM at the same time?

## Disk To The Rescue!

Luckily, Thin uses the local disk as middle-man. If the content-length of a request is over 112Kb then Thin writes the content to a temporary file. As each chunk of data arrives on each HTTP request socket then Thin writes it out to a temporary file rather than keeping it in memory.

This means that your 4Gb request is living on disk instead of in memory, so receiving a thousand 4Gb uploads concurrently now becomes feasible with a single Rails process if we have the network bandwidth to support it and fast enough disks.

## Not Perfect

My complaint of the spooling requests to disk is that Thin is not doing this in an event-based way.

It looks a little like this…

```ruby
@body = Tempfile.new('thin-body')
```

```ruby
# for each chunk of data received
@body << data

```

Each write of a chunk of request data to disk will block the EventMachine event-loop ("reactor") and the duration of this blocking will depend on your disk IO latency.

[StackOverflow : How to write (large) files with Ruby Eventmachine](https://stackoverflow.com/questions/4645761/how-to-write-large-files-with-ruby-eventmachine)

## Pushing Upstream

Using Nginx instead of your Rails application to handle slow connections is wise move. Nginx is highly optimized for this and is written in C. If you want to understand how to write highly optimized C code, then browsing through the Nginx code base is for you.

Nginx can spool up requests and pass a more complete HTTP request to your Rails application in one swift transaction. Fewer (bigger) chunks of data being received by your Rails application will result in less concurrency inside your Rails application. This will mean Thin has to do less work. Less work is good and your application's process has more cycles to do the _application_ logic it is designed for.

Nginx, out-of-the-box, will not handle file uploads for you, but there is a [module that you can install](https://www.grid.net.ru/nginx/upload.en.html) for that, which simply passes the file path of temporary path to your Rails application.

## EventMachine "Live Edition"

I mention this here, since Thin version 2 will be using "[eventmachine-le](https://github.com/ibc/EventMachine-LE)" instead of "[eventmachine](https://github.com/eventmachine/eventmachine)".

See [commit 084197d : "Use EM live edition gem."](https://github.com/macournoyer/thin/commit/084197daa0fa7b3d0662679fe4d65c4f6273ecd5)

EventMachine-LE is a fork (although it likes to be thought of as a "branch") of EventMachine. It is a drop-in replacement that incorporates many of the pull requests that have been not been merged by the core EventMachine team.

>  
> This branch incorporates interesting pull requests that are not yet included in the mainline EventMachine repository. The maintainers of that version prefer to minimize change in order to keep the stability with already existing EventMachine deployments, which provides an impressive multi-platform base for IPv4 TCP servers (e.g., Web servers) that don't need good UDP or IPv6 support.  
> <small> - EventMachine-LE README.md</small>
> 

One of the authors of this is [Iñaki Baz Castillo](https://github.com/ibc), who was also featured in a early blog-post I wrote, [Zero-Copy. Transfer Data Faster In Ruby](/zero-copy-transfer-data-faster-in-ruby).

## Comments

<div id="comments">
  <ol class="comment-list">
    <li id="comment-24689" class="comment even thread-even depth-1 comment reader">
      <img alt="Martinos" src="https://0.gravatar.com/avatar/ed77c0d1c8da409c4f69f67f934ff0bb?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Martinos</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, November 2nd, 2012, 11:50 am">November 2, 2012</abbr> at <abbr class="comment-time" title="Friday, November 2nd, 2012, 11:50 am">11:50 am</abbr>
      </div>
      <div class="comment-text">
        <p>Great article,<br />
It’s the best article that I have found about Thin. In every article that I have read, they say that Thin is faster than many other servers because it uses EventMachine and threads but they never say why. But now I know that it’s faster for pumping clients requests.</p>
        <p>Do you know if it’s the only une of EventMachine in Thin? if it’s the case, Nginx + unicorn might be faster that Thin for handling the buffering of http request.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

