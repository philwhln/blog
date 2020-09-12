---
title: "54 Hours In The Okanagan Building A Startup"
date: "2012-03-05T18:04:00-08:00"
template: "post"
draft: false
slug: "54-hours-in-the-okanagan-building-a-startup"
category: "Startups"
tags:
  - 54hours
  - accelerate okanagan
  - amazon s3
  - bc
  - canvas
  - html5
  - html5 canvas
  - kelowna
  - kelowna innovation centre
  - nginx
  - oauth
  - okanagan
  - python
  - startup weekend
  - startups
  - swokanagan
  - tornado
  - tweepy
  - twitter
  - vancouver
description: "Remember, Startup Weekend equates to about 21 hours of development time and that includes brainstorming, planning, talking to mentors, other meetings, eating and a bit of fun. Also, skill-sets between developers do not always line up. The key phrase is “ship it!” whether it is good, bad, ugly or broken. The clock is ticking."
socialImage:
  publicURL: "/media/images/2012/03/office_small.jpg"
---
<img align="left" src="/media/images/swokanagan/office.jpg?3" style="width: 350px;"/>

The first, and I’m certain not the last, [Startup Weekend Okanagan](https://okanagan.startupweekend.org/) was hosted by [Accelerate Okanagan](https://www.accelerateokanagan.com/) at the [Kelowna Innovation Centre](https://www.accelerateokanagan.com/venue/kelowna-innovation-centre-2/).

Driving into the Kelowna felt much like driving from SFO Airport into San Francisco. The Okanagan is dry, arid wine country with a mediterranean feel. If you ignore the patches of snow that are currently on the hills, you could almost imagine you were entering the bay-area 30 years ago. There are arrays of large bill-boards, but unlike modern San Francisco, none of them are for tech, unless you count washing machines in your “tech”. There are no billboards for “DropBox”, “iCloud” “Your Cloud” or “myCloud”. Just good wholesome _profitable_ companies who likely do _not_ have their social networking strategy figured out just yet.

*   [The Scene](/54-hours-in-the-okanagan-building-a-startup#the-scene)
*   [Our Team, The Badgers](/54-hours-in-the-okanagan-building-a-startup#our-team-the-badgers)
*   [Where Can I Find This Great Service?](/54-hours-in-the-okanagan-building-a-startup#where-can-i-find-this-great-service)
*   [Under The Hood](/54-hours-in-the-okanagan-building-a-startup#under-the-hood)
    
    *   [HTML5](/54-hours-in-the-okanagan-building-a-startup#html5)
    *   [Twitter OAuth](/54-hours-in-the-okanagan-building-a-startup#twitter-oauth)
    *   [Amazon S3](/54-hours-in-the-okanagan-building-a-startup#amazon-s3)
    *   [Tornado / Python](/54-hours-in-the-okanagan-building-a-startup#tornado-python)
    *   [Tweepy](/54-hours-in-the-okanagan-building-a-startup#tweepy)
    
    
    
*   [Conclusion](/54-hours-in-the-okanagan-building-a-startup#conclusion)
*   [54 Hours?](/54-hours-in-the-okanagan-building-a-startup#54-hours)
*   [Upcoming Startup Weekends](/54-hours-in-the-okanagan-building-a-startup#upcoming-startup-weekends)

<h2 id="the-scene">The Scene</h2>

I was curious if there was a tech scene in Kelowna and I was not disappointed. Of the 80 participants signed up for the weekend, the majority were not technical, but almost all fit the profile for building a startup. Everyone had brought their passion and enthusiasm to the event. The vibe the first night was quite exciting. Lots of pitches and some good ideas. Some were out of the scope of building in 2 days, some had been done before, some were just bad ideas and some… “yeah, that might not be a bad idea”.

<h2 id="our-team-the-badgers">Our Team, The Badgers</h2>

The team I joined was “Time Badger” (originally “Twitter Badge”).

We had more technical members than some of the other startup teams. We had all of two developers, including myself. In total we were 5, with 2 others adding support as needed.

Our plan was to build a service that could add a badge to your Twitter profile image and revert it back after a set period of time. I will not pitch the whole concept here, but there are endless things you can do with overlaying images and campaigns have the possibility of going viral (“sign up to get Justin Bieber’s eyebrows on your profile. Tuesday only”). Revenue model? Yes, we had one of those, too. Honest.

<h2 id="where-can-i-find-this-great-service">Where Can I Find This Great Service?</h2>

We actually managed to build a decent functioning service at <https://www.timebadger.com>. You can also find us at [@timebadge](https://twitter.com/#!/timebadge) on Twitter. Yes, it is not @timebadger, just @timebadge. Is it now actually harder to get Twitter handles than domain names?

[Check it out](https://www.timebadger.com).

[![](/media/images/swokanagan/support_timebadger.png)](https://www.facebook.com/TimeBadger)

<h2 id="under-the-hood">Under The Hood</h2>

<h3 id="html5">HTML5</h3>

We used HTML5 on the front-end, so we have limited the service to people that have a modern web browser. Remember, Startup Weekend equates to about 21 hours of development time and that includes brainstorming, planning, talking to mentors, other meetings, eating and [a bit of fun](https://twitter.com/#!/SWOkanagan/status/176109663497093120/photo/1). Also, skill-sets between developers do not always line up. The key phrase is “ship it!” whether it is good, bad, ugly or broken. The clock is ticking. 

[Brent Luehr](https://twitter.com/seratonik), my partner in crime for the weekend, did an amazing job of throwing up a UI using [HTML5′s Canvas](https://en.wikipedia.org/wiki/Canvas_element). Using the interface he built, you can drag your chosen badge image to overlay in the correct position and even scale it to the size you want. The browser itself merges the images and POSTs a single JPEG back to server. This was great. It meant that we did not be worry about installing any image manipulation packages on the server.

<h3 id="twitter-oauth">Twitter OAuth</h3>

We used Twitter for the login to the site. This made sense, since our focus was on using the Twitter API to update the user’s profile image and we needed them to authenticate with Twitter.

<h3 id="amazon-s3">Amazon S3</h3>

The reason for using [Amazon S3](https://aws.amazon.com/s3/) was for storing each user’s original Twitter profile image for reverting back to. We also stored our badged versions of the profile image in S3.

Amazon were a sponsor of the Startup Weekend and like most sponsors they were offering some of their services for free and prizes for the best usage of their service. Often few, if anyone, utilizes these low hanging fruit easy wins. This was a great tip from [Daryl Chymko](https://twitter.com/dchymko).

Since we were coding in Python we used the [Boto library](https://github.com/boto/boto) to interact with Amazon S3.

<h3 id="tornado-python">Tornado / Python</h3>

I had used the [Tornado web server/framework](https://www.tornadoweb.org/) for my [previous project](https://www.cooq.com/) and I knew we could easily do [Twitter OAuth](https://www.tornadoweb.org/documentation/auth.html#twitter). Luckily Brent had some Python experience and was onboard with using it. My fear had been that any project I joined would be using Ruby On Rails and my rusty RoR would slow progress. I think Tornado was a good choice, but I maybe biased here.

<h3 id="tweepy">Tweepy</h3>

My new found love for [Tweepy](https://github.com/tweepy/tweepy) comes from the fact that I was bashing my head against a wall trying to get the profile image upload to Twitter working. The rather unhelpful Twitter API was either returning a HTML page saying “something went wrong” or JSON as though something went right. Either way, the profile image was not updating. Tweepy was at the right place at the right time. After trying various approaches and feeling the sands of time slipping away, Tweepy just worked. “Woo-hoo!”, I cried and quickly moved onto the next task.

<h2 id="conclusion">Conclusion</h2>

After not going to Startup Weekend Vancouver a few months ago, I really regretted it when I heard what a great experience it had been for everyone involved. I found out about this Startup Weekend in Kelowna only few days before and jumped on the opportunity to go and I am glad I did. One of the sponsors, [Contractually](https://contractual.ly/), helped me out with a ticket, so I hit the road. 

I encourage anyone interested in building a startup to join this movement when it comes to your city. Most of these events are organized by volunteers who have been to Startup Weekend’s in other cities and wanted to bring it home. For instance, [Daryl Chymko](https://twitter.com/dchymko) went to Startup Weekend Vancouver and wanted to host one in the Okanagan, as did [Evan Willms](https://twitter.com/#!/EvanWillms), who is hosting Startup Weekend Victoria (BC, Canada) at the beginning of June.

<img src="/media/images/swokanagan/startup_hangover.png" style="margin-left: 20px;"/>

<img src="/media/images/swokanagan/pumped_about_career.png?2" style="margin-left: 100px;"/>

<img src="/media/images/swokanagan/blown_away.png" style="margin-left: 50px;"/>

<h2 id="54-hours">54 Hours?</h2>

They say Startup Weekend is “54 hours”, but that is only if you are completely nuts and do not sleep. I’m not nuts. As I write this, that “54 hours in the Okanagan” is growing, since the road back to Vancouver is closed due to accidents and snow. What a great opportunity to write a blog post, I thought.

<h2 id="upcoming-startup-weekends">Upcoming Startup Weekends</h2>

These Startup Weekends are happening in just the next few weeks.

__March 9-11, 2012__  
Bretagne, France  
Sinapore, Singapore

<a href="https://contractual.ly/"><img align="right" src="/media/images/contractually/contractually_ad.png"/></a>

__March 16-18__  
Cordaba, Argentina

__March 23-25__  
Pittsburgh, USA  
Halifax, Canada  
Orlando, USA  
Oslo, Norway  
London, UK  
Copenhagen, Denmark  
Charlottesville, USA  
Rouen, France  
Martinique, Martinique  
Trento, Italy

<div id="comments">
  <h3 id="comments-number" class="comments-header">4 responses to “54 Hours In The Okanagan Building A Startup”</h3>
  <ol class="comment-list">
    <li id="comment-10415" class="comment byuser comment-author-angeliqueduffield even thread-even depth-1 comment role-subscriber user-angeliqueduffield">
      <img alt="Angelique Duffield" src="https://1.gravatar.com/avatar/5338607eb8aa8628eb55140539f155fc?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.facebook.com/angelique.duffield">Angelique Duffield</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, March 5th, 2012, 9:12 pm">March 5, 2012</abbr> at <abbr class="comment-time" title="Monday, March 5th, 2012, 9:12 pm">9:12 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Great post Phil.<br />
Nice to read about it from a participant’s point of view…and a non-local at that!</p>
        <p>It was my first Startup weekend, and I thought it was a really successful, fun, co-operative event. </p>
        <p>I was the one taking pics on Sat &amp; Sun…they will be posted in an album called “Startup Weekend Kelowna 2012″ on my Facebook fan page:<br />
https://facebook.com/brightsparksocialmedia</p>
        <p>cheers<br />
Angelique<br />
Bright Spark Media</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-10416" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, March 5th, 2012, 9:42 pm">March 5, 2012</abbr> at <abbr class="comment-time" title="Monday, March 5th, 2012, 9:42 pm">9:42 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks for the comment Angelique! I look forward to seeing your pictures.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-10648" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Cynthia Gunsinger (@gunsinger)" src="https://0.gravatar.com/avatar/e951d8fd5bb797d3ace4bdbd787c1a60?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.gunsinger.com">Cynthia Gunsinger (@gunsinger)</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, March 10th, 2012, 9:08 am">March 10, 2012</abbr> at <abbr class="comment-time" title="Saturday, March 10th, 2012, 9:08 am">9:08 am</abbr>
      </div>
      <div class="comment-text">
        <p>great summary of the weekend from a participant’s perspective, phil! it was great to have you as a participant. <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
        </p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-10656" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="John" src="https://1.gravatar.com/avatar/11fe86969abe988d71b3cf0911f4cc20?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://contractual.ly/">John</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, March 10th, 2012, 12:56 pm">March 10, 2012</abbr> at <abbr class="comment-time" title="Saturday, March 10th, 2012, 12:56 pm">12:56 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Great summary Phil, and glad you had an amazing time. The Contractually team’s happy to get behind your epic weekend.<br />
Too many points of congruency made me smile while reading this… I grew up in Penticton, so always consider the Okanagan home. Really nice seeing signs of life beyond wine, beaches, and fruit stands!<br />
As one of the organizers of Start-up Weekend Vancouver (Nov. 2011), it was a no brainer to support the Kelowna team…<br />
Your Twitter project resonates too… given that I managed to enjoy a little “success” with Mentionmapp.<br />
Cheers for now</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

