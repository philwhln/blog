---
title: "Colorful Ruby Code In HTML Using CodeRay"
date: "2012-09-03T10:27:00-07:00"
template: "post"
draft: false
slug: "syntax-highlight-your-ruby-code-in-html"
category: "Ruby"
tags:
  - blogging
  - coderay
  - colorize
  - css
  - gem
  - html
  - pygments
  - render
  - ruby
  - syntax highlighting
description: "In this blog post I will demonstrate a quick and easy way I have found to syntax highlight your Ruby code in pretty colors for your blog posts or other HTML-based display purposes. Please note, this blog post is designed for use with color monitors only, obviously. The focus of this post is Ruby, but this method also applies for syntax highlighting C, C++, Python, CSS, Clojure, diff, Groovy, HAML, ERB, Java, JavaScript, PHP, SQL, XML and YAML."
socialImage:
  publicURL: "/media/images/photo.jpg"
---
<a href="https://www.flickr.com/photos/vinothchandar/4297342496/in/photostream/"><img align="left" src="https://farm5.staticflickr.com/4011/4297342496_985f56d8e2_n.jpg"/></a>In this blog post I will demonstrate a quick and easy way I have found to syntax highlight your Ruby code in pretty colors for your blog posts or other HTML-based display purposes. Please note, this blog post is designed for use with color monitors only, obviously.

The focus of this post is Ruby, but this method also applies for syntax highlighting C, C++, Python, CSS, Clojure, diff, Groovy, HAML, ERB, Java, JavaScript, PHP, SQL, XML and YAML.

## What’s Wrong With Good Ol’ Black and White?

