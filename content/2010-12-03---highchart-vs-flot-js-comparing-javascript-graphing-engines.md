---
title: "Highchart Vs Flot.js – Comparing JavaScript Graphing Engines "
date: "2010-12-03T14:55:00-08:00"
template: "post"
draft: false
slug: "highchart-vs-flot-js-comparing-javascript-graphing-engines"
category: "Software Engineering"
tags:
  - charts
  - client-side
  - flot.js
  - graphs
  - highchart
  - javascript
  - png
  - render
  - reporting
  - server-side
  - svg
  - web-development
description: "In previous projects at MailChannels I have used Flot.js for graphing. There were many reasons I chose this originally. The graphs are interactive and can be"
socialImage:
  publicURL: "/media/images/photo.jpg"
---
In previous projects at [MailChannels](https://www.mailchannels.com/blog) I have used [Flot.js](https://code.google.com/p/flot/) for graphing. There were many reasons I chose this originally. The graphs are interactive and can be manipulated within the browser without having to communicate back to the server. You can retrieve additional data from the server which enables you to be able to scan left and right on graph. You can animate transitions, zoom in and out, get the values at precise points on the graph and select an area data. When I found Flot.js I was mainly looking for something that was not ugly. I was using Perl on the server side, and Perl has a history of having ugly graphing solutions. I’ve since found some alternatives on the Perl side, but for me Flot.js was the new hotness at that time. You can read my presentation on [Graphs In Perl and Flot.js](https://www.slideshare.net/philwhln/graphs-in-perl-and-flotjs) over at slideshare.

So Flot.js is pretty cool, but what are it’s limitations? The number one limitation is the ability to render a graph anywhere other than within the browser. Since JavaScript is creating the graph it’s hard to automate emailing a static PNG/JPG/GIF image of the graph from the server without firing up a browser and then taking a snapshot.

Enter Highchart.

<div class="wp-caption alignnone" id="attachment_364" style="width: 556px">
<a href="/media/images/2010/12/Screen-shot-2010-12-03-at-2.52.25-PM.png">
<img alt="Example of Highchart graphs" class="size-full wp-image-364" height="211" src="/media/images/2010/12/Screen-shot-2010-12-03-at-2.52.25-PM.png" title="Example of Highchart graphs" width="546"/>
</a>
<p class="wp-caption-text">Example of Highchart graphs</p>
</div>

I discovered Highchart recently while working with [netSIGN](https://www.netsign.com). We were looking at graphing options in Ruby On Rails and came across Highchart. It’s much like Flot.js, but it is a commercial venture. Flot.js is free; released under the MIT license and is sponsored by [IOLA](https://www.iola.dk/) in Denmark. Developers on a shoe-string are unlikely to look at Highchart, just because of price tag, but for most projects I think the price it reasonable. What do you get for your money?

## Documentation

Highchart feels more polished than Flot.js and many of the standard features, simply work out of the box. Flots basic chart is very basic and you have to pass all the options to get something that starts to look nice. Maybe you prefer that though. Finding those options is so much nicer with Highcharts’ [Options Reference](https://www.highcharts.com/ref/). It’s easy to drill down and see exactly how the options piece together through the hierarchical menu. I do not want to knock Flot.js, since it does have documentation, but it is written more like a novel than the reference document and referring to it is time consuming.

## Rendering Offline
  
“Wow!”, was all I could think when I saw that Highchart can actually render the graphs on the server as PNG files. Ok, it’s not a perfect solution and still requires a web-browser, but as long the user is viewing the graph, a copy of the graph can be sent to the server for generation as a PNG or other file format.

How does it do it? The JavaScript in the browser generates SVG, which the browser can render visually. Highchart simply passes that generated SVG back, from the browser, to the server. The server then takes the SVG file and passes it to [SVG Rasterizer](https://xmlgraphics.apache.org/batik/tools/rasterizer.html), which is an Apache Project. It’s a Java program that can be called to follows

```
java -jar batik-rasterizer.jar graph.svg
```

## Limitations

The main limitation with both Flot.js and Highchart is that of generating offline reports. Even though Highchart gets further down the road, it cannot be done without first rendering the graph in the browser. I have not investigated, but I have a feeling that you could do this if you have a [server that can run JavaScript code](https://developer.mozilla.org/en/JavaScript_shells), such as [node.js](https://nodejs.org/).

## Conclusion

Yes, there is a cost, but I think it’s worth going with Highchart. Flot.js has not evolved much in the time I’ve known it and I think Highchart is a more mature product and is quicker to get up and running with. If you are a commercial venture (Highchart is free for non-commercial) and you do not want to spend a penny then Flot.js is still a great solution and I would not dissuade anyone from using it.

## Comments

<div id="comments">
  <ol class="comment-list">
    <li id="comment-490" class="comment even thread-even depth-1 comment reader">
      <img alt="Montreal Web Design" src="https://0.gravatar.com/avatar/2e41d52b2986c0c63a2f464e34b7b3f3?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://websitecenter.ca/">Montreal Web Design</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, December 26th, 2010, 7:46 pm">December 26, 2010</abbr> at <abbr class="comment-time" title="Sunday, December 26th, 2010, 7:46 pm">7:46 pm</abbr>
      </div>
      <div class="comment-text">
        <p>If you are using Java for HTTP server you might be able to integrate it with rhino https://www.mozilla.org/rhino/ and SVG Rasterizer</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-2951" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Khan" src="https://0.gravatar.com/avatar/4fd93a3eccffb8c13e3cf9becc18d3a3?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Khan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, May 18th, 2011, 10:08 pm">May 18, 2011</abbr> at <abbr class="comment-time" title="Wednesday, May 18th, 2011, 10:08 pm">10:08 pm</abbr>
      </div>
      <div class="comment-text">
        <p>It’ll be great if you can show an example of highcharts with perl.</p>
        <p>Thanks</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-3484" class="comment even thread-even depth-1 comment reader">
      <img alt="tau-lepton" src="https://0.gravatar.com/avatar/4926dd13fe5428873d77d8222d65f303?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">tau-lepton</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, June 26th, 2011, 4:57 pm">June 26, 2011</abbr> at <abbr class="comment-time" title="Sunday, June 26th, 2011, 4:57 pm">4:57 pm</abbr>
      </div>
      <div class="comment-text">
        <p>have you tried wkhtmltopdf? and wkhtmltoimage?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-9492" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, February 20th, 2012, 10:22 pm">February 20, 2012</abbr> at <abbr class="comment-time" title="Monday, February 20th, 2012, 10:22 pm">10:22 pm</abbr>
          </div>
          <div class="comment-text">
            <p>I have tried both of these and they work very well. An example of wkhtmltopdf with flot.js, that I worked on, can be seen to you download PDF reports from gtmetrix.com</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-25226" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Chris" src="https://0.gravatar.com/avatar/048c615183b42a20ec81b90af7ab221d?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Chris</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, November 8th, 2012, 12:13 pm">November 8, 2012</abbr> at <abbr class="comment-time" title="Thursday, November 8th, 2012, 12:13 pm">12:13 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Highchart is free o.O<br />
https://www.highcharts.com</p>
        <p>-Chris</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-25228" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, November 8th, 2012, 12:24 pm">November 8, 2012</abbr> at <abbr class="comment-time" title="Thursday, November 8th, 2012, 12:24 pm">12:24 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Chris. Only for a personal or non-profit project. A single website license is $80.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
        <li id="comment-27253" class="comment even depth-2 comment reader">
          <img alt="eLobrino" src="https://0.gravatar.com/avatar/8c7d91463125e762373cb26d0bfe7bd9?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">eLobrino</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, November 26th, 2012, 9:09 pm">November 26, 2012</abbr> at <abbr class="comment-time" title="Monday, November 26th, 2012, 9:09 pm">9:09 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Yeah it’s free to download and to use.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-43433" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Jaimin" src="https://1.gravatar.com/avatar/f29e862e012e69cf44a2380d17961313?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.tuvalabs.com">Jaimin</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, April 24th, 2013, 9:57 am">April 24, 2013</abbr> at <abbr class="comment-time" title="Wednesday, April 24th, 2013, 9:57 am">9:57 am</abbr>
      </div>
      <div class="comment-text">
        <p>Have you tried jqplot – https://www.jqplot.com/?</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-65206" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Alan Chavez" src="https://0.gravatar.com/avatar/4ba0e5820cec0a91f873370a50039b87?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.alanchavez.com">Alan Chavez</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, September 10th, 2013, 8:12 am">September 10, 2013</abbr> at <abbr class="comment-time" title="Tuesday, September 10th, 2013, 8:12 am">8:12 am</abbr>
      </div>
      <div class="comment-text">
        <p>For automated reports what I do is running PhantomJS (or casperJS) which are headless browsers and then just create the charts. </p>
        <p>Casper is based on PhantomJS, and PhantomJS works on node, that’s something you could use for offline reporting.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

