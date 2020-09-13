---
title: "Homebrew For Mac | How To Install And Use Homebrew"
date: "2010-12-09T01:51:00-08:00"
template: "post"
draft: false
slug: "homebrew-intro-to-the-mac-os-x-package-installer"
category: "Software Engineering"
tags:
  - brew
  - darwin ports
  - homebrew
  - install
  - mac os x
  - macports
  - mongodb
  - postgresql
  - ruby
  - sysadmin
description: "This is demonstration of installing Homebrew, the new Mac OS X package installer. Step-by-step instructions on installing Homebrew and using the brew command."
socialImage:
  publicURL: "/media/images/2010/12/homebrew_intro_sq.jpg"
---
I thought I should mention the very easy Homebrew package installer for Mac OS X. I’ve found it quite simple to use and have not turned back since I blew away my MacPorts.

You can find [Homebrew on gitub](https://github.com/mxcl/homebrew) and you can find the recommended installation instructions there which are as follows.

```
ruby -e "$(curl -fsSkL raw.github.com/mxcl/homebrew/go)"
```

This will download the “go” ruby script and execute it in one go. If you are unsure or just curious and can read ruby code, then you can see this script [here](https://raw.github.com/mxcl/homebrew/go).

In brief, this script changes permissions on various directories, downloads the Homebrew install files and installs them under /usr/local

<a href="https://www.flickr.com/photos/egansnow/238849691/" title="Untitled by Egan Snow, on Flickr"><img alt="" height="333" src="https://farm1.static.flickr.com/87/238849691_b7ce379bd7.jpg" width="500"/></a>

Before I ran this install, I blew away my MacPorts installation using [these instructions](https://guide.macports.org/chunked/installing.macports.uninstalling.html), which suggests the following…

```
sudo port -f uninstall installed

sudo rm -rf \
    /opt/local \
    /Applications/DarwinPorts \
    /Applications/MacPorts \
    /Library/LaunchDaemons/org.macports.* \
    /Library/Receipts/DarwinPorts*.pkg \
    /Library/Receipts/MacPorts*.pkg \
    /Library/StartupItems/DarwinPortsStartup \
    /Library/Tcl/darwinports1.0 \
    /Library/Tcl/macports1.0 \
    ~/.macports
```

Since Apple does not touch /usr/local I cleaned that out too, since anything in there on my Mac would have been installed by MacPorts. Homebrew installs under /usr/local and I wanted it to be nice and clean. Doing this will depend on what you’ve been up to with your Mac.

Some key files that will be installed by the installer are

```
/usr/local/bin/brew
```

This script does all the magic of installing, uninstalling, listing installed packages, showing package information and other tasks.

```
/usr/local/Cellar/
```

This directory is where the files are installed for the packages you install. You can see some of the package directories I have installed on this machine.

```
$ ls -dF1 /usr/local/Cellar/*
/usr/local/Cellar/bash-completion/
/usr/local/Cellar/geos/
/usr/local/Cellar/git/
/usr/local/Cellar/mongodb/
/usr/local/Cellar/ossp-uuid/
/usr/local/Cellar/pidof/
/usr/local/Cellar/postgis/
/usr/local/Cellar/postgresql/
/usr/local/Cellar/proj/
/usr/local/Cellar/readline/
/usr/local/Cellar/wget/
```

Once, you’re installed you should be able to run _brew install_ to install packages.

Packages are first downloaded to _/Library/Caches/Homebrew/_, so if you do uninstall and re-install, you will not have to re-download them a second time.

Here’s a dump of the files my cache directory.

```
ls -1 /Library/Caches/Homebrew/*
/Library/Caches/Homebrew/bash-completion-1.2.tar.bz2
/Library/Caches/Homebrew/geos-3.2.2.tar.bz2
/Library/Caches/Homebrew/git-1.7.3.2.tar.bz2
/Library/Caches/Homebrew/git-htmldocs-1.7.3.2.tar.bz2
/Library/Caches/Homebrew/git-manpages-1.7.3.2.tar.bz2
/Library/Caches/Homebrew/mongodb-1.6.3-x86_64.tgz
/Library/Caches/Homebrew/mysql-5.1.51.tar.gz
/Library/Caches/Homebrew/ossp-uuid-1.6.2.tar.gz
/Library/Caches/Homebrew/pidof-0.1.4.tar.gz
/Library/Caches/Homebrew/postgis-1.5.2.tar.gz
/Library/Caches/Homebrew/postgresql-9.0.1.tar.bz2
/Library/Caches/Homebrew/proj-4.7.0.tar.gz
/Library/Caches/Homebrew/proj-datumgrid-1.5.zip
/Library/Caches/Homebrew/readline-6.1.tar.gz
/Library/Caches/Homebrew/solr-1.4.1.tgz
/Library/Caches/Homebrew/wget-1.12.tar.bz2
```

I will demonstrate installing [Mongodb](https://www.mongodb.org/), which is a “scalable, high-performance, open source, document-oriented database”. Think “NoSQL”.

```
brew install mongodb
```

Here’s the output

```
==> Downloading https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-1.6.3.tgz
######################################################################## 100.0%
==> Caveats
If this is your first install, automatically load on login with:
    cp /usr/local/Cellar/mongodb/1.6.3-x86_64/org.mongodb.mongod.plist ~/Library/LaunchAgents
    launchctl load -w ~/Library/LaunchAgents/org.mongodb.mongod.plist

If this is an upgrade and you already have the org.mongodb.mongod.plist loaded:
    launchctl unload -w ~/Library/LaunchAgents/org.mongodb.mongod.plist
    cp /usr/local/Cellar/mongodb/1.6.3-x86_64/org.mongodb.mongod.plist ~/Library/LaunchAgents
    launchctl load -w ~/Library/LaunchAgents/org.mongodb.mongod.plist

Or start it manually:
    mongod run --config /usr/local/Cellar/mongodb/1.6.3-x86_64/mongod.conf
==> Summary
/usr/local/Cellar/mongodb/1.6.3-x86_64: 16 files, 83M, built in 2 seconds
```

A command I really like is the _info_ command.

```
brew info mongodb
```

It will give you all the information you need for a particular package for starting, stopping, restarting or other actions pertinent the service you installed. If will be similar to the information output when you first install a package. It’s good to know about this command so that you can always find the maintenance commands.

Here is an example of the info command for the postgresql package.

```
brew info postgresql
```

```
postgresql 9.0.1

https://www.postgresql.org/

Depends on: readline, ossp-uuid
/usr/local/Cellar/postgresql/9.0.1 (1229 files, 25M)

If builds of Postgresl 9 are failing and you have version 8.x installed,
you may need to remove the previous version first. See:

https://github.com/mxcl/homebrew/issues/issue/2510

To build plpython against a specific Python, set PYTHON prior to brewing:
  PYTHON=/usr/local/bin/python  brew install postgresql
See:

https://www.postgresql.org/docs/9.0/static/install-procedure.html

If this is your first install, create a database with:
    initdb /usr/local/var/postgres

If this is your first install, automatically load on login with:
    cp /usr/local/Cellar/postgresql/9.0.1/org.postgresql.postgres.plist ~/Library/LaunchAgents
    launchctl load -w ~/Library/LaunchAgents/org.postgresql.postgres.plist

If this is an upgrade and you already have the org.postgresql.postgres.plist loaded:
    launchctl unload -w ~/Library/LaunchAgents/org.postgresql.postgres.plist
    cp /usr/local/Cellar/postgresql/9.0.1/org.postgresql.postgres.plist ~/Library/LaunchAgents
    launchctl load -w ~/Library/LaunchAgents/org.postgresql.postgres.plist

Or start manually with:
    pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start

And stop with:
    pg_ctl -D /usr/local/var/postgres stop -s -m fast

If you want to install the postgres gem, including ARCHFLAGS is recommended:
    env ARCHFLAGS="-arch x86_64" gem install pg

To install gems without sudo, see the Homebrew wiki.

https://github.com/mxcl/homebrew/commits/master/Library/Formula/postgresql.rb
```

_brew list_ will tell you which packages are installed.

_brew help_ will give you the following

```
Usage: brew [-v|--version] [--prefix [formula]] [--cache [formula]]
            [--cellar [formula]] [--config] [--env] [--repository]
            [-h|--help] COMMAND [formula] ...

Principle Commands:
  install formula ... [--ignore-dependencies] [--HEAD]
  list [--unbrewed|--versions] [formula] ...
  search [/regex/] [substring]
  uninstall formula ...
  update

Other Commands:
  info formula [--github]
  options formula
  deps formula
  uses formula [--installed]
  home formula ...
  cleanup [formula]
  link formula ...
  unlink formula ...
  outdated
  missing
  prune
  doctor

Informational:
  --version
  --config
  --prefix [formula]
  --cache [formula]

Commands useful when contributing:
  create URL
  edit [formula]
  audit [formula]
  log formula
  install formula [-vd|-i]

For more information:
  man brew

To visit the Homebrew homepage type:
  brew home

```

I hope you find this quick intro to Homebrew helpful. Please leave a comment to question.

<div id="comments">
  <h3 id="comments-number" class="comments-header">33 responses to “Homebrew – Intro To The Mac OS X Package Installer”</h3>
  <ol class="comment-list">
    <li id="comment-995" class="comment even thread-even depth-1 comment reader">
      <img alt="Sukima" src="https://0.gravatar.com/avatar/a33233f156a5531707f130c28565de69?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://dev.tritarget.org">Sukima</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, February 7th, 2011, 4:55 pm">February 7, 2011</abbr> at <abbr class="comment-time" title="Monday, February 7th, 2011, 4:55 pm">4:55 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Out of curiosity why do you use sudo for all your commands. The docs specifically explain why that is a bad idea. A simple one time `sudo chown -R $USER /usr/local` fixes that issue. This makes life so much easier. sudo opens up a lot of opportunities to really blow things up. Your putting complete trust that there isn’t a bug or mistake in the hundreds of formula scripts. And entering in a password every time you use the brew command is a hassle.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-996" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, February 7th, 2011, 5:01 pm">February 7, 2011</abbr> at <abbr class="comment-time" title="Monday, February 7th, 2011, 5:01 pm">5:01 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Sukima. You are absolutely right. Force of habit. I’ve edited the text to remove “sudo” for the brew commands.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-1944" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Jasper" src="https://1.gravatar.com/avatar/5dc8d9906bf5ac3e07ee47a99f9c197e?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Jasper</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, April 12th, 2011, 2:55 pm">April 12, 2011</abbr> at <abbr class="comment-time" title="Tuesday, April 12th, 2011, 2:55 pm">2:55 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Stumbled across your post and because of that I read about homebrew… Just wanted to say thanks a lot! I was struggling like hell to get postgis working… and it works flawlessly now <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
        </p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-3400" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="texnic" src="https://1.gravatar.com/avatar/d8ff9e3ef50c2601ee8a1e986486af39?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">texnic</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, June 20th, 2011, 2:42 pm">June 20, 2011</abbr> at <abbr class="comment-time" title="Monday, June 20th, 2011, 2:42 pm">2:42 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Upon doing the ruby command, I’m getting</p>
        <p>curl: (60) SSL certificate problem, verify that the CA cert is OK. Details:<br />
error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed</p>
        <p>Though you have -k in the keys already. Any idea of what I could try? I’ve Mac OS 10.5.8.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-3427" class="comment odd alt depth-2 comment reader">
          <img alt="Ulops" src="https://1.gravatar.com/avatar/77e415210c8268f645b5054d6b4aa07f?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">Ulops</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, June 22nd, 2011, 6:22 am">June 22, 2011</abbr> at <abbr class="comment-time" title="Wednesday, June 22nd, 2011, 6:22 am">6:22 am</abbr>
          </div>
          <div class="comment-text">
            <p>Exactly the same issue. Also using 10.5.8</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-3428" class="comment byuser comment-author-admin bypostauthor even depth-3 comment role-administrator user-admin entry-author">
              <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Wednesday, June 22nd, 2011, 8:07 am">June 22, 2011</abbr> at <abbr class="comment-time" title="Wednesday, June 22nd, 2011, 8:07 am">8:07 am</abbr>
              </div>
              <div class="comment-text">
                <p>I’m on 10.6.7, so hard to debug your exact issue. The latest documentation on the homebrew Github site https://github.com/mxcl/homebrew#readme is</p>
                <p>   /usr/bin/ruby -e “$(curl -fsSL https://raw.github.com/gist/323731)”</p>
                <p>If that does not work, then please submit a report here https://github.com/mxcl/homebrew/issues</p>
                <p>Btw, they recommend reading this before filing a report https://allanmcrae.com/2011/05/how-to-file-a-bug-report/</p>
                <p>Hope that helps.</p>
              </div>
              <!-- .comment-text -->
              <ol class="children">
                <li id="comment-3457" class="comment odd alt depth-4 comment reader">
                  <img alt="texnic" src="https://1.gravatar.com/avatar/d8ff9e3ef50c2601ee8a1e986486af39?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
                  <div class="comment-meta comment-meta-data">
                    <div class="comment-author vcard">
                      <cite class="fn">texnic</cite>
                    </div>
                    <!-- .comment-author .vcard -->
                    <abbr class="comment-date" title="Friday, June 24th, 2011, 1:29 am">June 24, 2011</abbr> at <abbr class="comment-time" title="Friday, June 24th, 2011, 1:29 am">1:29 am</abbr>
                  </div>
                  <div class="comment-text">
                    <p>Thanks. This didn’t work, but the issue was eventually resolved by downloading the homebrew elsewhere: https://blog.boxcryptor.com/how-to-use-boxcryptor-with-encfs-on-mac-os-x#comment</p>
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
    <li id="comment-5412" class="comment even thread-even depth-1 comment reader">
      <img alt="Ramesh" src="https://1.gravatar.com/avatar/d00f996cc98439bd17549736591e8a96?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Ramesh</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, November 10th, 2011, 1:22 am">November 10, 2011</abbr> at <abbr class="comment-time" title="Thursday, November 10th, 2011, 1:22 am">1:22 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hi,</p>
        <p>I have configured the brew in linux, How to make  package for using brew ?</p>
        <p>Ramesh</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-5431" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Ramesh" src="https://1.gravatar.com/avatar/d00f996cc98439bd17549736591e8a96?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Ramesh</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, November 11th, 2011, 2:26 am">November 11, 2011</abbr> at <abbr class="comment-time" title="Friday, November 11th, 2011, 2:26 am">2:26 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hi,</p>
        <p>I have installed brew in my linux (slackware13.1 linux)</p>
        <p>While run the command given below</p>
        <p>brew create https://ftp.acc.umu.se/pub/gnome/sources/libsoup/2.33/libsoup-2.33.90.tar.gz</p>
        <p>and typed the formula</p>
        <p>require ‘formula’</p>
        <p>class Libsoup  Downloading https://ftp.acc.umu.se/pub/gnome/sources/libsoup/2.33/libsoup-2.33.90.tar.gz<br />
######################################################################## 100.0%<br />
Warning: Cannot verify package integrity<br />
The formula did not provide a download checksum<br />
For your reference the MD5 is: f905e685f4e2fa6b414522e06d8db3c8<br />
/usr/bin/mktemp: cannot make temp dir /tmp/homebrew-libsoup-2.33.90-XXXX: Invalid argument<br />
Error: Couldn’t create build sandbox<br />
root@slack64:/tmp/brew/homebrew# </p>
        <p>I am getting above error. Please help me how to fix this error</p>
        <p>Ramesh</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-5493" class="comment even thread-even depth-1 comment reader">
      <img alt="Kyle" src="https://0.gravatar.com/avatar/6c917dc1de71fccaf6d8c3afd0c10445?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Kyle</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, November 14th, 2011, 8:38 am">November 14, 2011</abbr> at <abbr class="comment-time" title="Monday, November 14th, 2011, 8:38 am">8:38 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hello,</p>
        <p>I’ve just started using Homebrew and I love it. The only issue I am having is that it doesnt seem to be installing the man pages for any packages I install. I there something I need to alter somewhere or a flag I need to set? </p>
        <p>Thanks,<br />
Kyle</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-5606" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Ramesh" src="https://0.gravatar.com/avatar/6d6c5e5ca914ad7f3e3a20ee68b423b0?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Ramesh</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, November 18th, 2011, 10:49 pm">November 18, 2011</abbr> at <abbr class="comment-time" title="Friday, November 18th, 2011, 10:49 pm">10:49 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Hi Kyle,</p>
        <p>Thanks for your reply,</p>
        <p>Please let me know, how can you configure in your system in details…</p>
        <p>Ramesh</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-5607" class="comment even thread-even depth-1 comment reader">
      <img alt="JM" src="https://1.gravatar.com/avatar/34c1cb33360ceb70eb37c4fb901f09e5?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">JM</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, November 18th, 2011, 11:54 pm">November 18, 2011</abbr> at <abbr class="comment-time" title="Friday, November 18th, 2011, 11:54 pm">11:54 pm</abbr>
      </div>
      <div class="comment-text">
        <p>What are the implications of changing all those permissions on system directories?   This strikes me as an unwise decision, particularly changing the group id’s</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-12441" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Paul" src="https://0.gravatar.com/avatar/c8dbea4e4fe77349e955a2bd71cc4f34?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Paul</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, April 11th, 2012, 3:46 pm">April 11, 2012</abbr> at <abbr class="comment-time" title="Wednesday, April 11th, 2012, 3:46 pm">3:46 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Looks handy, but unfortunately the gist url for the installer script is now a 404</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-12442" class="comment even thread-even depth-1 comment reader">
      <img alt="Paul" src="https://0.gravatar.com/avatar/c8dbea4e4fe77349e955a2bd71cc4f34?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Paul</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, April 11th, 2012, 3:48 pm">April 11, 2012</abbr> at <abbr class="comment-time" title="Wednesday, April 11th, 2012, 3:48 pm">3:48 pm</abbr>
      </div>
      <div class="comment-text">
        <p>I found out that gist site has been restructured a little</p>
        <p>The ruby installer script can now be installed using…</p>
        <p>ruby -e “$(curl -fsSLk https://raw.github.com/mxcl/homebrew/master/Library/Contributions/install_homebrew.rb)“</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-15917" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Josh Kosmala" src="https://1.gravatar.com/avatar/f8a3a3fb6eb5b06649ba4b1c97f21c2b?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.novaweb.co.nz">Josh Kosmala</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, May 27th, 2012, 5:04 pm">May 27, 2012</abbr> at <abbr class="comment-time" title="Sunday, May 27th, 2012, 5:04 pm">5:04 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Hey Dude,</p>
        <p>Nice article. Here is the latest Ruby link if you want to update the link in your article.</p>
        <p>/usr/bin/ruby -e “$(/usr/bin/curl -fsSL https://raw.github.com/mxcl/homebrew/master/Library/Contributions/install_homebrew.rb)“</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-15918" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, May 27th, 2012, 5:24 pm">May 27, 2012</abbr> at <abbr class="comment-time" title="Sunday, May 27th, 2012, 5:24 pm">5:24 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Josh! I’ve updated the post.</p>
            <p>Sorry Paul (comment above), I just noticed you pointed out the same thing last month!</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-23065" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Nuno Pimenta" src="https://1.gravatar.com/avatar/5f4059e4fcdd7d8e580bcbbdcd8d5f29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Nuno Pimenta</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, October 15th, 2012, 4:20 am">October 15, 2012</abbr> at <abbr class="comment-time" title="Monday, October 15th, 2012, 4:20 am">4:20 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hello,</p>
        <p>I’m trying to install package freerdp with homebrew;</p>
        <p>I did the following steps:</p>
        <p>1- Download Xcode 4.3.2 with command line tools<br />
2- Download homebrew;<br />
3- Execute command brew doctor to verify is everything is OK (give Warning: Unbrewed dylibs were found in /usr/local/lib, If you didn’t put them there on purpose they could cause problems when<br />
building Homebrew formulae, and may need to be deleted.)<br />
4- Download freerdp, brew install freerdp;<br />
5- Execute sudo brew link freerdp (Error: Cowardly refusing to `sudo brew link’<br />
You can use brew with sudo, but only if the brew executable is owned by root.)<br />
6- Execute brew link freerdp and its fine;</p>
        <p>How can i execute now freerdp and configure it?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-23075" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, October 15th, 2012, 10:13 am">October 15, 2012</abbr> at <abbr class="comment-time" title="Monday, October 15th, 2012, 10:13 am">10:13 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Nuno,</p>
            <p>I have not installed freerdp myself, but have you looked at running “brew info freerdp”. I find the “brew info” command usually gives you all the information you need to get running with the installed software.</p>
            <p>Thanks,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-23137" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Nuno Pimenta" src="https://1.gravatar.com/avatar/5f4059e4fcdd7d8e580bcbbdcd8d5f29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Nuno Pimenta</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, October 16th, 2012, 1:30 am">October 16, 2012</abbr> at <abbr class="comment-time" title="Tuesday, October 16th, 2012, 1:30 am">1:30 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hello Phil,</p>
        <p>Thanks for the replay…</p>
        <p>Yesterday i run the command brew info freerdp, but didn’t return value information:<br />
gab-03:~ macbook$ brew info freerdp<br />
freerdp: stable 1.0.1, HEAD<br />
https://www.freerdp.com/<br />
Depends on: cmake, pkg-config<br />
/usr/local/Cellar/freerdp/1.0.1 (131 files, 2,1M) *<br />
https://github.com/mxcl/homebrew/commits/master/Library/Formula/freerdp.rb<br />
gab-03:~ macbook$</p>
        <p>Then, in the terminal.app, i navigate to the local were the packages are instaled, /usr/local/Cellar/freerdp/1.0.1/bin and i execute ./xfreerdp to see the resultes:<br />
gab-03:bin macbook$ ./xfreerdp<br />
missing server name<br />
failed to parse arguments.<br />
gab-03:bin macbook$</p>
        <p>That’s fine, because i didn’t configure anything. My point is how to configure now freerdp to make a connection to the server?</p>
        <p>I have another question, i execute brew doctor, and i have the results:<br />
Warning: Unbrewed dylibs were found in /usr/local/lib.<br />
If you didn’t put them there on purpose they could cause problems when<br />
building Homebrew formulae, and may need to be deleted</p>
        <p>Unexpected dylibs:</p>
        <p>    /usr/local/lib/libgps.dylib    /usr/local/lib/libpteiddlg.1.21.0.dylib    /usr/local/lib/libpteidhttps.1.21.0.dylib    /usr/local/lib/libpteidlib.1.21.0.dylib    /usr/local/lib/libpteidlibopensc.1.21.0.dylib    /usr/local/lib/pteidpkcs11.1.21.0.dylib</p>
        <p>I didn’t put anything there, i must remove the files????</p>
        <p>Thank you very much for the replay.<br />
Greatings from Portugal.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-23207" class="comment even thread-even depth-1 comment reader">
      <img alt="Nuno Pimenta" src="https://1.gravatar.com/avatar/5f4059e4fcdd7d8e580bcbbdcd8d5f29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Nuno Pimenta</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, October 17th, 2012, 3:25 am">October 17, 2012</abbr> at <abbr class="comment-time" title="Wednesday, October 17th, 2012, 3:25 am">3:25 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hello everyone,</p>
        <p>I’m using freerdp on my Mac Mini, installed with homebrew.</p>
        <p>My system is OK, i can access to my Windows Server 2003 R2 with no problem, keyboard and mouse are working good.</p>
        <p>I use digital sign to all operations, but, the software for doing it can’t be installed in MAC. With the software, i use a smartcard reader, SCR3310, to reader my card.</p>
        <p>I need that freerdp permits using smartcard on my RDP session for digital sign, not for autentication. I can do this in a Windows XP because de RDP software permits using smart cards in the RDP session and my Windows Server 2003 R2 acept this.</p>
        <p>I did a little web resource and i am using the following command xfreerdp -u  -g  –plugin rdpdr –data scard:scard —  and i can acess to my server but the server don’t recognizes the smart card reader.</p>
        <p>In the terminal.app, i get the following message:<br />
gab-03:bin macbook$ xfreerdp -u  -g 1024*768 –plugin rdpdr –data scard:scard —<br />
loading plugin rdpdr<br />
freerdp_load_library_symbol: failed to open /usr/local/Cellar/freerdp/1.0.1/lib/freerdp/rdpdr: dlopen(/usr/local/Cellar/freerdp/1.0.1/lib/freerdp/rdpdr, 5): image not found<br />
freerdp_load_plugin: failed to load rdpdr/VirtualChannelEntry</p>
        <p>Can anyone point me any solution?</p>
        <p>Best regards</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-23222" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, October 17th, 2012, 8:45 am">October 17, 2012</abbr> at <abbr class="comment-time" title="Wednesday, October 17th, 2012, 8:45 am">8:45 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Nuno,</p>
            <p>Sorry to hear that you are still having problems.</p>
            <p>I think this outside the scope of this blog post, which is specifically for getting up and running with Homebrew.</p>
            <p>There is a mailing-list for freerdp, which you can find here.<br />
https://lists.sourceforge.net/lists/listinfo/freerdp-devel<br />
I think if you send your questions here, you will find the answers you are looking for.</p>
            <p>Thanks,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-23404" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="franc" src="https://1.gravatar.com/avatar/932f872b04b352a8f3d81288805f08be?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">franc</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, October 19th, 2012, 12:40 pm">October 19, 2012</abbr> at <abbr class="comment-time" title="Friday, October 19th, 2012, 12:40 pm">12:40 pm</abbr>
      </div>
      <div class="comment-text">
        <p>All links to install homebrew are 404.<br />
Is this site dead?</p>
        <p>frank</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-23885" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Stephen McConnell" src="https://0.gravatar.com/avatar/cc65fe45220b61a2efea28ffdf0b3c77?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Stephen McConnell</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, October 25th, 2012, 12:24 pm">October 25, 2012</abbr> at <abbr class="comment-time" title="Thursday, October 25th, 2012, 12:24 pm">12:24 pm</abbr>
      </div>
      <div class="comment-text">
        <p>ruby -e “$(curl -fsSL https://raw.github.com/mxcl/homebrew/master/Library/Contributions/install_homebrew.rb)”<br />
curl: (22) The requested URL returned error: 404</p>
        <p>The URL in the example returns a 404.   I cut and pasted it.  Didn’t work.  But the rest of the tutorial was excellent.</p>
        <p>Thanks</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-24377" class="comment even depth-2 comment reader">
          <img alt="franc" src="https://1.gravatar.com/avatar/932f872b04b352a8f3d81288805f08be?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">franc</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Tuesday, October 30th, 2012, 2:43 pm">October 30, 2012</abbr> at <abbr class="comment-time" title="Tuesday, October 30th, 2012, 2:43 pm">2:43 pm</abbr></div>
          <div class="comment-text">
            <p>On the website:</p>
            <p>https://mxcl.github.com/homebrew/</p>
            <p>the command:</p>
            <p>ruby -e “$(curl -fsSkL raw.github.com/mxcl/homebrew/go)”</p>
            <p>is working. There are more informations on:</p>
            <p>https://github.com/mxcl/homebrew/wiki/Installation</p>
            <p>it worked for me just some days ago, I could isntall FreeRDP (xfreerdp) with homebrew.</p>
            <p>frank</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-24378" class="comment byuser comment-author-admin bypostauthor odd alt depth-3 comment role-administrator user-admin entry-author">
              <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Tuesday, October 30th, 2012, 2:49 pm">October 30, 2012</abbr> at <abbr class="comment-time" title="Tuesday, October 30th, 2012, 2:49 pm">2:49 pm</abbr></div>
              <div class="comment-text">
                <p>Thanks Franc! I have updated the post above.</p>
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
    <li id="comment-24894" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Ann" src="https://0.gravatar.com/avatar/c3f086361a77da1e20dd1cd4059ddb8b?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Ann</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, November 4th, 2012, 9:49 pm">November 4, 2012</abbr> at <abbr class="comment-time" title="Sunday, November 4th, 2012, 9:49 pm">9:49 pm</abbr></div>
      <div class="comment-text">
        <p>When I pasted it at the terminal prompt, I got:</p>
        <p>Failed during: Error: /usr/bin/xcode-select returned unexpected error. init -q</p>
        <p>Could you please help?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-24895" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, November 4th, 2012, 9:57 pm">November 4, 2012</abbr> at <abbr class="comment-time" title="Sunday, November 4th, 2012, 9:57 pm">9:57 pm</abbr></div>
          <div class="comment-text">
            <p>Do you mean you pasted this into the terminal?<br />
ruby -e “$(curl -fsSkL raw.github.com/mxcl/homebrew/go)”</p>
            <p>Is this the full error message? Maybe this is related… https://github.com/mxcl/homebrew/issues/10378</p>
            <p>Do you have the latest Xcode installed?</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-32793" class="comment byuser comment-author-mark-scarton even thread-even depth-1 comment role-subscriber user-mark-scarton">
      <img alt="Mark Scarton" src="https://0.gravatar.com/avatar/8fc9fbd0ae3b7b915118101d5ccec142?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Mark Scarton</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, January 21st, 2013, 12:01 pm">January 21, 2013</abbr> at <abbr class="comment-time" title="Monday, January 21st, 2013, 12:01 pm">12:01 pm</abbr></div>
      <div class="comment-text">
        <p>Hi, I’m new to MacOS (long time Unix and Linux user) and am running the latest update of Mountain Lion. I’d really like to use homebrew to gain access to my GNU tools. But when I attempt to run the script that you provided, I am receiving the following error:</p>
        <p>ruby -e “$(curl -fsSkL raw.github.com/mxcl/homebrew/go)”<br />
-e:1: Invalid char `\342′ in expression<br />
-e:1: Invalid char `\200′ in expression<br />
-e:1: Invalid char `\234′ in expression</p>
        <p>Do you have any suggestions for what I might be doing wrong?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-33156" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, January 23rd, 2013, 9:43 am">January 23, 2013</abbr> at <abbr class="comment-time" title="Wednesday, January 23rd, 2013, 9:43 am">9:43 am</abbr></div>
          <div class="comment-text">
            <p>Hi Mark,</p>
            <p>That’s your shell complaining about the characters that you are pasting into it. I think the double quotes and some other character is being cut and pasted from this blog post incorrectly. Try typing it in and that should work.</p>
            <p>Thanks,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-38618" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Ranjna" src="https://1.gravatar.com/avatar/3002ad023745ac9836c751ee1cc3b6db?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Ranjna</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, March 21st, 2013, 10:34 am">March 21, 2013</abbr> at <abbr class="comment-time" title="Thursday, March 21st, 2013, 10:34 am">10:34 am</abbr></div>
      <div class="comment-text">
        <p>I am using mountain lion and was able to install homebrew, however, when i try installing any software using brew, I get Curl (7):failed to connect to 0.9.0.4: No route to host.</p>
        <p>I checked my firewall and it is not on. I even tried changing the url to use that really works in the rb file, but still get the same error. Any idea?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-38981" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, March 23rd, 2013, 10:56 pm">March 23, 2013</abbr> at <abbr class="comment-time" title="Saturday, March 23rd, 2013, 10:56 pm">10:56 pm</abbr></div>
          <div class="comment-text">
            <p>That sounds like a DNS issue you’re having. 0.9.0.4 is a bogus IP address and it’s unlikely that Homebrew has any hard-coded IP addresses in it’s code (at least I hope not).</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-41233" class="comment byuser comment-author-mickeyj4j even thread-even depth-1 comment role-subscriber user-mickeyj4j">
      <img alt="Michael Stoneham" src="https://1.gravatar.com/avatar/dc96cb3441678d051af87321c8dc3e6d?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.google.com/profiles/109657025376754851402">Michael Stoneham</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, April 7th, 2013, 5:45 am">April 7, 2013</abbr> at <abbr class="comment-time" title="Sunday, April 7th, 2013, 5:45 am">5:45 am</abbr></div>
      <div class="comment-text">
        <p>I am trying to find a list of what the apps are in Homebrew. and not just a list but an explanation of what the apps are and do. </p>
        <p>also i want to let you know that i found a gui for Homebrew https://github.com/vincentsaluzzo/Homebrew-GUI/downloads which is easy to install and use. this gui would rock if it had descriptions for the apps built in.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

