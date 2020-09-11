---
title: "Ruby exit, exit!, SystemExit and at_exit blunder"
date: "2012-09-08T15:25:00-07:00"
template: "post"
draft: false
slug: "ruby-exit-exit-systemexit-and-at_exit-blunder"
category: "Ruby"
tags:
  - at_exit
  - exception
  - exit
  - process management
  - rails
  - rescue
  - ruby
  - ruby on rails
  - supervisord
  - systemexit
description: "Recently, we hit a problem with Ruby's exit command. If something went horribly wrong and it made no sense for our application to continue in its current state then we would abort with exit 1. We use supervisord to manage processes, so in this case when we exited with exit status of 1, supervisord would assume something went wrong and restart the process for us. Or at least that is what we thought..."
socialImage:
  publicURL: "/media/images/photo.jpg"
---
Recently, we hit a problem with Ruby’s “[exit](https://www.ruby-doc.org/core-1.9.3/Kernel.html#method-i-exit)” command. If something went horribly wrong and it made no sense for our application to continue in its current state then we would abort with “exit 1″. We use [supervisord](https://supervisord.org/) to manage processes, so in this case when we exited with exit status of 1, [supervisord](https://supervisord.org/) would assume something went wrong and restart the process for us. Or at least that is what we thought…

“exit 1″ does not actually cause the process to exit, it just raises a [SystemExit](https://www.ruby-doc.org/core-1.9.3/SystemExit.html) exception. This is very clearly explained in the [documentation](https://www.ruby-doc.org/core-1.9.3/Kernel.html#method-i-exit). 

```ruby
begin
  exit
  puts "never get here"
rescue SystemExit
  puts "rescued a SystemExit exception"
end
puts "after begin block"

```

In our case, nothing was catching the exception. So what was it?

The other way to handle exits is to run an “[at\_exit](https://www.ruby-doc.org/core-1.9.3/Kernel.html#method-i-at_exit)” block. Ruby runs any “at\_exit” block when it gets an “exit” call, but does not run these with a “exit!” call.

Here is what our rather naive at\_exit block looked like…

```ruby
at_exit do

  # do cleanup

  # now exit for real
  exit

end

```

This harmless looking piece of code was turning our “exit 1″ into an “exit 0″. When supervisord sees one of its processes exit with a status of zero, it assumes all is good and does not try to restart the process. This is big problem for uptime and a major reason for using a tool like supervisord.

Instead we should be using “exit!” if we want to die hard \[with a vengeance\] and be sure that the process exited with a status of 1 and is restarted by supervisord. This would bypass all SystemExit rescue blocks and at\_exit blocks. 

Alternatively, and a better solution, is that we never call “exit” inside an “at\_exit” block and we make sure that all SystemExit rescue blocks and at\_exit blocks are used with great caution and echo the original exit status when necessary.

## Update

[Jonathan Rochkind](https://bibwild.wordpress.com/) made a really great clarification of the exit / at\_exit situation in [a comment](/ruby-exit-exit-systemexit-and-at_exit-blunder/comment-page-1#comment-21913) below. I think it is important to read it.

## Posted in : [ Ruby](/category/ruby-2)

<div id="comments">
  <h3 id="comments-number" class="comments-header">11 responses to “Ruby exit, exit!, SystemExit and at_exit blunder”</h3>
  <ol class="comment-list">
    <li id="comment-21416" class="comment byuser comment-author-avdi-grimm even thread-even depth-1 comment role-subscriber user-avdi-grimm">
      <img alt="Avdi Grimm" src="https://1.gravatar.com/avatar/fea3202718832fe11a0db6bb44a5cc37?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://plus.google.com/104757475552569715504">Avdi Grimm</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, September 8th, 2012, 9:21 pm">September 8, 2012</abbr> at <abbr class="comment-time" title="Saturday, September 8th, 2012, 9:21 pm">9:21 pm</abbr>
      </div>
      <div class="comment-text">
        <p>A good reminder; not enough people understand Ruby’s process termination system. Incidentally, I prefer `abort(“Some error message”)` to `exit(1)`. It sets the exit status to 1, and also sets the error message in the SystemExit exception.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-21418" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, September 8th, 2012, 9:58 pm">September 8, 2012</abbr> at <abbr class="comment-time" title="Saturday, September 8th, 2012, 9:58 pm">9:58 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Great tip Avdi! I’ve not used abort() before.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
        <li id="comment-21574" class="comment even depth-2 comment reader">
          <img alt="Deryl R. Doucette" src="https://0.gravatar.com/avatar/6dadaf9b9c5aab451a2f8de98752946c?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">Deryl R. Doucette</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, September 12th, 2012, 6:14 am">September 12, 2012</abbr> at <abbr class="comment-time" title="Wednesday, September 12th, 2012, 6:14 am">6:14 am</abbr>
          </div>
          <div class="comment-text">
            <p>Yes, did not know about the abort() either! I’ve been hardcoding ‘exit 255′ in my scripts. Thanks!</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-21430" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Jonathan Rochkind" src="https://0.gravatar.com/avatar/0f50b9a2ad85666d537d39bda49327ee?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://bibwild.wordpress.com">Jonathan Rochkind</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, September 9th, 2012, 6:44 am">September 9, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 9th, 2012, 6:44 am">6:44 am</abbr>
      </div>
      <div class="comment-text">
        <blockquote>
          <p>at_exit blocks are used with great caution and echo the original exit status when necessary.</p>
        </blockquote>
        <p>Any clue on how one would do this, is there any way an “at_exit” block can access the ‘original exit status’?</p>
        <p>Or wait, is the key thing just that you do not need to and should not call `exit` inside `at_exit` — at_exit shoudl do it’s thing without calling `exit`, and then when it’s done being executed the process will still exit on it’s own, with the original exit value. Is that right?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-21436" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, September 9th, 2012, 10:12 am">September 9, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 9th, 2012, 10:12 am">10:12 am</abbr>
          </div>
          <div class="comment-text">
            <p>Good question, Jonathan. Yes, I think putting a “exit” inside an “at_exit” block makes no sense, because the application is already in the process of exiting. Putting an “exit!” instead of an “at_exit” block should also be avoided. If you have multiple at_exit blocks then this would cause some to run and others not to run, because one of the blocks decided to do this hard exit. It is technically possible to reason about the order in which these “at_exit” blocks execute (“If multiple handlers are registered, they are executed in reverse order of registration”), but who is going to keep track of this? It makes for better code to avoid “exit” or “exit!” within “at_exit” blocks altogether, especially a soft exit (“exit”). It is possible that some code within an at_exit block realizes that a situation has occurred where it needs to hard exit (“exit!”), so that nothing else is executed, but I would try to re-engineer the code so that this is not needed.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
        <li id="comment-21437" class="comment byuser comment-author-avdi-grimm odd alt depth-2 comment role-subscriber user-avdi-grimm">
          <img alt="Avdi Grimm" src="https://1.gravatar.com/avatar/fea3202718832fe11a0db6bb44a5cc37?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://plus.google.com/104757475552569715504">Avdi Grimm</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, September 9th, 2012, 10:28 am">September 9, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 9th, 2012, 10:28 am">10:28 am</abbr>
          </div>
          <div class="comment-text">
            <p>You can always find out what the exit status is by looking at the SystemExit exception, which will be found in `$!` (aka `$ERROR_INFO` if you require ‘English’). If I may shamelessly self-promote for a moment, I go into this in depth in Exceptional Ruby.</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-21438" class="comment byuser comment-author-admin bypostauthor even depth-3 comment role-administrator user-admin entry-author">
              <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Sunday, September 9th, 2012, 10:54 am">September 9, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 9th, 2012, 10:54 am">10:54 am</abbr>
              </div>
              <div class="comment-text">
                <p>Great tip Avdi. I love shameless self-promotions! <img src="/media/images/smilies/icon_wink.gif" alt=";)" class="wp-smiley" />
                </p>
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
    <li id="comment-21450" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Rick Hull" src="https://0.gravatar.com/avatar/af79cb74b6f6ca8f1102e415a3b9c9c2?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Rick Hull</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, September 9th, 2012, 5:03 pm">September 9, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 9th, 2012, 5:03 pm">5:03 pm</abbr>
      </div>
      <div class="comment-text">
        <p>&gt; Yes, I think putting a “exit” inside an “at_exit” block makes no sense</p>
        <p>Pardon the phrase, but this is the real WTF.  Maybe at_exit good practices would be a good blog post.  I’ve never had to use it so I’ve never looked into it myself.  Also, I wholeheartedly recommend Avdi’s exceptional book.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-21607" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="David Barri" src="https://0.gravatar.com/avatar/8c6a913c20ee6f2001ce85198b887bac?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://japgolly.blogspot.com.au/">David Barri</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, September 13th, 2012, 6:05 am">September 13, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 13th, 2012, 6:05 am">6:05 am</abbr>
      </div>
      <div class="comment-text">
        <p>What a coincidence! I just blogged about something very similar!<br />
https://japgolly.blogspot.com.au/2012/09/problems-with-atexit-and-exit-and-rspec.html</p>
        <p>Nice.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-21913" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Jonathan Rochkind" src="https://0.gravatar.com/avatar/0f50b9a2ad85666d537d39bda49327ee?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://bibwild.wordpress.com">Jonathan Rochkind</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, September 19th, 2012, 9:08 am">September 19, 2012</abbr> at <abbr class="comment-time" title="Wednesday, September 19th, 2012, 9:08 am">9:08 am</abbr>
      </div>
      <div class="comment-text">
        <p>After running into my own weird at_exit related bug, I discovered this MRI bug report:</p>
        <p>https://bugs.ruby-lang.org/issues/5218</p>
        <p>I think a LOT of people’s at_exit related problems are actually due to this bug. Note that while the bug is marked fixed in the tracker, the reproduction still reproduces for me in 1.9.3p194, so I don’t think it’s made it into a release yet. </p>
        <p>And that bug report reproduction case also reveals something interesting — you ARE allowed to call `exit` in an `at_exit` block.  It does not keep subsequent  `at_exit` blocks from running. If everything is working properly, the exit code of the last `at_exit` block to call `exit` ‘wins’. But because of that bug, everything may not be working properly.  There is a workaround with monkey-patch redefining `at_exit` in that bug report. </p>
        <p>That bug doesn’t actually account for YOUR problem as above. Your problem was a legitimate software error — don’t call ‘exit’ in an `at_exit` block unless you actually WANT to set the exit code in the `at_exit` block. If you don’t, and don’t need to call `exit` in it, you’re fine. If you DO need to call `exit’ in an at_exit block to set the exit code… then the MRI bug may interfere with your desires.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-21914" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, September 19th, 2012, 9:43 am">September 19, 2012</abbr> at <abbr class="comment-time" title="Wednesday, September 19th, 2012, 9:43 am">9:43 am</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Jonathan for this great clarification! I think anyone who reads this post should read your comment. I will make a note of it at the bottom of the post.</p>
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

