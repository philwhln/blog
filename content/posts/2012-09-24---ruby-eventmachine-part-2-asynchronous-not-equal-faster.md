---
title: "Ruby’s EventMachine – Part 2 : Asynchronous != Faster"
date: "2012-09-24T13:19:00-07:00"
template: "post"
draft: false
slug: "ruby-eventmachine-part-2-asynchronous-not-equal-faster"
category: "EventMachine"
tags:
  - asynchronous
  - benchmark
  - event-loop
  - eventmachine
  - fast
  - memcached
  - ruby
  - synchronous
  - tornado
description: "In this post I will look synchronous vs asynchronous programming with Ruby's EventMachine, to show that asynchronous does not always mean that your code will run faster. In part 1 of this series on Ruby's EventMachine I discussed the benefits of event-based programming in general. I am a big fan of event-based programming, as you will see in these posts, but I wanted to flip the coin over and look at one of the down-sides of event-based programming."
socialImage:
  publicURL: "/media/images/photo.jpg"
---
In this post I will look synchronous vs asynchronous programming with Ruby’s [EventMachine](https://github.com/eventmachine/eventmachine), to show that asynchronous does not always mean that your code will run faster.

In [part 1](/rubys-eventmachine-part-1-event-based-programming) of this series on Ruby’s EventMachine I discussed the benefits of event-based programming in general. I am a big fan of event-based programming, as you will see in these posts, but I wanted to flip the coin over and look at one of the down-sides of event-based programming. 

## The Cost

Managing events does not come for free. There is overhead of wrapping code in callbacks, stashing context, queuing events, deleting events, managing timer events and communication the the operating system.

## Exhibit A

With the example I am using for this post, we talk over a TCP network connection and perform a high number of transactions in a short period of time. If you read [part 1](/rubys-eventmachine-part-1-event-based-programming), then you will know that this sounds like an ideal use-case for EventMachine to really shine.

The example I am going to use is Memcached. Memcached is fast. If you have a low-latency network connection to Memached, then it is really fast.

Memcached, as its name implies, runs in memory, so the only thing that is going to slow it down is it being overwhelmed with network requests or some inefficiency in its algorithms (which are CPU bound). Personally, I have never hit the upper-bound of either of these, as there is always something else in the my architecture which croaks first.

## The Test

I wrote a test to see if asynchronous memcached communication, using EventMachine, or synchronous memcached communication, using the [memcached](https://github.com/evan/memcached/blob/master/README.rdoc) gem, would be faster.

Here is the code…

```ruby

require 'eventmachine' # for async
require 'memcached'    # for sync
require 'benchmark'

DEBUG = false
TEST_SIZE = 100_000

def debug msg
  if DEBUG
    $stderr.puts msg
  end
end

def async
  EM.run do
    cache = EventMachine::Protocols::Memcache.connect 'localhost', 11211
    debug "sending SET requests..."
    (1..TEST_SIZE).each do |n|
      cache.set "key#{n}", "value#{n}" do
        debug "  SET key#{n} complete"
      end
    end
    debug "SET requests sent"
    debug "sending GET requests..."
    (1..TEST_SIZE).each do |n|
      cache.get "key#{n}" do |value|
        debug "  GET key#{n} = #{value} complete"
      end
    end
    debug "GET requests sent"
    debug "sending DEL requests..."
    (1..TEST_SIZE).each do |n|
      cache.del("key#{n}") do
        debug "  DEL key#{n} complete"
        if n == TEST_SIZE
          EM.stop
        end
      end
    end
    debug "DEL requests sent"
  end
end

def sync
  cache = Memcached.new("localhost:11211")
  debug "sending SET requests..."
  (1..TEST_SIZE).each do |n|
    cache.set "key#{n}", "value#{n}"
    debug "  SET key#{n} complete"
  end
  debug "SET requests sent"
  debug "sending GET requests..."
  (1..TEST_SIZE).each do |n|
    value = cache.get "key#{n}"
    debug "  GET key#{n} = #{value} complete"
  end
  debug "GET requests sent"
  debug "sending DEL requests..."
  (1..TEST_SIZE).each do |n|
    cache.delete("key#{n}")
    debug "  DEL key#{n} complete"
  end
  debug "DEL requests sent"
end

puts Benchmark.measure { puts "sync:";  sync  }
puts Benchmark.measure { puts "async:"; async }

```

## The Results

We are using [benchmark](https://ruby-doc.org/stdlib-1.9.3/libdoc/benchmark/rdoc/Benchmark.html) to measure the time taken.

```
$ ruby memcached.rb
sync:
  4.170000   4.360000   8.530000 ( 17.299242)
async:
 32.150000   0.990000  33.140000 ( 33.246160)

```

>  
> This report shows the user CPU time, system CPU time, the sum of the user and system CPU times, and the elapsed real time. The unit of time is seconds. (from [Benchmark docs](https://ruby-doc.org/stdlib-1.9.3/libdoc/benchmark/rdoc/Benchmark.html))
> 

So we can see from the above that EventMachine-based version took about twice as long to run, took 8 times as much user CPU time and over 4 times as much system CPU time. That is quite significant.

## Avoid EventMachine With Memcached?

This test needs to be put in context. The test was being run on same machine as Memcached, so the network latency was extremely low. EventMachine was not being used for anything else and this script had no other tasks to perform in-between sending requests to Memcached and receiving the responses, so blocking was not an issue.

I could benchmark this and conclude that synchronous Memcached usage was the way to go. I would then roll it out to production, where Memcached is running on a different machine in a different data-center (please do not do this), and the latency would kill this synchronous script. Where you have latency and many requests, asynchronous event-based programming is usually going to win.

Therefore, if the context is such that this kind of synchronous model works better for you, performance is important and you can be sure that things are not going to change, then _maybe_ it is worth consideration.

## Suck It Up

Nearly everything application I write now is event-based. I use EventMachine in Ruby and Tornado’s [io\_loop](https://www.tornadoweb.org/documentation/ioloop.html) in Python. I write high performance code and do everything non-blocking, because I do not want anything to halt my event-loop, ever.

I will gladly take a little overhead and fire up a new process or a new machine, if necessary, if it means that an external service like Memcached will not bring my event-loop to a halt when it has issues. It may be fast now, but one network glitch or Memcached crash may render my event-loop defunct. I might go from processing 10k requests per second to processing 1 per second, if I have a timeout of 1 second on one blocking network connection. So, yes, I will gladly suck up this asynchronous overhead in the short-term to protect from \[expected\] unexpected issues in the future.

## Can EventMachine Be Faster?

I am a believer that anything can be better, faster, stronger. EventMachine is already heavily written in C++, which itself is a clear sign that its operation is CPU-bound.

There is a pure ruby implementation of EventMachine. You can play with this to compare the performance of the C++ implementation. In basic tests, you unlikely see a difference. The general benefit you get with an event-based system, when dealing with latency in disk and network I/O, far out-weighs the overhead of the event-system. It is only when you start to hit it extremely hard you will see the differences.

A faster EventMachine would be great, but it will make little difference to you when comparing with asynchronous code. You can never escape the overhead that asynchronous code adds. Therefore, synchronous code will continue to much look much faster in examples like the one above.

Event-based programming enables your application to utilize 100% of the cpu, because anything not cpu-bound can be passed off to the operating system. Therefore, if our code-base is truly event-based, we would only see the benefit of a faster EventMachine once we hit 100% utilization of the CPU.

__Side note:__ I have hit some bugs when using the pure-ruby implementation in my code and the EventMachine test-suite was not passing for me when trying to use pure-ruby. The test-suite is now passing with the latest HEAD of the [git repository](https://github.com/eventmachine/eventmachine), so these might have been a temporary issues, but it highlighted to me the order of priority for C++ vs pure-ruby implementations.

## Resources

*   [EventMachine Mailing-list](https://groups.google.com/forum/?fromgroups=#!forum/eventmachine)

## Ruby’s EventMachine

Other posts in this series

*   [Part 1 : Event-based Programming](/rubys-eventmachine-part-1-event-based-programming)
*   [Part 2 : Asynchronous != Faster](/ruby-eventmachine-part-2-asynchronous-not-equal-faster)

<div id="comments">
  <h3 id="comments-number" class="comments-header">9 responses to “Ruby’s EventMachine – Part 2 : Asynchronous != Faster”</h3>
  <ol class="comment-list">
    <li id="comment-22146" class="comment even thread-even depth-1 comment reader">
      <img alt="yuan" src="https://0.gravatar.com/avatar/47081f2906ff164aadfa8eea575c4189?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">yuan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, September 24th, 2012, 7:56 pm">September 24, 2012</abbr> at <abbr class="comment-time" title="Monday, September 24th, 2012, 7:56 pm">7:56 pm</abbr>
      </div>
      <div class="comment-text">
        <p>great!  how can I integrate eventmachine to my rails apps?</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-22151" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Ludovic Henry" src="https://0.gravatar.com/avatar/cfaf18c547ad83146d89bba7cdca1339?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://ludovic-henry.com">Ludovic Henry</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, September 24th, 2012, 10:48 pm">September 24, 2012</abbr> at <abbr class="comment-time" title="Monday, September 24th, 2012, 10:48 pm">10:48 pm</abbr>
      </div>
      <div class="comment-text">
        <p>What are the results if you use epoll instead of select? Thank you.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-22286" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, September 27th, 2012, 10:53 am">September 27, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 27th, 2012, 10:53 am">10:53 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Ludovic,</p>
            <p>I added EM.epoll before EM.run and here are the results</p>
            <p>$ ruby memcached.rb<br />
sync:<br />
  3.950000   4.240000   8.190000 ( 16.496183)<br />
async:<br />
 40.090000   1.110000  41.200000 ( 41.708882)</p>
            <p>Actually a little worse performance, but this is running on my Mac and epoll is for multiplexed I/O that is available in Linux 2.6 kernels.</p>
            <p>Cheers,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-22319" class="comment odd alt depth-3 comment reader">
              <img alt="Ludovic Henry" src="https://0.gravatar.com/avatar/cfaf18c547ad83146d89bba7cdca1339?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn" title="https://ludovic-henry.com">Ludovic Henry</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Friday, September 28th, 2012, 1:19 am">September 28, 2012</abbr> at <abbr class="comment-time" title="Friday, September 28th, 2012, 1:19 am">1:19 am</abbr>
              </div>
              <div class="comment-text">
                <p>If you are using OS/X – BSD, you should be using kqueue with “EM.kqueue = true if EM.kqueue?”.</p>
                <p>About Epoll, if it’s not available on the platform (not on linux2.6+), it should fallback to select..</p>
              </div>
              <!-- .comment-text -->
            </li>
            <!-- .comment -->
          </ol>
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-22451" class="comment even thread-even depth-1 comment reader">
      <img alt="Phil Pirozhkov" src="https://1.gravatar.com/avatar/bb3cf80b7af6b40db1b3b2445ee738a1?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Phil Pirozhkov</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, September 30th, 2012, 3:45 pm">September 30, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 30th, 2012, 3:45 pm">3:45 pm</abbr>
      </div>
      <div class="comment-text">
        <p>It’s a very good example that EM is still far from being perfect.<br />
This is mostly due to the poor memcached protocol implementation, which adds all these callbacks to an array and is poping it one by one.</p>
        <p>There’s a problem in test itself, you should consider that in evented style you never know which operation finishes first, and you cannot be sure that DEL is sent after SET and GET. So this should be better:</p>
        <div class="CodeRay">
          <div class="code">
            <pre>require 'em-synchrony'
require 'em-synchrony/em-memcache'
def async2
  EventMachine.synchrony do
    cache = EM::P::Memcache.connect
    TEST_SIZE.times do |n|
      cache.set "key#{n}", "value#{n}"
    end
    TEST_SIZE.times do |n|
      value = cache.get "key#{n}"
    end
    TEST_SIZE.times do |n|
      cache.delete("key#{n}")
    end
    EventMachine.stop
  end
end
puts Benchmark.measure { puts "async:"; async2 }
</pre>
          </div>
        </div>
        <p>However, the overhead is the same.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-22455" class="comment odd alt depth-2 comment reader">
          <img alt="Phil Pirozhkov" src="https://1.gravatar.com/avatar/bb3cf80b7af6b40db1b3b2445ee738a1?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">Phil Pirozhkov</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, September 30th, 2012, 4:29 pm">September 30, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 30th, 2012, 4:29 pm">4:29 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Forgot to mention that on my machine sync is only 50%-100% faster than async given your benchmark. (2 cores, linux 3.5.4).</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-22538" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Jarmo Pertman" src="https://1.gravatar.com/avatar/ff97ca87af59ee68ceff5877a8365788?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://itreallymatters.net">Jarmo Pertman</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, October 4th, 2012, 10:54 am">October 4, 2012</abbr> at <abbr class="comment-time" title="Thursday, October 4th, 2012, 10:54 am">10:54 am</abbr>
      </div>
      <div class="comment-text">
        <p>You could also try Benchmark.bmbm instead of .measure to see if there’s any effect when having also the rehearsal phase.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-24355" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Nicolás" src="https://1.gravatar.com/avatar/d26aa7a45965be73497f83347f1800f8?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Nicolás</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, October 30th, 2012, 6:18 am">October 30, 2012</abbr> at <abbr class="comment-time" title="Tuesday, October 30th, 2012, 6:18 am">6:18 am</abbr>
      </div>
      <div class="comment-text">
        <p>Greate post! Waiting for part 3!!!</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-24376" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Tuesday, October 30th, 2012, 2:43 pm">October 30, 2012</abbr> at <abbr class="comment-time" title="Tuesday, October 30th, 2012, 2:43 pm">2:43 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Be careful what you wish for…<br />
https://www.bigfastblog.com/rubys-eventmachine-part-3-thin</p>
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

