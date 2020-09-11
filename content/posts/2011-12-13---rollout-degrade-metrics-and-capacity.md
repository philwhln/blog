---
title: "Rollout, Degrade, Metrics and Capacity"
date: "2011-12-13T16:16:00-08:00"
template: "post"
draft: false
slug: "rollout-degrade-metrics-and-capacity"
category: "DevOps"
tags:
  - a-b-testing
  - averages
  - capacity planning
  - degrade
  - fetlife
  - ganglia
  - metrics
  - monitoring
  - percentiles
  - redis
  - rollout
  - ruby
  - scala
description: "Last week I attended a presentation by James Golick, CTO of fetlife.com. Intriguingly advertised as Scaling a Website to 300 Million Page Views Per Month with"
socialImage:
  publicURL: "/media/images/2011/12/rollout-degrade-metrics-and-capacity_sq.jpg"
---
<div style="height: 280px"><a href="https://www.flickr.com/photos/yvoictra/3498203833/"><img align="left" height="251" src="https://commondatastorage.googleapis.com/philwhln/blog/media/images/rollout_degrade_metrics_and_capacity/rollout-degrade-metrics-and-capacity.jpg" width="320"/></a> Last week I attended a presentation by <a href="https://jamesgolick.com/">James Golick</a>, CTO of <a href="https://www.fetlife.com">fetlife.com</a>. Intriguingly advertised as <a href="https://www.meetup.com/vancouver-scala/events/38342382/">“Scaling a Website to 300 Million Page Views Per Month with Scala”</a> I thought it was worth a break from building <a href="https://www.cooq.com/">cooq.com</a> to attend. Fetlife have been heavy users of Ruby, but are transitioning to Scala and having been using Scala code in production for 2 years. Several of the tools mentioned below are built in Ruby, but they plan to migrate them to Scala over time.</div>

## Rollout

[Rollout](https://github.com/jamesgolick/rollout) is a Ruby package used to deploy new features in a controlled manner. It uses a central configuration hosted within [Redis](https://redis.io/) to manage instantaneous deployment on a per-user basis. For instance, Fetlife uses it specify that 10% of their user-base should be able to see feature X when they view the website. This is done very basically by modulo of user id. At 10% any user that has a user id ending in “1″ will see the new feature. The same subset of users will always see any new features first.

Updates to Rollout’s configuration will occur immediately due to the inline code fetlife uses within it’s web application.

```

if $rollout.active?(:chat, User.first)
    # implement chat feature
end

```

The above code checks with Rollout whether the “chat” feature is active for this user. If it is then we go on to run through the code that implements this feature. This _$rollout.active_ statement results in a call to Redis which responses with _true_ or _false_.

For me this seems like a pretty elegant solution. The downside is that a site with a lot of users and a lot of features is going to being hitting Redis pretty hard (requests to Redis = user requests x features). Luckily, Redis runs in memory, so it is very fast. It can also be scaled horizontally across a cluster to increase the throughput of requests.

## Degrade

[Degrade](https://github.com/jamesgolick/degrade) is another open source Ruby package written by Fetlife. It focuses on isolating failures by monitoring the error rates from a given feature and applying Rollout rules as failure thresholds are reached.

It is not necessary to use Rollout to take action when a threshold is reached. The trigger is simply a programmer-defined Ruby function that should be called. Obviously, Degrade and Rollout go hand-in-hand, so if you are using Rollout, then it makes sense to use Rollout to degrade the use of the failing feature.

Like Rollout, Degrade uses [Redis](https://redis.io/) to manage the failure statistics.

## Metrics

### Ganglia

In his talk, James spoke highly of [Ganglia](https://ganglia.sourceforge.net/) for system monitoring metrics. It’s very scalable and can be used to monitoring large clusters, such as clusters running sizeable [Hadoop](https://www.cloudera.com/blog/2009/03/hadoop-metrics/) deployments.

Any metrics can be pushed to Ganglia, so it can be [integrated with your code](https://www.igvita.com/2010/01/28/cluster-monitoring-with-ganglia-ruby/) easily. Functions can be wrapped in timers and captured durations can be pushed to the Ganglia daemon. Ganglia creates a new RRD file for each new metric that is pushed to it.

### Don’t Be Average

>  
> _“By the time an anomaly is reflected in the average (mean), it’s already too late”_  
> <small> – James Golick, BitLove</small>
> 

James highlighted the importance of looking a percentiles over averages. Looking at edges cases is much more useful. If you see 1% of requests taking 10 times longer than other requests, then there’s an obvious problem that can be looked into and fixed before it becomes a real issue. If you wait until you see the average rise, it may be too late. We all know the knock on effect of what happens when things start backing up. More memory is used. Less memory is available to resolve the issue and the situation can spiral out of control quickly.

### Take A Broad View

Looking at single servers statistics in Ganglia, of an alternative metrics system, can hide anomalies. James pointed out that if 1 server out of 30 was taking 20 milliseconds longer to do something, by itself would not look unhealthy. But in a line-up with the other 29 servers, it would stand out and would warrant further investigation. This may be another early sign of something bad on the horizon.

There are too many statistics for each server to possibly notice anomalies if you view them one server at a time.

## Capacity Planning

Capacity Planning is something that James’ team at Fetlife focus on on a regular basis. They are not highly scientific about it, but they understand that it is highly important.

James suggested looking at the book [The Art Of Capacity Planning](https://shop.oreilly.com/product/9780596518585.do), but the suggested that it might put you to sleep due to it heavy focus on _best-fit-curves_. Rather than curves, Fetlife use a simple straight line to project future hardware requirements and continuously readjust this line over time. He suggested that unless you are Facebook high-science in this area is not important. For Facebook, being off the curve can cost millions of dollars in redundant hardware or potential downtime, so the risks are higher. At the smaller scale this cost of complexity outweighs the cost of having 10% too much hardware.

### Understand Your Spikes

Capacity planning is not just about projecting your current usage. Websites have spikes. During certain sports events Twitter and Facebook see large spikes in traffic. They understand their spikes, know how big their potential are and plan accordingly. James points out that for most sites spikes are very small and even with the worst spiky offenders, after a couple of spikes you have a pretty good idea of what to expect.

<div id="comments">
  <h3 id="comments-number" class="comments-header">One response to “Rollout, Degrade, Metrics and Capacity”</h3>
  <ol class="comment-list">
    <li id="comment-6614" class="pingback even thread-even depth-1 pingback reader">
      <img alt="Link: Rollout Degrade Metrics and Capacity | Loosely Connected" src="https://0.gravatar.com/avatar/?d=https://www.bigfastblog.com/css/library/media/images/pingback.png&amp;s=80" class="avatar avatar-80 photo avatar-default" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://looselyconnected.wordpress.com/2011/12/16/link-rollout-degrade-metrics-and-capacity/">Link: Rollout Degrade Metrics and Capacity | Loosely Connected</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, December 16th, 2011, 8:13 am">December 16, 2011</abbr> at <abbr class="comment-time" title="Friday, December 16th, 2011, 8:13 am">8:13 am</abbr>
      </div>
      <div class="comment-text">
        <blockquote>
          <p>This is a pretty fascinating article. While very high level, it talks about things I’ve been interested in (on a much smaller scale) for a while.</p>
          <p>I’ve hand-coded some solutions that do these sorts of things, but not to this scale or completeness. The article surprised me with the nature of some of the technologies they were using; for example, I’d always thought of Ganglia as a monitoring solution, not for metrics gathering.</p>
        </blockquote>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

