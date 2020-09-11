---
title: "A/B Testing in Ruby On Rails"
date: "2012-10-23T13:15:00-07:00"
template: "post"
draft: false
slug: "ab-testing-in-ruby-on-rails"
category: "FluidFeatures"
tags:
  - a/b testing
  - fluidfeatures
  - gem
  - multi-variant testing
  - ruby
  - ruby on rails
description: "In this post I am going to demonstrate, step-by-step, a way to A/B test new features from within Ruby-On-Rails using FluidFeatures. FluidFeatures is currently in closed-beta and requires request for inclusion in the beta program. We are looking for enthusiastic A/B testers, so we can get feedback on the service before releasing it to the masses."
socialImage:
  publicURL: "/images/photo.jpg"
---
<div style="border: 1px #ddd solid; margin: 5px; margin-bottom: 20px; padding: 5px; background-color: #FFFF99;">
<strong>Please note</strong>, the latest and greatest integration steps for the <strong>fluidfeatures-rails</strong> gem can be found on<br/>
<a href="https://github.com/FluidFeatures/fluidfeatures-rails#readme">GitHub</a> at <a href="https://github.com/FluidFeatures/fluidfeatures-rails#readme">https://github.com/FluidFeatures/fluidfeatures-rails#readme</a></div>

In this post I am going to demonstrate, step-by-step, a way to A/B test new features from within Ruby-On-Rails using FluidFeatures. FluidFeatures is currently in closed-beta and requires request for inclusion in the beta program. We are looking for enthusiastic A/B testers, so we can get feedback on the service before releasing it to the masses.

## A/B Testing?

A/B testing or multi-variant testing is the best way to test your hypothesis about what your users want or what works best to get them to do what you want them to do. Generally, when these things come together, your users and your website have found a good fit and you are making a service that is more useful to your users, more intuitive, the right price or sometimes… just the right color. 

With A/B testing you can find answers to questions, such as “Is a list a categories down the side of the web page better, or a search box with a cleaner design?”.

The general idea of A/B testing is that some users will see one version of a website feature and other users see a different version. These two versions of the feature might ultimately provide the same functionality, but provide the user with a slightly different experience, depending on which version they are shown. The success of the test will generally be based on the user taking some action. The winner is determined by whether version A or version B receives the most success hits. If the difference in success hits between A and B is significant, then this is a clear indication of which one you should implement for all of your user-base.

The simplest A/B tests include scenarios like “the button is on the right” vs “the button is on the left” or “the background is green” vs “the background is blue”. Whilst these might sound like pointless exercises, the Gods of A/B testing have decided that sometimes these small changes really do push up the conversion rate. Whether it is the psychology of colors, the genetically encoded aesthetic preferences we all carry or some ancient voodoo, if the statistics are pointing at a winning version of a feature of your website, then you would be a fool to ignore it.

Even a small consistent difference will pay back with compound interest when combined with numerous other changes methodically measured in the same way.

While we all think we know the best solution, the best design and the best UX, in this ever changing world, we really know nothing unless we have the statistics to back it up. Anyone who is doing anything of significance on the web is doing some form of measuring and A/B testing.

## Ruby Or JavaScript?

