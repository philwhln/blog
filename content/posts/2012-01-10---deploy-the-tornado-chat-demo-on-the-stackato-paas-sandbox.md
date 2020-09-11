---
title: "Deploy The Tornado Chat Demo On The Stackato PaaS Sandbox "
date: "2012-01-10T09:57:00-08:00"
template: "post"
draft: false
slug: "deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox"
category: "Infrastrucutre"
tags:
  - activestate
  - clojure
  - cloudfoundry
  - java
  - mongodb
  - paas
  - perl
  - python
  - ruby
  - sandbox
  - stackato
  - tornado
  - vmware
description: "In this blog post I'll go through an example, in repeatable steps, of how to get up and running with the Tornado chat demo on ActiveState's public sandbox for Stackato. ActiveState's Stackato PaaS (Platform-as-a-Service) is based on VMware's open-source PaaS, Cloud Foundry, and offers an enterprise PaaS solution that will run on any public cloud, private cloud, laptop or desktop."
socialImage:
  publicURL: "/images/photo.jpg"
---
<img align="left" src="https://commondatastorage.googleapis.com/philwhln/blog/images/stackato-tornado-demo/stackato-logo.png"/>

[ActiveState](https://www.activestate.com/) have recently released their [Stackato](https://www.activestate.com/cloud) PaaS (Platform-as-a-Service) which is based on VMware’s open-source PaaS, [Cloud Foundry](https://www.cloudfoundry.com/), and offers an enterprise PaaS solution that will run on any public cloud, private cloud, laptop or desktop.

In this blog post I’ll go through an example, in repeatable steps, of how to get up and running with the [Tornado chat demo](https://github.com/facebook/tornado/tree/master/demos/chat) on ActiveState’s public sandbox for Stackato.

*   [Get An ActiveState Account](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#get-an-activestate-account)
*   [Install The Stackato Client](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#install-the-stackato-client)
*   [Sandbox Terms and Services](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#sandbox-terms-and-services)
*   [Obtaining Stackato Sandbox Access](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#obtaining-stackato-sandbox-access)
*   [Deploying The Demo Application](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#deploying-the-demo-application)
*   [Our App In Action](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#our-app-in-action)
*   [Stop The Chatter](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#stop-the-chatter)
*   [Conclusion](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#conclusion)
*   [Resources](/deploy-the-tornado-chat-demo-on-the-stackato-paas-sandbox#resources)

[Tornado](https://www.tornadoweb.org/) is an event-driven application framework written in Python. It is used by [Quora](/quoras-technology-examined), [Summify](/summifys-technology-examined), [Facebook](https://www.facebook.com/), [Friendfeed](https://friendfeed.com/) and my own [Cooq.com](https://www.cooq.com). Stackato supports a multitude of languages and frameworks, so Tornado could be swapped out with one of the many other [demos](https://github.com/ActiveState/stackato-samples) available, which include [PHP](https://github.com/ActiveState/stackato-samples/tree/master/php), [Perl](https://github.com/ActiveState/stackato-samples/tree/master/perl), [Ruby](https://github.com/ActiveState/stackato-samples/tree/master/ruby), [Node.js](https://github.com/ActiveState/stackato-samples/tree/master/node), [Java](https://github.com/ActiveState/stackato-samples/tree/master/java) or [Clojure](https://github.com/ActiveState/stackato-samples/tree/master/clojure). Although the JVM languages (Java and Clojure) are currently not supported due to their heavy resource requirements. This demo application also uses [MongoDB](https://www.mongodb.org/) as its backend datastore.

<h2 id="get-an-activestate-account">Get An ActiveState Account</h2>

[Sign up for an ActiveState account](https://account.activestate.com/signup/?activationnext=https%3A%2F%2Fcommunity.activestate.com%2Fstackato)

You get the usual verification email to confirm your email address.

<h2 id="install-the-stackato-client">Install The Stackato Client</h2>

The server software and various virtual machines, which contain the pre-installed server software, is [available for download](https://community.activestate.com/stackato/download), we do not need them, since here we are using the ActiveState’s sandbox for all our server requirements.

[Download Stackato Client](https://community.activestate.com/stackato/download) (v0.18 at time of writing this).

```
unzip stackato-0.3.13.0.18-macosx10.5-i386-x86_64.zip
chmod +x stackato-0.3.13.0.18-macosx10.5-i386-x86_64
mv stackato-0.3.13.0.18-macosx10.5-i386-x86_64 /usr/local/bin/stackato

```

_Note: Be sure /usr/local/bin is in your $PATH before continuing_

<h2 id="sandbox-terms-and-services">Sandbox Terms and Services</h2>

I thought I’d include the limitations of what you can do with the Sandbox.

*   You can use the Sandbox at no charge for up to 45 days
*   It is not for production use
*   Applications launched will resolve to a \*.sandbox.activestate.com URL
*   Account resources is limited to 256Mb application memory, 10Gb disk-space and 50Gb bandwidth
*   Maximum of 2 applications (Python, Perl, Ruby, Node.Js, or other languages as supported in the Sandbox)
*   Maximum of 2 services which represent a given database, such as MySQL, MongoDB, Redis, or Postgres

You should read the [full terms and services](https://docs.stackato.com/sandbox.html) for more details.

<h2 id="obtaining-stackato-sandbox-access">Obtaining Stackato Sandbox Access</h2>

Request access to the Stackato Sandbox from the main [Stackato page](https://community.activestate.com/stackato).

You can find your Sandbox username and password on your [ActiveState account page](https://account.activestate.com/).

<h2 id="deploying-the-demo-application">Deploying The Demo Application</h2>

First inform the stackato client which Stackato installation your will be deploying to. ActiveState’s Stackato Sandbox is found at _https://api.sandbox.activestate.com_, so we run the following command.

```
$ stackato target https://api.sandbox.activestate.com

Successfully targeted to [https://api.sandbox.activestate.com]
```

Next, login to Stackato using _stackato login_.

```
$ stackato login

Email: <your email>
Password: ****************
Successfully logged into [https://api.sandbox.activestate.com]

```

Checkout the demo applications, from github, locally onto your machine.

```
$ cd ~/src
$ git clone https://github.com/ActiveState/stackato-samples

Cloning into stackato-samples...
remote: Counting objects: 2353, done.
remote: Compressing objects: 100% (1152/1152), done.
remote: Total 2353 (delta 1067), reused 2319 (delta 1033)
Receiving objects: 100% (2353/2353), 1.05 MiB | 275 KiB/s, done.
Resolving deltas: 100% (1067/1067), done.

```

Choose your demo application by simply cd’ing into that directory…

```
$ cd stackato-samples/python/tornado-chat-mongo
```

The first time I ran the deployment I just accepted all the defaults…

```
$ stackato push tornado-chat

Would you like to deploy from the current directory ?  [Yn]:
Application Deployed URL [tornado-chat.sandbox.activestate.com]:
Application Url: tornado-chat.sandbox.activestate.com
Detected a Python Application, is this correct ?  [Yn]:
Memory Reservation [64M]:
Creating Application [tornado-chat]: Error 701: The URI: 'tornado-chat.sandbox.activestate.com' has already been taken or reserved

```

The default domain _tornado-chat.sandbox.activestate.com_ is already taken, so I’ll need to come up with another one.

Let’s run _stackato push tornado-chat_ again, but this time use a custom sub-domain…

```
$ stackato push tornado-chat

Would you like to deploy from the current directory ?  [Yn]:
Application Deployed URL [tornado-chat.sandbox.activestate.com]: bigfastblog-chat.sandbox.activestate.com
Application Url: bigfastblog-chat.sandbox.activestate.com
Detected a Python Application, is this correct ?  [Yn]:
Memory Reservation [64M]:
Creating Application [tornado-chat]: OK
Creating mongodb Service [mongo-chat]: OK
Binding Service [mongo-chat]: OK
Uploading Application [tornado-chat]:
  Checking for available resources: OK
  Packing application: OK
  Uploading (7K): 100% OK
Push Status: OK
Staging Application [tornado-chat]: OK
Starting Application [tornado-chat]: OK
```

<h2 id="our-app-in-action">Our App In Action</h2>

```
$ stackato apps

+--------------+---+---------+------------------------------------------+------------+
| Application  | # | Health  | URLS                                     | Services   |
+--------------+---+---------+------------------------------------------+------------+
| tornado-chat | 1 | RUNNING | bigfastblog-chat.sandbox.activestate.com | mongo-chat |
+--------------+---+---------+------------------------------------------+------------+
```

On visiting my custom sub-domian _bigfastblog-chat.sandbox.activestate.com_, we encounter the Google OAuth login.

>  
> Bigfastblog-chat.sandbox.activestate.com is asking for some information from your Google Account
> 

After clicking “Allow”, we’re taken to the rather basic chat interface that is the Tornado Chat demo.

We are logged in our as Google Accounts user and can start chatting!

![](https://commondatastorage.googleapis.com/philwhln/blog/images/stackato-tornado-demo/chat.png)

<h2 id="stop-the-chatter">Stop The Chatter</h2>

We can stop the application, if we intend to restart again.

```
$ stackato stop tornado-chat
$ stackato apps

+--------------+---+---------+------------------------------------------+------------+
| Application  | # | Health  | URLS                                     | Services   |
+--------------+---+---------+------------------------------------------+------------+
| tornado-chat | 1 | STOPPED | bigfastblog-chat.sandbox.activestate.com | mongo-chat |
+--------------+---+---------+------------------------------------------+------------+
```

Encase you are curious, this is what you’ve seen when you visit your sub-domain for this app…

![](https://commondatastorage.googleapis.com/philwhln/blog/images/stackato-tornado-demo/deleted.png)

```
VCAP ROUTER: 404 - DESTINATION NOT FOUND

No application is currently registered at this URL. If you are the site owner,
please make sure that your application is in RUNNING state by running the
`stackato stats 
```

Or we can simply delete it from the Sandbox.

```
$ stackato delete tornado-chat

Provisioned service [mongo-chat] detected would you like to delete it ?:  [yN]: y
Deleting application [tornado-chat]: OK
Deleting service [mongo-chat]: OK

$ stackato apps

No Applications
```

<h2 id="conclusion">Conclusion</h2>

The Stackato sandbox gives you a great way to get a small taste of what Stackato is all about. When you graduate past using the sandbox, you will want to look at the “micro-cloud”. It’s free indefinitely for non-commercial use.

<h2 id="resources">Resources</h2>

*   [The Official Stackato Website](https://community.activestate.com/stackato)
*   [\[YouTube\] Stackato Demo – Micro Cloud to Private PaaS](https://www.youtube.com/watch?v=CunjLaS7Rzo)
*   [Stackato Resources](https://www.activestate.com/cloud#resources)
*   [Other Stackato Demo Applications](https://github.com/ActiveState/stackato-samples)
*   [Cloud Foundry Homepage](https://www.cloudfoundry.com/)

<div id="comments">
  <h3 id="comments-number" class="comments-header">2 responses to “Deploy The Tornado Chat Demo On The Stackato PaaS Sandbox”</h3>
  <ol class="comment-list">
    <li id="comment-7782" class="comment even thread-even depth-1 comment reader">
      <img alt="jiaxiaolei" src="https://1.gravatar.com/avatar/b2bf7f0f9e98806ba6cfc56f5dd66ab9?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">jiaxiaolei</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, January 10th, 2012, 10:57 pm">January 10, 2012</abbr> at <abbr class="comment-time" title="Tuesday, January 10th, 2012, 10:57 pm">10:57 pm</abbr>
      </div>
      <div class="comment-text">
        <p>hi, phil:</p>
        <p>I read your words about stackato in python-tornado groups, and according to your gudiance I finished a server in  ~/src/stackato-samples/python of my own computer. you can find it in “https://jia-tornado-chat.sandbox.activestate.com/”</p>
        <p>A couple advises may be provided as follows:</p>
        <p>1: when you try to login, the default url is “https://api.stackato.local”, so :</p>
        <p>$ stackato login<br />
Attempting login to [https://api.stackato.local]<br />
Email: jiaxiaolei19871112@gmail.com<br />
Password: ****************<br />
Cannot access target (couldn’t open socket: host is unreachable (Name or service not known)</p>
        <p>In my view, some description like below should be appeared in your blog, do you think so ? <img src="/images/smilies/icon_biggrin.gif" alt=":D" class="wp-smiley" />
        </p>
        <p>$ stackato target  https://api.sandbox.activestate.com<br />
Successfully targeted to [https://api.sandbox.activestate.com]</p>
        <p>2：Use the tornado-chat-mongo, I push my app in ActiveState. You can see in the url https://jia-tornado-chat.sandbox.activestate.com/<br />
okay, compared with your blog,  it’s ugly!<br />
so, can you tell me what did you do about the blog we were watching ？<br />
Any response will be welcome!</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-8135" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, January 19th, 2012, 6:39 am">January 19, 2012</abbr> at <abbr class="comment-time" title="Thursday, January 19th, 2012, 6:39 am">6:39 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Jiaxiaolei,</p>
            <p>Thanks for the comment on my blog post. I saw your follow-up post and was glad you were also able to fire up the Tornado Chat Demo app!</p>
            <p>Yes, I missed a step. The “stackato target” command. Well spotted! I’ll update the post accordingly.</p>
            <p>Thanks,<br />
Phil</p>
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

