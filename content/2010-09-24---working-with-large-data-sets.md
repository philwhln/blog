---
title: "Working With Large Data Sets"
date: "2010-09-24T02:35:00-07:00"
template: "post"
draft: false
slug: "working-with-large-data-sets"
category: "Software Engineering"
tags: []
description: "For the past three and a half years I have been working for a start-up in downtown Vancouver. We have been developing a high performance SMTP proxy that can"
socialImage:
  publicURL: "/media/images/photo.jpg"
---


<div id="comments">
  <h3 id="comments-number" class="comments-header">13 responses to “Working With Large Data Sets”</h3>
  <ol class="comment-list">
    <li id="comment-41" class="comment even thread-even depth-1 comment reader">
      <img alt="SL" src="https://1.gravatar.com/avatar/74e69ac9f70069738b907b47bd2c6c50?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">SL</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, September 24th, 2010, 5:04 am">September 24, 2010</abbr> at <abbr class="comment-time" title="Friday, September 24th, 2010, 5:04 am">5:04 am</abbr>
      </div>
      <div class="comment-text">
        <p>Very cool stuff. I’ve always been hesitant to blog about anti-spam for fear of spammers picking up on it and modifying their techniques. Are you concerned that spammers might do that in this case?</p>
        <p>I’m just starting a tech company and hope to be dealing with large data sets. It’s a balance between planning for the future and acting now. If there’s no action now, there’s not future but without planning the future will be disastrous.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-42" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <div class="rpx_google_small rpx_icon_small rpxauthor" title="admin"></div>
              <cite class="fn">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Friday, September 24th, 2010, 5:18 am">September 24, 2010</abbr> at <abbr class="comment-time" title="Friday, September 24th, 2010, 5:18 am">5:18 am</abbr>
          </div>
          <div class="comment-text">
            <p>Well, if you have a technology that you intend to sell, then you need to talk about it, and the bad guys usually figure it out quickly enough anyway. Actually, I’ve recently moved on from MailChannels to start my own tech company, so I’m not trying to sell the technology right now. Without getting into too much detail, slowing connections targets the economics of sending spam. Spammers have to get an enormous amount of spam out to get enough people to click on the links or buy of the controversial pharmaceuticals that they are selling. Their click-through-rate is incredibly small. Even if they know you are slowing them down, they do not have the time to wait around. It’s better for them to move onto the next guy.</p>
            <p>My approach to starting a tech company is getting things up and running as quickly as possible. Use the languages and tools that you are most comfortable with. You do not need to scale until you are successful, at which point it’s easier to rewrite things in a more scalable way, or hire somebody else who knows how to do it, and concentrate on putting out the other fires in your successful business. You’ll probably be rewriting things anyway as the product or service evolves and you learn more about the need that you are addressing. When you make things scalable you lock yourself into certain choices that limits your ability to be dynamic.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-43" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Alan" src="https://0.gravatar.com/avatar/ebd3cf4b0b5fd724f593e5d47c866d6e?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://alan.blog-city.com/">Alan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, September 24th, 2010, 4:08 pm">September 24, 2010</abbr> at <abbr class="comment-time" title="Friday, September 24th, 2010, 4:08 pm">4:08 pm</abbr>
      </div>
      <div class="comment-text">
        <p>@Phil excellent post. I appreciate the problems you face when dealing with extremely high numbers in a short space of time.  Your distributed search with Lucene sounds interesting.  Would love to hear more about that in future.</p>
        <p>One method we use with extreme effectiveness is the use of the data structure, BloomFilter.  This data structure is an extremely effective tool for determining if you have seen something before without any expensive database or search lookup.   It is the technology a lot of the big hash map implementations use internally.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-45" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <div class="rpx_google_small rpx_icon_small rpxauthor" title="admin"></div>
              <cite class="fn">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Friday, September 24th, 2010, 11:47 pm">September 24, 2010</abbr> at <abbr class="comment-time" title="Friday, September 24th, 2010, 11:47 pm">11:47 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Alan,</p>
            <p>Thank you for the comment. The Bloom Filter does sound like a great tool, that I’ve not come across before. I will have to read up on it.</p>
            <p>In this case, where the goal is to build concrete statistics, I think the false-positive aspect of the Bloom Filter would have affected the results. Even if it was ever so slightly, I would have felt uncomfortable publishing them and referring to somebody else’s technology (Spamhaus in this case) where I knew there may be a slight inaccuracy.</p>
            <p>I see that the Bloom Filter addresses memory consumption. Since I had so much data, memory was not an issue. That’s sounds backwards and it is. Since the data cannot possibly fit into a machine memory, I used several technics I discovered from working with Hadoop. Basically streaming data to and from disk in the most efficient way, performing several transformations of the data and writing the data to disk to divide, organise, sort, merge and conquer. I think the peak memory I used was far less than I had on the machine, and this was generally highest when sorting the individual files.</p>
            <p>I’m considering writing a blog post on the details of how I tackled this data, as this blog post proved to be quite popular. Would that be something you would be interested in?</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-52" class="comment even thread-even depth-1 comment reader">
      <img alt="Antti Siira" src="https://0.gravatar.com/avatar/cd4b8031f3ade684376d8b7985728dbd?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Antti Siira</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, September 25th, 2010, 11:16 pm">September 25, 2010</abbr> at <abbr class="comment-time" title="Saturday, September 25th, 2010, 11:16 pm">11:16 pm</abbr></div>
      <div class="comment-text">
        <p>If you want to see bloom filters used in spam processing, you could check out gross[1] that uses them for greylisting. It has proven[2] itself quite efficient way to block large amount of spam.</p>
        <p>[1] https://code.google.com/p/gross/<br />