Here is some [arbitrary code](https://github.com/BigFastSite/userviews/blob/master/app/controllers/home_controller.rb) in [app/controllers/home\_controller.rb](https://github.com/BigFastSite/userviews/blob/master/app/controllers/home_controller.rb)

```ruby
class HomeController < ApplicationController

  FEATURE_ICONS_DIRECTORY = File.join(::Rails.root, "public/img/feature-icons")

  def index
    # Find all the images in the "feature-icons" directory, strip off
    # the directory and extension, to just leave a list of icon names
    @feature_names =
      Dir.entries(File.join(FEATURE_ICONS_DIRECTORY)).
        reject { |f| ! f.end_with? ".png" }.
        map    { |f| f.split(".")[0] }.
        sort
  end
end

```

Yawn. So black and white. So hard to read. Would it not be great if we could display this in color?

Here is how you can convert this code to color...

## Install the CodeRay gem

First we need to install the [CodeRay](https://coderay.rubychan.de/) gem. 

```
gem install coderay
```

## Colorize That Code

The CodeRay gem is very feature rich, but here is a simple out-the-box example that will colorize our code for syntax highlighting...

```
ruby -e '
    require "coderay";
    filename = "app/controllers/home_controller.rb";
    print CodeRay.scan_file(filename).div
'
```

This example will output HTML with inline CSS. This is a good quick and dirty way to get to colorized code, but your front-end CSS person, who likes clean HTML, will probably hate you. Replacing the ".div" with ".html" will not include the CSS and it will be up to you to provide the style for the CSS classes.

## Take It Out Of The Oven

Here is what the colorized code will look like on your blog, webpage or presentation...

```ruby
class HomeController < ApplicationController

  FEATURE_ICONS_DIRECTORY = File.join(::Rails.root, "public/img/feature-icons")

  def index
    # Find all the images in the "feature-icons" directory, strip off
    # the directory and extension, to just leave a list of icon names
    @feature_names =
      Dir.entries(File.join(FEATURE_ICONS_DIRECTORY)).
        reject { |f| ! f.end_with? ".png" }.
        map    { |f| f.split(".")[0] }.
        sort
  end
end

```

And there it is. Not much more to say about that other than how easy it was. Although it was easy the CodeRay gem is very rich.

## Resources

*   [CodeRay Official Website](https://coderay.rubychan.de/)
*   [CodeRay on GitHub](https://github.com/rubychan/coderay)
*   [More CodeRay examples (1470 of them!)](https://coderay.rubychan.de/rays)

<div id="comments">
  <h3 id="comments-number" class="comments-header">9 responses to “Colorful Ruby Code In HTML Using CodeRay”</h3>
  <ol class="comment-list">
    <li id="comment-21217" class="comment byuser comment-author-tednielsen even thread-even depth-1 comment role-subscriber user-tednielsen">
      <img alt="Ted Nielsen" src="https://0.gravatar.com/avatar/6a7c0d92f8bd76d56550bdc8ee1dd5f3?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.facebook.com/theodorenielsen">Ted Nielsen</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, September 3rd, 2012, 2:09 pm">September 3, 2012</abbr> at <abbr class="comment-time" title="Monday, September 3rd, 2012, 2:09 pm">2:09 pm</abbr>
      </div>
      <div class="comment-text">
        <p>I prefer Array.select{ statement } over Array.reject{ !statement }; it lends itself to higher readability and maintenance.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-21228" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, September 3rd, 2012, 7:10 pm">September 3, 2012</abbr> at <abbr class="comment-time" title="Monday, September 3rd, 2012, 7:10 pm">7:10 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks for the tip Ted!</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-21323" class="comment byuser comment-author-rr-rosas even thread-odd thread-alt depth-1 comment role-subscriber user-rr-rosas">
      <img alt="Rodrigo Rosenfeld Rosas" src="https://0.gravatar.com/avatar/2abdb50caf0dc5b510330f68b02db8e4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://plus.google.com/112046748094887665556">Rodrigo Rosenfeld Rosas</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, September 6th, 2012, 6:06 am">September 6, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 6th, 2012, 6:06 am">6:06 am</abbr>
      </div>
      <div class="comment-text">
        <p>The missing feature from CodeRay is lack of support for CoffeeScript <img src="/media/images/smilies/icon_sad.gif" alt=":(" class="wp-smiley" />  I had to implement CS highlighting using client-side libraries in my homepage <img src="/media/images/smilies/icon_sad.gif" alt=":(" class="wp-smiley" />
        </p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-21330" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, September 6th, 2012, 7:56 am">September 6, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 6th, 2012, 7:56 am">7:56 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Rodrigo,</p>
            <p>I missed the fact that CodeRay did not support CoffeeScript. Thanks for pointing that out. Could you share some details about your implementation? I am sure that there are others hitting this post with the same issue.</p>
            <p>Cheers,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-21441" class="comment byuser comment-author-rr-rosas even depth-3 comment role-subscriber user-rr-rosas">
              <img alt="Rodrigo Rosenfeld Rosas" src="https://0.gravatar.com/avatar/2abdb50caf0dc5b510330f68b02db8e4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn" title="https://plus.google.com/112046748094887665556">Rodrigo Rosenfeld Rosas</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Sunday, September 9th, 2012, 12:12 pm">September 9, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 9th, 2012, 12:12 pm">12:12 pm</abbr>
              </div>
              <div class="comment-text">
                <p>Hi Phil, I’m using codemirror for highlighting CoffeeScript in my site. My site is a custom Rails application inspired by Toto’s idea:</p>
                <p>https://codemirror.net/<br />
https://github.com/cloudhead/toto</p>
                <p>My articles are written in Markdown and code examples lie between “@@@ language-identifier”..”@@@” blocks.</p>
                <p>Most are processed using CodeRay but for CoffeeScript I had to use client-side processing cause I couldn’t find any Ruby highlighting library that understands CoffeeScript <img src="/media/images/smilies/icon_sad.gif" alt=":(" class="wp-smiley" />
                </p>
              </div>
              <!-- .comment-text -->
              <ol class="children">
                <li id="comment-21442" class="comment byuser comment-author-admin bypostauthor odd alt depth-4 comment role-administrator user-admin entry-author">
                  <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
                  <div class="comment-meta comment-meta-data">
                    <div class="comment-author vcard">
                      <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
                    </div>
                    <!-- .comment-author .vcard -->
                    <abbr class="comment-date" title="Sunday, September 9th, 2012, 12:39 pm">September 9, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 9th, 2012, 12:39 pm">12:39 pm</abbr>
                  </div>
                  <div class="comment-text">
                    <p>Sweet. Thanks for introducing me to Toto and https://codemirror.net/ looks cool! Pretty rich API https://codemirror.net/doc/manual.html</p>
                    <p>Question: When I’m editing with CodeMirror, is it sending the edited text back to Ruby for re-syntax-highlighting?</p>
                  </div>
                  <!-- .comment-text -->
                  <ol class="children">
                    <li id="comment-21443" class="comment byuser comment-author-rr-rosas even depth-5 comment role-subscriber user-rr-rosas">
                      <img alt="Rodrigo Rosenfeld Rosas" src="https://0.gravatar.com/avatar/2abdb50caf0dc5b510330f68b02db8e4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
                      <div class="comment-meta comment-meta-data">
                        <div class="comment-author vcard">
                          <cite class="fn" title="https://plus.google.com/112046748094887665556">Rodrigo Rosenfeld Rosas</cite>
                        </div>
                        <!-- .comment-author .vcard -->
                        <abbr class="comment-date" title="Sunday, September 9th, 2012, 12:44 pm">September 9, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 9th, 2012, 12:44 pm">12:44 pm</abbr>
                      </div>
                      <div class="comment-text">
                        <p>I never enabled editing to the codes blocks displayed in my articles <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
                        </p>
                        <p>But anyway, Ruby is not involved at all with CodeMirror. It is all handled in the client-side. See an example here:</p>
                        <p>https://codemirror.net/demo/fullscreen.html</p>
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
          </ol>
        </li>
        <!-- .comment -->
        <li id="comment-21348" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, September 6th, 2012, 10:12 pm">September 6, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 6th, 2012, 10:12 pm">10:12 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Pygments, which is similar to CodeRay but is written in Python, supports CoffeeScript. https://pygments.org/docs/lexers/</p>
            <p>This is what GitHub uses for their syntax highlighting.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-21390" class="comment even thread-even depth-1 comment reader">
      <img alt="Jaime Iniesta" src="https://0.gravatar.com/avatar/0e6c0ba9935b52866fd5c54dd886cf2e?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://jaimeiniesta.com">Jaime Iniesta</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, September 8th, 2012, 5:36 am">September 8, 2012</abbr> at <abbr class="comment-time" title="Saturday, September 8th, 2012, 5:36 am">5:36 am</abbr>
      </div>
      <div class="comment-text">
        <p>I recently used CodeRay to colorize syntax in the comments of a rails app, letting the users define the code sections, applying autolinking and simple formatting. Here’s how:</p>
        <p>https://jaimeiniesta.posterous.com/how-to-colorize-code-in-comments-using-codera</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