When I say, “A/B Testing in Ruby On Rails”, you probably think I am talking about something like [Optimizely](https://www.optimizely.com/) or [Google Website Optimizer](https://analytics.blogspot.ca/2012/06/helping-to-create-better-websites.html) (now [Google Content Experiments](https://analytics.blogspot.ca/2012/06/helping-to-create-better-websites.html)). These are great solutions and if you are serious about A/B testing your Ruby-On-Rails application then you should [check them out](https://www.youtube.com/watch?v=0S0IrbwpfzE). These solutions are JavaScript solutions which work by dropping a JavaScript snippet onto your webpage and changing the user’s experience at the browser level. The JavaScript will make the choice of what the current user sees by changing the DOM (HTML on the page) or redirecting them to another page.

Managing this in JavaScript makes it easy to setup, but it also means that all the A/B versions are available to all users and are just obscured by the browser. The server has to potentially render the full content for both versions for each visitor. This is not ideal if you are testing a beta-version of a feature and you do not want it to affect all your users. Or you are testing pricing or new features where you really do not want any chance of someone hacking your JavaScript to expose the unpublished features. Your marketing team would not be happy.

So, in short, I talking about taking this down to the Ruby level on the server-side.

## Benefits of Server-Side A/B Testing

When a user visits your website, the earlier you know which features they will see, or which versions of which features, the less work you have to do. It also opens up a whole new world of A/B testing. Have you ever wanted to test different variations of an SQL query against each other in production and see which is faster? Why not test the newer SQL query on 1% of your user-base?

If your A/B testing originates on the server-side, a feature version’s per-user enabled status is a simple boolean that is now available at _every_ level of your architecture, not _just_ in the browser. You can look at the impact of that enabled status in aggregate against specific goals on your website (e.g. “user clicks on the Buy button”) or offline (e.g. “user sends us an email”) using the same system.

If you manage your A/B testing on the server-side then it is easier to push information up to the browser for inclusion in Google Analytics. If you manage it in the browser, using JavaScript, then there is little the server-side can do once the web page is already rendered.

## Let’s Do It!

First we add the __fluidfeatures-rails__ gem to your Rails application’s Gemfile.

```
gem "
```

In your __app/controllers/application\_controller.rb__ file you need a method that fluidfeatures-rails can call to find out who the current user is, or if the user is anonymous. In our example we call this __get\_current\_user__, which will return a Hash with at least and __:id__ value.

__user\[:id\]__ is the unique identifier of this user. If we do not have this then we assume this is an anonymous (not logged in) user. If the user is anonymous then fluidfeatures-rails generates its own anonymous-user-id and sets a cookie so that the anonymous user receives a consistent experience on each request.

```ruby
def fluidfeatures_current_user(verbose=false)
  current_user = get_current_user
  if current_user
    # logged in user
    { :id => current_user[:id] }
  else
    # anonymous user
    nil
  end
end

```

Next, load the fluidfeatures-rails initializer at the end of you Applications instantiation.

```ruby

module MyRailsApp
  class Application < Rails::Application

    FluidFeatures::Rails.initializer
  end
end

```

Great. Now we can start plugging in our feature versions into our request handler. We have one feature called “ab-test” with version “a” and “b”…

```ruby
class AbTestController < ApplicationController
  def index
    if fluidfeature("ab-test", { :version => "a" })
      # Our version "A" code here
    end

    if fluidfeature("ab-test", { :version => "b" })
      # Our version "B" code here
    end
  end
end
```

## Hooking Up With The Dashboard

Once we are logged into FluidFeatures.com, we can create a new “application”. This can be found at <https://www.fluidfeatures.com/dashboard>

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/new-application.png)

Give our application a name.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/input-app-name.png)

Find our application’s credentials.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/click-credentials.png)

We will copy these credentials and pass them to our rails application as environment variables.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/credentials.png)

## Flesh Out Our App

app/controllers/application\_controller.rb

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery

  def get_current_user
    # this app does not have a concept of logged-in users
    nil
  end

  def fluidfeatures_current_user(verbose=false)
    current_user = get_current_user
    if current_user
      # logged in user
      { :id => current_user[:id] }
    else
      # anonymous user
      {}
    end
  end

end

```

app/controllers/ab\_test\_controller.rb

```ruby
class AbTestController < ApplicationController
  def index
    if fluidfeature("ab-test", { :version => "a" })
      # Our version "A" code here
      render :text => "Do you like Apples? <a href=\"yes\">YES</a> | <a href=\"no\">NO</a>"
      return
    end
    if fluidfeature("ab-test", { :version => "b" })
      # Our version "B" code here
      render :text => "Do you like Bananas? <a href=\"yes\">YES</a> | <a href=\"no\">NO</a>"
      return
    end
    render :text => "Nothing to see here"
  end

  def yes
    fluidgoal("yes")
    render :text => "Great. Me too!"
  end

  def no
    fluidgoal("no")
    render :text => "That's a shame. I love them!"
  end

end

```

## Running Our App

We pass in the FluidFeatures application credentials we obtained from the dashboard (see above) when we run __script/rails server__.

```
FLUIDFEATURES_APPID=1wy1vvnue4tgh \
FLUIDFEATURES_SECRET=jzJm2JA2mhURGC8DOivVbt23h5rt1GSfFaasCy5s \
FLUIDFEATURES_BASEURI=https://local.fluidfeatures.com/service \
script/rails server
=> fluidfeatures-rails initializing as app 1wy1vvnue4tgh with https://local.fluidfeatures.com/service
=> Booting WEBrick
=> Rails 3.2.8 application starting in development on https://0.0.0.0:3000
=> Call with -d to detach
=> Ctrl-C to shutdown server
[2012-10-22 14:31:37] INFO  WEBrick 1.3.1
[2012-10-22 14:31:37] INFO  ruby 1.9.3 (2012-04-20) [x86_64-darwin11.4.0]
[2012-10-22 14:31:37] INFO  WEBrick::HTTPServer#start: pid=61664 port=3000

