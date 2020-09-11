---
title: "Ruby’s EventMachine – Part 1 : Event-based Programming"
date: "2012-09-14T15:19:00-07:00"
template: "post"
draft: false
slug: "rubys-eventmachine-part-1-event-based-programming"
category: "EventMachine"
tags:
  - apache
  - epoll
  - eventlib
  - eventmachine
  - ioloop
  - nginx
  - perl
  - poe
  - python
  - ruby
  - ruby on rails
  - thin
  - tornado
description: "In this first post, of a series on Ruby's EventMachine, I will introduce EventMachine and explain why event-based programming is good for your wallet. EventMachine, which just turned 1.0.0 this week, is more than just a gem, it is a new paradigm for many Ruby programmers and is not always easy to just drop into your existing stack. As the name suggests, it gives you event-based programming."
socialImage:
  publicURL: "/media/images/photo.jpg"
---
In this first post, of a series on Ruby’s [EventMachine](https://github.com/eventmachine/eventmachine), I will introduce EventMachine and explain why event-based programming is good for your wallet.

EventMachine, which just turned [1.0.0](https://rubygems.org/gems/eventmachine/versions/1.0.0) this week, is more than just a gem, it is a new paradigm for many Ruby programmers and is not always easy to just drop into your existing stack. As the name suggests, it gives you [event-based programming](https://en.wikipedia.org/wiki/Event-driven_programming).

I have been doing event-based programming for several years now, in Perl ([Event::Lib](https://search.cpan.org/~vparseval/Event-Lib-1.03/lib/Event/Lib.pm) and [POE](https://poe.perl.org/)), Python ([Tornado](https://www.tornadoweb.org/documentation/ioloop.html)), C ([nginx](https://trac.nginx.org/nginx/wiki)) and now Ruby ([EventMachine](https://github.com/eventmachine/eventmachine)) and I think it is _the_ way to get the most out of your server’s resources.

## Apache vs Nginx

For the sake of example, I’m going to side-step and look at a switch that you may have already made, from Apache to Nginx. If you have switched over from Apache to Nginx and thought, “hey, this thing has a much smaller footprint than Apache”, then you have touched one the main of benefit of Nginx… it is built using event-based programming.

With Apache, if a new client connects, you need a new process, or at least use of process that another client is not using. If the client is slow in sending its HTTP request, then your process will be consumed until that client has finished its role in the request-response cycle. When other client connections come in, they either have sit in a queue, waiting, or new Apache processes are fired up (or used from a pool) to handle the connections.

Therefore, with Apache, your throughput is governed by the number of Apache processes you can run simultaneously divided by the average time it takes to process requests. The number of Apache processes you can run simultaneously is dependant on the RAM you have on your machine and RAM is expensive. The average time it takes to process requests is partly dependant on you (how long does your application server take to process the requests) and your users (how slow are they sending and receiving the HTTP data). The second one, your users, is _not_ under your control. Anytime you hit things that are not under your control and affect your systems performance, you are on shaky ground and you make denial-of-service attacks so much easier.

When Nginx receives a new client connect it simply creates a new socket and not much more. Therefore, the same process can potentially [handle tens of thousands requests](https://www.kegel.com/c10k.html). Instead of blocking the process, while waiting for data to be sent or received from the socket, it creates a callback and says to the operating system, “hey, let me know when some more data comes in on this socket. I’ve got other things to do”. By doing this the process is free to do anything else until something new happens with that socket. In this way, Nginx’s performance is unaffected by a few slow clients. Whether a client is fast or slow, Nginx uses approximately the same system resources, whereas Apache is holding onto resources that might be in a dormant state.

## Out-Of-The-Box Rails

Out of the box, Rails is much like Apache. It is a single process and any work you do, such as communicating with the client over HTTP or talking to database, is done in a blocking manner. The process sits and waits for I/O responses and nothing else can get in during that time.

## Thin

[Thin](https://github.com/macournoyer/thin/) is a Ruby web application server built by [Marc-André Cournoyer](https://macournoyer.com/). Thin is built using EventMachine. Things like [client connections](https://github.com/macournoyer/thin/blob/master/lib/thin/connection.rb) are handled in an event-based manner. Therefore, if you have used Thin, you have used EventMachine.

## Stop Blocking Thin!

If you have unwittingly used EventMachine, via Thin, then it is likely that you have also severely degraded Thin’s potential performance. 

Within an event-based application, everything must be non-blocking to get the real benefit of event-based programming. The event-loop is running in a single thread within a single process. Therefore, the first time you do something that is blocking, it causes the whole event loop to stop and wait for you. That means while you are talking to you database for “just 10 milliseconds”, nothing else happens. You would be surprised at how many operations you can perform in those 10ms of halting your entire event loop. In that time, potentially hundreds of connections, that only hit RAM, could have come and gone, but instead they get queued waiting for your blocking action.

## The Perfect Event Loop

The perfect event-loop is one which passes every blocking request to the operating system, via the event library. This basically means any time you touch the disk or a network network, but also includes talking other processes.

Achieving a perfectly non-blocked event-loop is hard in practice. The packages you generally use are not designed this way, and finding event-friendly alternatives is hard. This is the biggest issue I have hit with every language I done event-based programming in.

## Good For My Wallet?

As I mentioned briefly at the beginning of the post, event-based programming is good for your wallet. This is due to the fewer processes you need to run and how much you can squeeze out of each process. Less processes equals less RAM and RAM is the thing you will often find is the biggest cost in firing up servers. This is especially true if you work with [IaaS](https://en.wikipedia.org/wiki/Infrastructure_as_a_service#Service_models) archiectures such [Amazon EC2](https://aws.amazon.com/) or [Heroku](https://www.heroku.com/).

## Going Deeper…

This is first of several blog posts I will be writing on EventMachine. In future posts, I’ll be hitting on topics such as talking to your data-store, connecting to 3rd party services (such as Twitter), shelling out to other processes and running background tasks. Most importantly, how to do all this in a non-blocking event-based way, using EventMachine inside you Rails application.

If there is a particular topic that you are interested in, then please leave a comment below.

## Ruby’s EventMachine

Other posts in this series

*   [Part 1 : Event-based Programming](/rubys-eventmachine-part-1-event-based-programming)
*   [Part 2 : Asynchronous != Faster](/ruby-eventmachine-part-2-asynchronous-not-equal-faster)

<div id="comments">
  <h3 id="comments-number" class="comments-header">9 responses to “Ruby’s EventMachine – Part 1 : Event-based Programming”</h3>
  <ol class="comment-list">
    <li id="comment-21958" class="comment even thread-even depth-1 comment reader">
      <img alt="none" src="https://1.gravatar.com/avatar/79cf3edd02da25d9fcf0440f568972ed?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.none.com.ua">none</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, September 20th, 2012, 6:54 am">September 20, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 20th, 2012, 6:54 am">6:54 am</abbr>
      </div>
      <div class="comment-text">
        <p>Мery interesting topic.<br />
I hope that the new posts will appear soon enough….</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-21959" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Phil Pirozhkov" src="https://1.gravatar.com/avatar/bb3cf80b7af6b40db1b3b2445ee738a1?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Phil Pirozhkov</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, September 20th, 2012, 7:04 am">September 20, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 20th, 2012, 7:04 am">7:04 am</abbr>
      </div>
      <div class="comment-text">
        <p>Thanks for the writing.<br />
Will be very interesting to hear in next parts:<br />
 – EventMachine 2.0 rewrite<br />
 – why the hell EM uses threads (deferrables)<br />
 – eventmachine-le and why eventmachine doesn’t accept pull requests<br />
 – eventmachine vs libuv comparison</p>
        <p>I think the rest is covered already in Tutorials already (https://github.com/eventmachine/eventmachine/wiki/Tutorials)</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-21961" class="comment even thread-even depth-1 comment reader">
      <img alt="Gilbert Beveridge" src="https://1.gravatar.com/avatar/572376aaf246f870ecf00b7f17a8105d?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Gilbert Beveridge</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, September 20th, 2012, 9:52 am">September 20, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 20th, 2012, 9:52 am">9:52 am</abbr>
      </div>
      <div class="comment-text">
        <p>I am wondering what benefits could be gained for test automation with event based programming? Running cucumber,rspec,unit tests, selenium tests etc. Would it help with concurrency and timing issues ? i.e. waiting for the application to respond. waiting for a tweet, comment?</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-21993" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Peter Lane" src="https://0.gravatar.com/avatar/6d43da5df0e52fc31c55e6a227bb23f4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://rubyprogramming.net/">Peter Lane</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, September 21st, 2012, 3:56 am">September 21, 2012</abbr> at <abbr class="comment-time" title="Friday, September 21st, 2012, 3:56 am">3:56 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hey, relativ guter und klar strukturierter Beitrag über Ruby programming. War für mich neu. Danke! Ciao, Peter Lane.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-22061" class="comment byuser comment-author-kryptykphysh even thread-even depth-1 comment role-subscriber user-kryptykphysh">
      <img alt="Kryptyk Physh" src="https://1.gravatar.com/avatar/15d5c93f2d5cce3488f985a5a3c946fc?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://plus.google.com/105358230357528729129">Kryptyk Physh</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, September 22nd, 2012, 5:18 pm">September 22, 2012</abbr> at <abbr class="comment-time" title="Saturday, September 22nd, 2012, 5:18 pm">5:18 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Cool! EventMachine is one of those things I’ve been keeping in the corner of my eye for a while, but found the learning curve a bit steep. Any ski-poles you can lend would be greatly appreciated. <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
        </p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-22288" class="comment byuser comment-author-lanceolsen odd alt thread-odd thread-alt depth-1 comment role-subscriber user-lanceolsen">
      <img alt="Lance Olsen" src="https://0.gravatar.com/avatar/c9dd4461ec1dcffe2c2f02b255abe1c6?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.facebook.com/lance.olsen.58">Lance Olsen</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, September 27th, 2012, 12:56 pm">September 27, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 27th, 2012, 12:56 pm">12:56 pm</abbr>
      </div>
      <div class="comment-text">
        <p>The fact that all the gems I use probably have tons of blocking operations makes the idea of trying to implement event-based programming seem kind of like spitting in the wind. Is there really hope?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-22445" class="comment even depth-2 comment reader">
          <img alt="Phil Pirozhkov" src="https://1.gravatar.com/avatar/bb3cf80b7af6b40db1b3b2445ee738a1?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">Phil Pirozhkov</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, September 30th, 2012, 10:45 am">September 30, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 30th, 2012, 10:45 am">10:45 am</abbr>
          </div>
          <div class="comment-text">
            <p>Which gems exactly?</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-22325" class="comment byuser comment-author-jzinedine odd alt thread-even depth-1 comment role-subscriber user-jzinedine">
      <img alt="Jahangir Zinedine" src="https://0.gravatar.com/avatar/87f4948e30f52a367ea83c307be11231?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://plus.google.com/118363599612356884984">Jahangir Zinedine</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, September 28th, 2012, 2:24 am">September 28, 2012</abbr> at <abbr class="comment-time" title="Friday, September 28th, 2012, 2:24 am">2:24 am</abbr>
      </div>
      <div class="comment-text">
        <p>Thanks, I’d like to see parallel programming paradigms at the start at least just at the topic level.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-24424" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Ram" src="https://1.gravatar.com/avatar/fa767aff37fcc04b8923e3fe7163c0f3?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Ram</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, October 31st, 2012, 12:30 am">October 31, 2012</abbr> at <abbr class="comment-time" title="Wednesday, October 31st, 2012, 12:30 am">12:30 am</abbr>
      </div>
      <div class="comment-text">
        <p>Great read, thanks for posting. Will check out other posts in the series.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

