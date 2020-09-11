---
title: "Fluid Feature Rollout For Ruby On Rails"
date: "2012-09-13T17:25:00-07:00"
template: "post"
draft: false
slug: "fluid-feature-rollout-for-ruby-on-rails"
category: "FluidFeatures"
tags:
  - continuous integration
  - degrade
  - feature
  - fluidfeatures
  - lean
  - production
  - rollout
  - ruby
  - ruby on rails
description: "In this blog post I am going to introduce a new service for Ruby On Rails developers called FluidFeatures that helps you manage rolling out new features to your site and provides a way to easily do A/B testing (competing versions of a feature) right in the belly of your code."
socialImage:
  publicURL: "/media/images/photo.jpg"
---
In this blog post I am going to introduce a new service for Ruby On Rails developers, that I have been working on, called [FluidFeatures](https://fluidfeatures.com/) that helps you manage rolling out new features to your site and provides a way to easily do A/B testing (competing versions of a feature) right in the belly of your code.

I wrote a post back in December called, [“Rollout, Degrade, Metrics and Capacity”](/rollout-degrade-metrics-and-capacity) which focused heavily on the Ruby gems [rollout](https://github.com/jamesgolick/rollout) and [degrade](https://github.com/jamesgolick/degrade), created by [James Golick](https://jamesgolick.com/). If you’ve used those popular gems or are thinking of using them, then [FluidFeatures](https://fluidfeatures.com/) is also worth a look, though offers a slightly different feature set.

```

# This is an existing feature, so we set it's initial enabled state to fully on.
# This will be the "default" version of the feature.
if fluidfeature("weather", { :enabled => true })
  @icons << "weather"
end
# This version of the "weather" feature we're calling "rainbows".
# When we first rollout this feature, it will be fully disabled.
if fluidfeature("weather", { :version => "rainbows" })
  @icons << "rainbows"
end
if fluidfeature("weather", { :version => "sunshine" })
  @icons << "sunshine"
end

```

## More Coming Soon…

The above example code snippets are part of demo application we have built called [UserViews](https://github.com/BigFastSite/userviews) that you can download from [github.com](https://github.com/BigFastSite/userviews)

In a follow-up post I will demonstrate, step-by-step, how to install the [fluidfeatures-rails gem](https://rubygems.org/gems/fluidfeatures-rails) and the demo application, [UserViews](https://github.com/BigFastSite/userviews), so that you can try out FluidFeatures for yourself.