```

fluidfeatures-rails detects new features when it encounters them at run-time, then reports them back to the dashboard. When we make our first make I request, neither “a” or “b” versions are enabled.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/nothing-to-see.png)

But during this request we have reported back to the fluidfeatures service that we have a new feature “ab-test” and we have detected two versions, “a” and “b”.

If we look at the dashboard we will see this feature.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/ab-test-feature-detected.png)

## Configuring The Split

Let’s set version “a” to 75% exposure and version “b” to 25% exposure. This means that 75% of the visitors will see version “a” and 25% will see version “b”.

From the code above we can see that version “a” will show the message “Do you like Apples?” and version “b” will show the message “Do you like Bananas?”. Both will give links to “YES” and “NO”. These “YES” and “NO” links are our goals. We want our users to click these links and we will track how successful each version of our question is.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/ab-test-75-25.png)

## And We Are Live!

If we now visit the application, instead of seeing “Nothing to see here”, we know we have a 75% chance of seeing “Do you like Apples?” and a 25% chance of seeing “Do you like Bananas?”.

We see “Do you like Bananas?”.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/do-you-like-bananas.png)

## Keeping Score

Let’s click on “YES” and indicate that we do like bananas.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/great-me-too.png)

Meanwhile, back on the dashboard, we see the detected goal pop-up.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/one-banana-lover.png)

Statistically not very significant, but we can see we are tracking some statistics.

## Where’s The Apples?

No matter how many times I refresh my browser, I see “Do you like Bananas?” due to fluidfeatures-rails giving me a consistent experience while using this application.

If I open the application in a different browser, I now see “Do you like Apples?” and I can click “YES” there, too.

## Statistical Significance

Over time, we get more clicks on “YES” and “NO” for “apples” and “bananas”. This application is really popular!

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/yes-goal.png)

We can see that “a” (for “apples”) received more clicks than “b” (for “bananas”). 29 for “a” vs 20 “b”. Although, from the percentages it appears that “b” is more than twice as popular. How can that be?

If we look back at the feature settings, we are only showing “b” to 25% of the users that come to our site, so each click is much more statistically significant than the clicks we receive from “a”, which is shown to 75% of our users.

We can change the exposure level that each version receives and FluidFeatures will keep track of the statistical significance of each goal (“YES” click) at the time it occurs.

![](https://commondatastorage.googleapis.com/philwhln/blog/images/ab-test-rails/full-dashboard.png)

## Download The Code

You can download the code from the application at  
<https://github.com/FluidFeatures/MyRailsApp>

## Learn More

In this video, we demonstrate FluidFeatures for feature rollout and A/B testing.

<div class="wistia_embed" data-video-height="400" data-video-width="640" id="wistia_cu1iy2f0vn" style="width:640px;height:400px;">
<object classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" id="wistia_cu1iy2f0vn_seo" style="display:block;height:400px;position:relative;width:640px;">
<param name="movie" value="https://embed-0.wistia.com/flash/embed_player_v2.0.swf?2012-06-01"/>
<param name="allowfullscreen" value="true"/>
<param name="allowscriptaccess" value="always"/>
<param name="bgcolor" value="#000000"/>
<param name="wmode" value="opaque"/>
<param name="flashvars" value="banner=true&amp;controlsVisibleOnLoad=true&amp;hdUrl%5Bheight%5D=800&amp;hdUrl%5Btype%5D=hdflv&amp;hdUrl%5Burl%5D=https%3A%2F%2Fembed-0.wistia.com%2Fdeliveries%2Ff8e538273e5d816ecc847577438edb59cd78db87.bin&amp;hdUrl%5Bwidth%5D=1280&amp;mediaDuration=522.7&amp;showVolume=true&amp;stillUrl=https%3A%2F%2Fembed-0.wistia.com%2Fdeliveries%2F1134c1888f3ffc77f96ad2242984b943e67a16c3.jpg%3Fimage_crop_resized%3D640x400&amp;unbufferedSeek=true&amp;videoUrl=https%3A%2F%2Fembed-0.wistia.com%2Fdeliveries%2F26b9223ed5ac05de797190485ae6ca65fc686186.bin"/>
<embed allowfullscreen="true" allowscriptaccess="always" bgcolor="#000000" flashvars="banner=true&amp;controlsVisibleOnLoad=true&amp;hdUrl%5Bheight%5D=800&amp;hdUrl%5Btype%5D=hdflv&amp;hdUrl%5Burl%5D=https%3A%2F%2Fembed-0.wistia.com%2Fdeliveries%2Ff8e538273e5d816ecc847577438edb59cd78db87.bin&amp;hdUrl%5Bwidth%5D=1280&amp;mediaDuration=522.7&amp;showVolume=true&amp;stillUrl=https%3A%2F%2Fembed-0.wistia.com%2Fdeliveries%2F1134c1888f3ffc77f96ad2242984b943e67a16c3.jpg%3Fimage_crop_resized%3D640x400&amp;unbufferedSeek=true&amp;videoUrl=https%3A%2F%2Fembed-0.wistia.com%2Fdeliveries%2F26b9223ed5ac05de797190485ae6ca65fc686186.bin" name="wistia_cu1iy2f0vn_html" src="https://embed-0.wistia.com/flash/embed_player_v2.0.swf?2012-06-01" style="display:block;height:100%;position:relative;width:100%;" type="application/x-shockwave-flash" wmode="opaque"/>
</object>
</div>

<script charset="ISO-8859-1" src="https://fast.wistia.com/static/concat/E-v1%2CpostRoll-v1.js"></script>

  

<script>
wistiaEmbed = Wistia.embed("cu1iy2f0vn", {
  version: "v1",
  videoWidth: 640,
  videoHeight: 400,
  volumeControl: true,
  controlsVisibleOnLoad: true
});
Wistia.plugin.postRoll(wistiaEmbed, {
  version: "v1",
  text: "Click here to visit&lt;br/&gt;FluidFeatures.com",
  link: "https://www.fluidfeatures.com/?utm_source=bigfastblog.com&amp;#038;utm_medium=video&amp;#038;utm_content=ab_testing_in_rails&amp;#038;utm_campaign=interest",
  style: {
    backgroundColor: "#616161",
    color: "#ffffff",
    fontSize: "36px",
    fontFamily: "Gill Sans, Helvetica, Arial, sans-serif"
  }
});
</script>

  

<script charset="ISO-8859-1" src="https://fast.wistia.com/embed/medias/cu1iy2f0vn/metadata.js"></script>

<https://github.com/FluidFeatures/userviews>

## FluidFeatures Trial

FluidFeatures is a new service and the most important thing for the service right now is quality feedback from quality users. We are in closed beta right now and looking for some great Rails developers to start A/B testing their applications using FluidFeatures.

Please sign-up to FluidFeatures Beta Users mailing-list if you are a Ruby On Rails developer and you are interested in using this service.

<!-- Begin MailChimp Signup Form -->

<link href="https://cdn-images.mailchimp.com/embedcode/slim-081711.css" rel="stylesheet" type="text/css"/>

<div id="mc_embed_signup">
<form action="https://scalablecreations.us2.list-manage.com/subscribe/post?u=707d1f1a27e889d195e47a636&amp;id=d7b1c949a8" class="validate" id="mc-embedded-subscribe-form" method="post" name="mc-embedded-subscribe-form" novalidate="" target="_blank">
<input class="email" id="mce-EMAIL" name="EMAIL" placeholder="email address" required="" type="email" value=""/>
<div class="clear">
<input class="button" id="mc-embedded-subscribe" name="subscribe" type="submit" value="Subscribe"/></div>
</form>
</div>

## Updated

*   Changed fluidfeature\_current\_user to fluidfeature__s__\_current\_user <small>(2013-01-17)</small>

<!--End mc_embed_signup-->

## Posted in : [ Ruby](/category/ruby-2)

<div id="comments">
  <h3 id="comments-number" class="comments-header">One response to “A/B Testing in Ruby On Rails”</h3>
  <ol class="comment-list">
    <li id="comment-23676" class="comment even thread-even depth-1 comment reader">
      <img alt="alex kuznetsov" src="https://1.gravatar.com/avatar/d03097fc0e24158dcddf952f9d4d0800?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://cophenetic.com">alex kuznetsov</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, October 23rd, 2012, 8:24 pm">October 23, 2012</abbr> at <abbr class="comment-time" title="Tuesday, October 23rd, 2012, 8:24 pm">8:24 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Interesting gem. I don’t usually use these often, but it has a nifty ui, so I might give it a try.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

