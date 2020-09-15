---
title: "DevOps: Hero Culture"
date: "2014-01-23T00:00:00-07:00"
template: "post"
draft: true
slug: "devops-hero-culture"
category: "DevOps"
tags:
  - devops
description: "A common theme I hear spoken of is that of the firefighters. Firefighters are the ones who are often up at 3am frantically trying to keep systems online. The flip-side to this is the fire preventers. Fire preventers are those that proactively build systems to prevent outages occurring and safeguarding them when they do."
---
_This post was originally published on [ActiveState's blog](https://www.activestate.com/blog/), but is longer available there._

![](/media/images/devops-hero-culture.jpg)
_Image courtesy of [thenationalguard@flickr](https://www.flickr.com/photos/thenationalguard/4443666692/)_

I often speak with Operations people, whether it be at conferences, meetups, through the grapevine, or random encounters. A common theme I hear spoken of is that of the firefighters. Firefighters are the ones who are often up at 3am frantically trying to keep systems online. The flip-side to this is the fire preventers. Fire preventers are those that proactively build systems to prevent outages occurring and safeguarding them when they do.

Most Operations teams consist of both firefighters and preventers. It is also likely that the same people are playing both roles. The problem is that sometimes they are too busy putting out fires to step back and look for preventative measures.

Fire fighting is a noble job, a job for the brave, it saves companies from outage, from loss in revenue, from loss due to bad publicity. Yes, it is job for heroes. And this is where the problem lies.

## Awesome Firefighters!

It’s 2am. Do you know where your database backup is? Frank, in your Operations team, does and he has been head down in his laptop for the past hour trying to get it re-instantiated into a new master database server. Unfortunately, this machine has been sitting dormant for a while, and it is not up-to-date with the latest security fixes and patches. Frank is not totally familiar with building or updating machines, so Spencer is also up with him trying to update the machine. As usual, it is the perfect 2am storm. Everything that can be broken, is.

By 4am, things are back online and all the pizza has been consumed. Frank and Spencer head home and attempt to sleep regardless of the Red Bull coursing through their bloodstream. Nobody would deny that they fought a brave fight here tonight and they go home self-confessed heroes.

At 8am, other members of the IT team start to come. Most are unaware of the awesomeness that occurred in this very same building last night. Right here in this office!

Word gets around. They might get a standing ovation when they come in at 11am, or maybe just a big smile and a pat on the back from the head of IT. Either way, they know what they did.

## Fire Preventers

It’s 2pm. Trevor has just finished writing a Jenkins job that will rebuild a new version of the master database every day with 99% of the data already loaded onto it. He heard about what happened last night and thought that this would be a great way reduce the downtime that occurred during the night. He also has a idea how he might be able to automatically instantiate the new machine when the primary database goes down.

The head of IT begrudgingly signed off the the virtual machine infrastructure that Trevor’s colleague, Jason, proposed would be best for this kind of thing.

They have been building many such systems over the past 6 months, but so far nothing has made it into production. This is mainly due to lack of confidence in switching over some major infrastructure to the new system. The head of IT, Boris, has consulted with his top firefighters, Frank and Spencer, and they are not confident with the new technology: “What if it causes an outage?”, “We understand the current system well and can quickly fix things that go wrong”, “If it ain’t broke, why fix it?”.

## DevOps Buy-In

Trevor and Jason are very interesting in doing things in a more DevOps way. They are proactively building systems with in-built resilience, predictability and repeatability. This is great for our fictional IT department and will result in a lot less outages in the long-run, even if there are some teething problems in the short-term. Unfortunately, they do not have buy-in from their head of IT, Boris.

Boris is unable to see past the short-term issues and to the long-term gains. He also looks to his trusted “heroes” for guidance, but their judgement might not be taking into account the bigger picture and the long-term view.

The blocker here could simply be that their department has come down with a case of “hero culture”.

## Hero Culture

Hero culture is a blocker to DevOps for several reasons.

For a non-metrics driven organization, 3am fire-fighting is something you can quantify with your gut. The systems go down and fire-fighting fixes it. Proactive measures, on the other hand, are harder to quantify. The systems are up and proactive measures keep the systems up. So what?

Maybe integrating the proactive measures actually causes a temporary distribution, in which case they may be seen as a negative. Worst case scenario is that this causes a recoil, and the project gets shelved.

3am fire-fighting is exciting. When you are in race against time and you are using all your wits to slay the beast that is a network outage, you are going to have a fair bit of adrenaline coursing through your body. Pair that with the dopamine that is released with each step closer you get to a solution. It is going to be hard to take a desk job after fighting on the front lines. So when someone suggests a way to do away with the excitement that is bringing meaning into your life, there might just be a tiny bit of you that resists the change.

Now imagine that most of the team is involved in fire-fighting. New recruits see the older recruits getting praised for their brave work in the line-of-fire and they want that kind of praise and reward too. Before long everyone is focused on putting out fires and it is no ones interest to step back and take on the risks that long-term DevOps-focused goals entail.

## Conclusion

If you have a quorum of fight-fighters, who actually thrive on fire-fighting, then introducing proactive preventative measures may be difficult. We humans hate change at the best of times, even if it is for a long-term benefit.

It is easier to see, and reward, those doing battle than those seeking peace. The fight for peace, just as for DevOps, is never over. It is a continuous unrewarding struggle. A 3am fire-fight, on the other hand, is usually done by 4 or 5am, and the achievements are clear.

But let us not dismiss the firefighters. They will always be needed and without them our systems would surely fail.