[2] https://lists.utu.fi/pipermail/gross/2007/000035.html</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-54" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <div class="rpx_google_small rpx_icon_small rpxauthor" title="admin"></div>
              <cite class="fn">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, September 27th, 2010, 2:35 am">September 27, 2010</abbr> at <abbr class="comment-time" title="Monday, September 27th, 2010, 2:35 am">2:35 am</abbr></div>
          <div class="comment-text">
            <p>Hi Antti,</p>
            <p>My experience, from working with our customers at MailChannels, was that grey-listing started to become ineffective a couple of years ago as the spambot writers started to incorporate this technology into their software and perform retries, just as a legitimate email server would. Not actually having worked with grey-listing myself, I do have any hard figures to back-up conversations with customers, who sometimes became our customers whilst looking for alternatives to grey-listing.</p>
            <p>You are obviously an expert in this area, so I would be really interested in any statistics or other data you have on the effectiveness of grey-listing. Do you have a blog?</p>
            <p>Thanks,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-56" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Michael Fever" src="https://0.gravatar.com/avatar/21bf480a84fcc2051cdd6c8bb079248c?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://vaporizer-store.net">Michael Fever</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, September 27th, 2010, 2:36 pm">September 27, 2010</abbr> at <abbr class="comment-time" title="Monday, September 27th, 2010, 2:36 pm">2:36 pm</abbr></div>
      <div class="comment-text">
        <p>Thats awesome, congratulations. It’s really refreshing to see these kinds of innovations coming out of Canada, even it if it Vancouver and not Toronto lol =)</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-57" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <div class="rpx_google_small rpx_icon_small rpxauthor" title="admin"></div>
              <cite class="fn">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, September 27th, 2010, 11:09 pm">September 27, 2010</abbr> at <abbr class="comment-time" title="Monday, September 27th, 2010, 11:09 pm">11:09 pm</abbr></div>
          <div class="comment-text">
            <p>Thanks Michael. Hopefully Toronto will catch up with the technological juggernaut that Vancouver is.</p>
            <p>On a serious note, MailChannels is developing some very cool stuff in the anti-spam space. They’re expanding the development team right now, so if there is anyone in Vancouver interested, you should pop in and say hi.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-489" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Montreal Web Design" src="https://0.gravatar.com/avatar/2e41d52b2986c0c63a2f464e34b7b3f3?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://websitecenter.ca/">Montreal Web Design</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, December 26th, 2010, 7:37 pm">December 26, 2010</abbr> at <abbr class="comment-time" title="Sunday, December 26th, 2010, 7:37 pm">7:37 pm</abbr></div>
      <div class="comment-text">
        <p>You might want to look at s4 by Yahoo. They recently open-sourced it.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-492" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <div class="rpx_google_small rpx_icon_small rpxauthor" onclick="window.location.href='https://www.google.com/profiles/101358683928607234715'" title="admin"></div>
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, December 26th, 2010, 8:10 pm">December 26, 2010</abbr> at <abbr class="comment-time" title="Sunday, December 26th, 2010, 8:10 pm">8:10 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Montreal Web Design. Here’s a link for anyone interested https://s4.io/</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
  <div class="comment-navigation paged-navigation">
									</div>
  <!-- .comment-navigation -->
</div>

