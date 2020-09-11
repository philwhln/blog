---
title: "Gitolite Installation Step-By-Step"
date: "2012-07-23T22:04:00-07:00"
template: "post"
draft: false
slug: "gitolite-installation-step-by-step"
category: "Infrastrucutre"
tags:
  - git
  - github
  - gitolite
  - ssh-keygen
description: "In this post I will walk you through how to setup your own gitolite server and start hosting your own git repositories. Gitolite is a great tool for doing this and works in a very git-like way for managing users, access-rights and setting up new repositories for collaboration between your team. A common reason to setup Gitolite is to remove reliance on services like GitHub."
socialImage:
  publicURL: "/media/images/photo.jpg"
---
In this post I will walk you through how to setup your own gitolite server and start hosting your own git repositories.

Gitolite is a great tool for doing this and works in a very git-like way for managing users, access-rights and setting up new repositories for collaboration between your team. A common reason to setup Gitolite is to remove reliance on services like GitHub.

Let’s begin…

## We need hardware!

First, I fired a new machine on [Linode](https://goo.gl/W7Ocv). I’m not going to go into the details of that, but it was a fresh install of “Ubuntu 12.04 LTS 64bit” with root user setup and ssh access to the root user via password. This is simply how [Linode](https://goo.gl/W7Ocv) does it. I know that many people will try this on Amazon Web Services, so I’ve prefixed all appropriate root commands with “sudo” and you will only have replace “root” with “ubuntu” (or whatever the default username is on the AMI you fire-up).

## Mr. Gitolite

First, we are going to login to our remote machine, which will be hosting gitolite on, and we are going to create the “gitolite” user.

```
phil@air:~
$ ssh root@66.175.220.39
The authenticity of host '66.175.220.39 (66.175.220.39)' can't be established.
RSA key fingerprint is 0c:9a:e3:6d:03:21:ce:1b:8c:bb:1d:20:69:cd:98:75.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '66.175.220.39' (RSA) to the list of known hosts.
root@66.175.220.39's password:
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.4.2-x86_64-linode25 x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Jul 24 02:57:20 UTC 2012

  System load:  0.07              Processes:           77
  Usage of /:   2.4% of 19.48GB   Users logged in:     0
  Memory usage: 7%                IP address for eth0: 66.175.220.39
  Swap usage:   0%

  Graph this data and manage this system at https://landscape.canonical.com/

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@localhost:~# sudo adduser \
   --system \
   --shell /bin/bash \
   --gecos 'git version control' \
   --group \
   --disabled-password \
   --home /home/gitolite gitolite
Adding system user `gitolite' (UID 107) ...
Adding new group `gitolite' (GID 114) ...
Adding new user `gitolite' (UID 107) with group `gitolite' ...
Creating home directory `/home/gitolite' ...
```

Now we can return to our local machine.

```
root@localhost:~# exit
logout
Connection to 66.175.220.39 closed.
```

## Anyone seen my keys?

In order to provide easy access to the gitolite user on the remote machine, we need to create a key-pair. This is the public and private keys used for ssh.

First, we will create the key pair on our local machine and we will call this key-pair “gitolite”.

```
phil@air:~
$ cd ~/.ssh
phil@air:~/.ssh
$ ssh-keygen -t rsa -f gitolite
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in gitolite.
Your public key has been saved in gitolite.pub.
The key fingerprint is:
ca:31:15:88:c1:0a:c7:42:09:f7:b5:dc:22:d2:52:62 phil@air.local
The key's randomart image is:
+--[ RSA 2048]----+
|+oE.oo...        |
|o+o=oo.o .       |
| +o.+ + o        |
|  .o . o         |
|      o S        |
|     . +         |
|      o          |
|                 |
|                 |
+-----------------+
```

We can see that the 2 keys have been created.

```
phil@air:~/.ssh
$ ls -l ~/.ssh/gitolite*
-rw-------  1 phil  staff  1679 23 Jul 19:58 /Users/phil/.ssh/gitolite
-rw-r--r--  1 phil  staff   396 23 Jul 19:58 /Users/phil/.ssh/gitolite.pub
```

“gitolite” is the private key and “gitolite.pub” is the public key.

Upload the public key to the root user’s account on the remote machine.

```
phil@air:~/.ssh
$ scp ~/.ssh/gitolite.pub root@66.175.220.39:
root@66.175.220.39's password:
gitolite.pub                                                                                                                 100%  396     0.4KB/s   00:00
```

Login to the root users account on the remote machine…

```
phil@air:~/.ssh
$ ssh root@66.175.220.39
root@66.175.220.39's password:
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.4.2-x86_64-linode25 x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Jul 24 03:00:05 UTC 2012

  System load:  0.02              Processes:           75
  Usage of /:   2.4% of 19.48GB   Users logged in:     0
  Memory usage: 7%                IP address for eth0: 66.175.220.39
  Swap usage:   0%

  Graph this data and manage this system at https://landscape.canonical.com/

Last login: Tue Jul 24 02:57:20 2012 from s12079106cffd1777.vc.shawcable.net
```

…and move the key to the gitolite user home directory, being sure to also change the permissions.

```
root@localhost:~# sudo mv gitolite.pub /home/gitolite
root@localhost:~# sudo chown gitolite:gitolite /home/gitolite/gitolite.pub
```

## Let’s git it on

As I mentioned at top of this post, the remote machine is a fresh machine, so we need to install the “git” command. btw, this command is specific to Ubuntu.

```
root@localhost:~# sudo apt-get install git
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  git-man liberror-perl
Suggested packages:
  git-daemon-run git-daemon-sysvinit git-doc git-el git-arch git-cvs git-svn git-email git-gui gitk gitweb
The following NEW packages will be installed:
  git git-man liberror-perl
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 6,741 kB of archives.
After this operation, 15.2 MB of additional disk space will be used.
Do you want to continue [Y/n]?
Get:1 https://us.archive.ubuntu.com/ubuntu/ precise/main liberror-perl all 0.17-1 [23.8 kB]
Get:2 https://us.archive.ubuntu.com/ubuntu/ precise/main git-man all 1:1.7.9.5-1 [630 kB]
Get:3 https://us.archive.ubuntu.com/ubuntu/ precise/main git amd64 1:1.7.9.5-1 [6,087 kB]
Fetched 6,741 kB in 3s (1,846 kB/s)
Selecting previously unselected package liberror-perl.
(Reading database ... 21183 files and directories currently installed.)
Unpacking liberror-perl (from .../liberror-perl_0.17-1_all.deb) ...
Selecting previously unselected package git-man.
Unpacking git-man (from .../git-man_1%3a1.7.9.5-1_all.deb) ...
Selecting previously unselected package git.
Unpacking git (from .../git_1%3a1.7.9.5-1_amd64.deb) ...
Processing triggers for man-db ...
Setting up liberror-perl (0.17-1) ...
Setting up git-man (1:1.7.9.5-1) ...
Setting up git (1:1.7.9.5-1) ...
```

Now that “git” is installed, let’s login as the gitolite user, via the root account….

```
root@localhost:~# sudo su - gitolite
```

…and download the “gitolite” installation files.

```
gitolite@localhost:~$ git clone git://github.com/sitaramc/gitolite
Cloning into 'gitolite'...
remote: Counting objects: 7472, done.
remote: Compressing objects: 100% (2464/2464), done.
remote: Total 7472 (delta 5120), reused 7200 (delta 4883)
Receiving objects: 100% (7472/7472), 1.76 MiB | 1.23 MiB/s, done.
Resolving deltas: 100% (5120/5120), done.
```

We will install it in the gitolite user’s home directory under “~/bin”, so we first create this directory…

```
gitolite@localhost:~$ mkdir bin
```

Now run the install…

```
gitolite@localhost:~$ gitolite/install -to /home/gitolite/bin
```

We’re not there yet. This “install” actually only installed the command-line tool that we will use to setup our gitolite server.

So, let’s run that command-line tool to setup our gitolite server, giving it the public key that we uploaded, called “gitolite.pub”.

```
gitolite@localhost:~$ /home/gitolite/bin/gitolite setup -pk gitolite.pub
Initialized empty Git repository in /home/gitolite/repositories/gitolite-admin.git/
Initialized empty Git repository in /home/gitolite/repositories/testing.git/
WARNING: /home/gitolite/.ssh missing; creating a new one
WARNING: /home/gitolite/.ssh/authorized_keys missing; creating a new one
```

Logout of the gitolite user….

```
gitolite@localhost:~$ exit
logout
```

…and exit the machine.

```
root@localhost:~# exit
logout
Connection to 66.175.220.39 closed.
```

## Reduce carpal tunnel by 20%!

Just to make it easy to we are going to give our machine an ssh alias, by adding some config under our ~/.ssh directory.

Edit (or create) ~/.ssh/config

```
phil@air:~
$ vim ~/.ssh/config
```

and add the following lines…

```
Host gitbox
   User gitolite
   Hostname 66.175.220.39
   Port 22
   IdentityFile ~/.ssh/gitolite
```

The IP is specific to my install, so you will want to change that.

## Take control

With gitolite, the way we manage repositories, users and access-rights is via a git repository. I love this fact. Git controlling git.

Let’s download the gitolite-admin from our newly created git repository.

```
phil@air:~
$ git clone gitbox:gitolite-admin
Cloning into gitolite-admin...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (6/6), done.
```

## What do we have here?

Let’s take a look in the repository…

```
phil@air:~
$ cd gitolite-admin/
```

```
phil@air:~/gitolite-admin (master)
$ ls -l
total 0
drwxr-xr-x  3 phil  staff  102 23 Jul 20:22 keydir
drwxr-xr-x  3 phil  staff  102 23 Jul 20:22 conf
```

## Everyone gets a key

“keydir” is where store our user’s keys. Currently, you will see a gitolite.pub file in there. That is the “gitolite” user we use to administrate our gitolite installation.

Let’s create a new gitolite user called “phil” by copying phil’s public key into the “keydir” directory. You can create the key-pair for this user, just as you did for the gitolite user at the top of this post.

```
phil@air:~/gitolite-admin (master)
$ cp ~/.ssh/phil_rsa.pub keydir/phil.pub
phil@air:~/gitolite-admin (master)
$ git add keydir/phil.pub
phil@air:~/gitolite-admin (master)
$ git commit -m 'added user "phil" to gitolite'
[master d595439] added user "phil" to gitolite
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 keydir/phil.pub
phil@air:~/gitolite-admin (master)
```

Pushing to our remote machine will create that user.

```
$ git push
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 714 bytes, done.
Total 4 (delta 0), reused 0 (delta 0)
To gitbox:gitolite-admin
   7f266e0..d595439  master -> master
```

## Privileges

Under the “conf” directory you will find “conf/gitolite.conf” which controls which repositories are available on the system and who has which rights to those repositories.

Let’s edit that file with vim…

```
phil@air:~/gitolite-admin (master)
$ vim conf/gitolite.conf
```

…and add the groups “@admin” and “@staff” and a new repository called “bigfastblog”.

```
 @admin      = phil
 @staff      = phil

 repo gitolite-admin
     RW+     = gitolite @admin

 repo testing
     RW+     = @all

 repo bigfastblog
     RW+     = phil @admin
     R       = @staff
```

As you can see, anyone in our @staff group has read-only privileges to the new “bigfastblog” git repository, but only “phil” and users we add to the @admin group will be able to git-push to that repository.

```
phil@air:~/gitolite-admin (master)
$ git commit conf/gitolite.conf -m 'added "bigfastblog" repo'
[master 357bbc8] added "bigfastblog" repo
 1 files changed, 9 insertions(+), 1 deletions(-)
```

When we “git push” our changes to this configuration back to our gitolite server, we can actually see in the dialog that the “bigfastblog” git repository is being created.

```
phil@air:~/gitolite-admin (master)
$ git push
Counting objects: 7, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 428 bytes, done.
Total 4 (delta 0), reused 0 (delta 0)
remote: Initialized empty Git repository in /home/gitolite/repositories/bigfastblog.git/
To gitbox:gitolite-admin
   d595439..357bbc8  master -> master
```

I hope that gives you a good overview of how to install gitolite. If you found this helpful, please subscribe to this blog or follow me on twitter [@philwhln](https://www.twitter.com/philwhln).

<div id="comments">
  <h3 id="comments-number" class="comments-header">47 responses to “Gitolite Installation Step-By-Step”</h3>
  <ol class="comment-list">
    <li id="comment-19584" class="comment even thread-even depth-1 comment reader">
      <img alt="Dakiss" src="https://0.gravatar.com/avatar/236c8c94d65381b3b8912d689fa7ede4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Dakiss</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, July 25th, 2012, 4:47 am">July 25, 2012</abbr> at <abbr class="comment-time" title="Wednesday, July 25th, 2012, 4:47 am">4:47 am</abbr>
      </div>
      <div class="comment-text">
        <p>Good tutorial, clear, effective. Thank you!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-19710" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Roger Ivy" src="https://1.gravatar.com/avatar/df7b8a8bc19dd3bc020708a8fd59e2be?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Roger Ivy</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, July 28th, 2012, 2:29 am">July 28, 2012</abbr> at <abbr class="comment-time" title="Saturday, July 28th, 2012, 2:29 am">2:29 am</abbr>
      </div>
      <div class="comment-text">
        <p>Thanks for this Phil … I’m testing it end-2-end.</p>
        <p>In the initial hardware section, where you wrote “fresh install of “Ubuntu 12.04 LTS 64bit” with root user setup and ssh access to the root user via password”</p>
        <p>I was under the impression that Ubuntu only gives you access to root via “sudo”, that you can’t actually be “root”. Did you set up your initial user as “root”?</p>
        <p>…or did you do something like this:<br />
 sudo passwd<br />
 Password:<br />
 Enter new UNIX password:<br />
 Retype new UNIX password:</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-19727" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, July 28th, 2012, 3:24 pm">July 28, 2012</abbr> at <abbr class="comment-time" title="Saturday, July 28th, 2012, 3:24 pm">3:24 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Roger,</p>
            <p>That “root” user is a Linode thing. When you fire-up a new machine you can set the root password before you boot the machine. So, yes, you login via the “root” user. I did not actually do anything myself to get into this position. I know AWS AMI images for Ubuntu generally do not allow you to login via as root, so I hope this does not cause too much confusion.</p>
            <p>For AWS, simply replace “root” for “ubuntu” (or the username you have on the machine) and when you run commands as that user (and only that user), ensure that you have them prefixed with “sudo”.</p>
            <p>Hope that helps.</p>
            <p>[AWS = Amazon Web Services]</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
        <li id="comment-19728" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, July 28th, 2012, 3:32 pm">July 28, 2012</abbr> at <abbr class="comment-time" title="Saturday, July 28th, 2012, 3:32 pm">3:32 pm</abbr>
          </div>
          <div class="comment-text">
            <p>I’ve updated the post, so you will only have replace “root” with “ubuntu” (or the username you have on the machine) and everything should work. Let me know if you hit any issues.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-20441" class="comment even thread-even depth-1 comment reader">
      <img alt="Warren Le" src="https://1.gravatar.com/avatar/7efa7217ae2100015f2b37d6d5d63d37?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Warren Le</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, August 7th, 2012, 5:41 am">August 7, 2012</abbr> at <abbr class="comment-time" title="Tuesday, August 7th, 2012, 5:41 am">5:41 am</abbr>
      </div>
      <div class="comment-text">
        <p>“You used a key that is already set to give you shell access. You cannot use the same key to get shell access as well as access gitolite repos.” (https://sitaramc.github.com/gitolite/emergencies.html#ce)</p>
        <p>In our case is ‘sudo mv gitolite.pub /home/gitolite’<br />
and ‘/home/gitolite/bin/gitolite setup -pk gitolite.pub’</p>
        <p>Which means we used gitolite.pub to get shell access and access to gitolite repos as well.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-20596" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Sitaram Chamarty" src="https://1.gravatar.com/avatar/f731ab5313beb6fd680c8a9a51612925?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Sitaram Chamarty</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, August 12th, 2012, 6:54 pm">August 12, 2012</abbr> at <abbr class="comment-time" title="Sunday, August 12th, 2012, 6:54 pm">6:54 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Phil,</p>
        <p>That’s a very nice and clearly written blog post.</p>
        <p>But the reason I am writing is this: I can’t tell you how much I appreciate the deprecation notice at the top of your older (2010 or so) blog post on this.</p>
        <p>In contrast, I had to badger the owner of a then-popular (but already long obsolete) blogpost on gitosis for a few months before he finally relented and put in a very weak disclaimer.  Luckily it’s no longer among the top hits and we no longer get as many people on #git asking for help because they followed that blog and something went wrong.</p>
        <p>So… thanks again for making my life easier.</p>
        <p>regards,</p>
        <p>sitaram</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-20910" class="comment even thread-even depth-1 comment reader">
      <img alt="Joey" src="https://0.gravatar.com/avatar/c229d3ea637f7829b29615f2ea11f586?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://emwork.wordpress.com/">Joey</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, August 23rd, 2012, 6:31 pm">August 23, 2012</abbr> at <abbr class="comment-time" title="Thursday, August 23rd, 2012, 6:31 pm">6:31 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Nice and Superb Gitolite tutorial!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-21344" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="jakkudanieru" src="https://0.gravatar.com/avatar/4e6326e613172caff2d98f9c530d6bdc?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">jakkudanieru</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, September 6th, 2012, 8:32 pm">September 6, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 6th, 2012, 8:32 pm">8:32 pm</abbr>
      </div>
      <div class="comment-text">
        <p>No ssh-add ~/.ssh/gitolite.pub ?<br />
Kind of stuff that save people time isn’t it?</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-22187" class="comment even thread-even depth-1 comment reader">
      <img alt="bfn" src="https://0.gravatar.com/avatar/6b5fa39f5bc4b397eb7e55766eecd8c4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">bfn</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, September 25th, 2012, 2:39 pm">September 25, 2012</abbr> at <abbr class="comment-time" title="Tuesday, September 25th, 2012, 2:39 pm">2:39 pm</abbr>
      </div>
      <div class="comment-text">
        <p>I’ve followed the steps, but ssh asks me for a password:</p>
        <p>$ git clone myserver:gitolite-admin<br />
Cloning into ‘gitolite-admin’…<br />
gitolite@myadress.com‘s password:</p>
        <p>I never set a password, and if I press “Enter”, it says “Permission denied, please try again”. Any idea? The server is on Debian Squeeze, and the local machine on Ubuntu 12.04.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-22283" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, September 27th, 2012, 9:43 am">September 27, 2012</abbr> at <abbr class="comment-time" title="Thursday, September 27th, 2012, 9:43 am">9:43 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Bfn,</p>
            <p>If you are seeing the password prompt, then this indicates that the ssh keys is not setup correctly, so it is falling back to password authentication. Maybe you missed a step?</p>
            <p>Thanks,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-22434" class="comment even depth-3 comment reader">
              <img alt="bfn" src="https://0.gravatar.com/avatar/6b5fa39f5bc4b397eb7e55766eecd8c4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn">bfn</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Sunday, September 30th, 2012, 6:32 am">September 30, 2012</abbr> at <abbr class="comment-time" title="Sunday, September 30th, 2012, 6:32 am">6:32 am</abbr>
              </div>
              <div class="comment-text">
                <p>This was a problem in the /etc/ssh/sshd_config file, I added “gitolite” user after the AllowUsers directive, and now this is ok <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
                </p>
                <p>(cf: https://sitaramc.github.com/gitolite/sts.html#stsapp1 )</p>
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
    <li id="comment-22410" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Geoff Ford" src="https://0.gravatar.com/avatar/a1534fcf8cbea6b06f93f9b4cb57c1b9?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Geoff Ford</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, September 29th, 2012, 5:47 pm">September 29, 2012</abbr> at <abbr class="comment-time" title="Saturday, September 29th, 2012, 5:47 pm">5:47 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Thank you very much for this awesome tutorial. The only one I have found that takes you right through the process and actually works at the end!!</p>
        <p>I would just like to add an additional step for anyone wanting to add new users on other workstations as it took me a while to get this bit working correctly (i.e. not asking for a password).</p>
        <p>*Firstly create your keys on the new users machine.<br />
*Then copy the public key to the keydir folder of the gitolite-admin repo on whichever machine you set up the above steps to administer gitolite from and commit your changes as described above.<br />
*On the same admin machine edit the conf/gitolite.conf to add your user wherever required and commit your changes again.</p>
        <p>Now it was at this point I ran into problems when trying to clone a repo on to my new user’s machine. I kept getting a prompt for a password for my new user. The solution was to create the following .ssh/config file on the new user’s machine:</p>
        <p>Host gitbox<br />
    Hostname your_hostname_here<br />
    User gitolite<br />
    Port 22<br />
    IdentityFile ~/.ssh/new_user_key</p>
        <p>Note that whilst you still use the new user’s private key in IdentityFile, the magic comes from entering gitolite as the User as it is this username that is used to make the ssh connection that then passes the new_user_key for authentication. This works because when new users are added to gitolite-admin, the public keys are added as additional keys to the gitolite user on the server in .ssh/authorized_keys.</p>
        <p>You can now clone a remote repo to the new user’s machine with the following:</p>
        <p>git clone gitbox:your_repo_name</p>
        <p>Hope that saves a bit of someone’s time <img src="/media/images/smilies/icon_smile.gif" alt=":-)" class="wp-smiley" />
        </p>
        <p>Thanks</p>
        <p>Geoff.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-22413" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, September 29th, 2012, 8:52 pm">September 29, 2012</abbr> at <abbr class="comment-time" title="Saturday, September 29th, 2012, 8:52 pm">8:52 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Geoff,</p>
            <p>Thanks for the great feedback! Sorry you hit problems with this. The post did not go as far as checking out the repositories for added users, so I think your advice will be a great help to anyone reading this post.</p>
            <p>Thanks,<br />
Phil</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
        <li id="comment-26800" class="comment odd alt depth-2 comment reader">
          <img alt="Javier" src="https://1.gravatar.com/avatar/11891fe2e1571de97c87379be9f71b3c?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">Javier</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, November 22nd, 2012, 8:51 am">November 22, 2012</abbr> at <abbr class="comment-time" title="Thursday, November 22nd, 2012, 8:51 am">8:51 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi,<br />
I see the message “remote: Initialized empty Git repository in /home/gitolite/repositories/bigfastblog.git/”<br />
But… when I my local machine I try to clone this not works <img src="/media/images/smilies/icon_sad.gif" alt=":(" class="wp-smiley" />
              <br />
$git clone gitbox:bigfastblog<br />
======<br />
FATAL: R any bigfastblog gitolite DENIED by fallthru<br />
(or you mis-spelled the reponame)<br />
fatal: The remote end hung up unexpectedly<br />
======</p>
            <p>Ok… In<br />
vim ~/.ssh/config<br />
I need add<br />
@admin = gitolite</p>
            <p>And now works :S</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
        <li id="comment-43404" class="comment even depth-2 comment reader">
          <img alt="silviu" src="https://1.gravatar.com/avatar/ddc65d49ac32a0167c764340e9ee956e?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">silviu</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, April 24th, 2013, 4:29 am">April 24, 2013</abbr> at <abbr class="comment-time" title="Wednesday, April 24th, 2013, 4:29 am">4:29 am</abbr>
          </div>
          <div class="comment-text">
            <p>thnx alot guys,<br />
the top quality tutorial from phil and your hint,geoff, saved my day.</p>
            <p>cheers</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-24783" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Fredrik" src="https://1.gravatar.com/avatar/7bc7aab38a0e3328bfb9785668892fdb?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Fredrik</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, November 3rd, 2012, 8:42 am">November 3, 2012</abbr> at <abbr class="comment-time" title="Saturday, November 3rd, 2012, 8:42 am">8:42 am</abbr>
      </div>
      <div class="comment-text">
        <p>Great tutorial! Usually you face some sort of a obstacle following a tutorial, but here everything just worked flawless and you also explained each steps very well. I usually do not reply on tutorials (which i guess is somewhat rude, but here i had to make an exception) Thanks alot!</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-24785" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, November 3rd, 2012, 9:19 am">November 3, 2012</abbr> at <abbr class="comment-time" title="Saturday, November 3rd, 2012, 9:19 am">9:19 am</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Fredrik! Glad you found it useful. Please tweet or share on your favorite social network, so that others more easily find it.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-27624" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Oguz Meteer" src="https://1.gravatar.com/avatar/f8161911f95714b93ac6f4a2a6b2d858?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://guztech.nl">Oguz Meteer</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, November 30th, 2012, 12:43 pm">November 30, 2012</abbr> at <abbr class="comment-time" title="Friday, November 30th, 2012, 12:43 pm">12:43 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Hi Phil,</p>
        <p>Thanks for the clear tutorial. However I’m having some difficulty installing gitolite. The step where you have to execute:</p>
        <p>gitolite/install -to /path/to/home/directory/of/user/gitolite</p>
        <p>gives me a “Permission denied” error. I have not created the gitolite home directory in /home/gitolite, because I’m running a RAID5 mirror on my server, and wanted to place the home directory there. Since I used sudo su – gitolite, I would not have expected this error. Any clues?</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-29279" class="comment even thread-even depth-1 comment reader">
      <img alt="chelifero" src="https://0.gravatar.com/avatar/e5d38633af71f6abd494a0bfa71378f0?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">chelifero</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, December 18th, 2012, 7:33 am">December 18, 2012</abbr> at <abbr class="comment-time" title="Tuesday, December 18th, 2012, 7:33 am">7:33 am</abbr>
      </div>
      <div class="comment-text">
        <p>Thanks Phil for this tutorial!!!<br />
But…<br />
I want use the same user account to admin gitolite and collaborate, so in my machine I’ve the gitolite-admin folder with this conf/gitolite.conf file:</p>
        <p>@admin      =   user1<br />
@staff      =   user1</p>
        <p>repo gitolite-admin<br />
    RW+     =   gitolite @admin</p>
        <p>repo testing<br />
    RW+     =   @all</p>
        <p>repo test_repository<br />
    RW+     =   user1 @admin<br />
    R       =   @staff</p>
        <p>and this ~/.ssh/config file:</p>
        <p>Host gitbox<br />
Hostname remote_repo_ip_address<br />
User gitolite<br />
Port 22<br />
IdentityFile ~/.ssh/user1</p>
        <p>When I try to push conf/gitolite.conf file changes, I see at terminal:</p>
        <p>user1@machine1 ~/gitolite-admin $ git push<br />
FATAL: W any gitolite-admin user1 DENIED by fallthru<br />
(or you mis-spelled the reponame)</p>
        <p>Instead if I modify ~/.ssh/config file in this way:</p>
        <p>Host gitbox<br />
Hostname remote_repo_ip_address<br />
User gitolite<br />
Port 22<br />
IdentityFile ~/.ssh/gitolite</p>
        <p>Push is ok, but I can’t work with git as user1. I can’t switch between these file version of file for working.<br />
What am I doing wrong? Is that possible?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-29285" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Tuesday, December 18th, 2012, 9:19 am">December 18, 2012</abbr> at <abbr class="comment-time" title="Tuesday, December 18th, 2012, 9:19 am">9:19 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi Chelifero,</p>
            <p>gitolite.conf looks ok to me.</p>
            <p>Do you have a key for that user? You should have this file ~/gitolite-admin/keydir/user1.pub which will be the public key of the user1 user.</p>
            <p>user1 cannot push the user1.pub key for the gitolite-admin repo, or add itself as an admin, but it sounds as though you done that with the gitolite user (at least the gitolite.conf part).</p>
            <p>   cp ~/.ssh/user1.pub ~/gitolite-admin/keydir/user1.pub<br />
   cd ~/gitolite-admin<br />
   git add keydir/user1.pub<br />
   git commit -m ‘added pub key for user1′<br />
   git push</p>
            <p>Now you should be able to edit the IdentityFile of User gitolite in .ssh/config to point to ~/.ssh/user1</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-29328" class="comment even depth-3 comment reader">
              <img alt="chelifero" src="https://0.gravatar.com/avatar/e5d38633af71f6abd494a0bfa71378f0?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn">chelifero</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Wednesday, December 19th, 2012, 12:36 am">December 19, 2012</abbr> at <abbr class="comment-time" title="Wednesday, December 19th, 2012, 12:36 am">12:36 am</abbr>
              </div>
              <div class="comment-text">
                <p>Hi Phil,<br />
thanks for your answer <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
                </p>
                <p>I’ve already done staging, commit and push the pub key for the user1. I see that with the command git log in user1 pc and I see the key file in /home/gitolite on the remote pc…</p>
                <p>What do you think about permission of /home/gitolite/user1.pub file on the remote pc? I seek:<br />
chown gitolite:gitolite /home/gitolite/user1.pub<br />
… and …<br />
chown user1:user1 /home/gitolite/user1.pub<br />
but not change the result.</p>
              </div>
              <!-- .comment-text -->
              <ol class="children">
                <li id="comment-29368" class="comment byuser comment-author-admin bypostauthor odd alt depth-4 comment role-administrator user-admin entry-author">
                  <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
                  <div class="comment-meta comment-meta-data">
                    <div class="comment-author vcard">
                      <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
                    </div>
                    <!-- .comment-author .vcard -->
                    <abbr class="comment-date" title="Wednesday, December 19th, 2012, 12:43 pm">December 19, 2012</abbr> at <abbr class="comment-time" title="Wednesday, December 19th, 2012, 12:43 pm">12:43 pm</abbr>
                  </div>
                  <div class="comment-text">
                    <p>Maybe try posting your issue to Gitolite mailing list…<br />
https://groups.google.com/forum/?fromgroups=#!forum/gitolite</p>
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
    <li id="comment-44058" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Joe" src="https://0.gravatar.com/avatar/2903b2adf218be15aafd5e445c1420be?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Joe</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, April 30th, 2013, 8:49 am">April 30, 2013</abbr> at <abbr class="comment-time" title="Tuesday, April 30th, 2013, 8:49 am">8:49 am</abbr>
      </div>
      <div class="comment-text">
        <p>Great stuff – explains a lot of what’s happening when trying to access your git repos via gitolite.</p>
        <p>I need some guidance on the ssh side of things.</p>
        <p>I’ve installed Gitolite and Gitlab 5.0 (Mac OS X Mountain Lion Server – 10.8)<br />
I can see the Gitlab page just fine – and can add and create users, projects, etc.  I can then look on the machine under the repositories and satellites directory and see everything is there hunky-dorey….</p>
        <p>The problem is when I try to clone down up push/pull up to a rego.  I can not connect for the life of me – and I’m sure it has something to do with how SSH is either configured or what the git account is being set up as when trying to grab from a repo.</p>
        <p>For instance when I do:  ssh git@fe77.com (that’s the gitolite machine), I get:</p>
        <p>PTY allocation request failed on channel 0 </p>
        <p>(that’s when on the machine and doing it locally – when on another machine and doing that command – I get in).</p>
        <p>Now, this is different than what I should get.  I should get a listing of what repos are allowed with their RW+, etc settings.</p>
        <p>I even try :  ssh git@fe77.com info</p>
        <p>info: Writing node (dir)Top…<br />
info: Done.<br />
File: dir  Node: Top  This is the top of the INFO tree<br />
  This (the Directory node) gives a menu of major topics.<br />
  Typing “d” returns here, “q” exits, “?” lists all INFO commands, “h”<br />
  gives a primer for first-timers, “mEmacs” visits the Emacs topic,<br />
  etc…</p>
        <p>(and a whole lot of stuff about emacs, etc)…</p>
        <p>Logically, I should get the access rites for repos to Gitolite again – doesn’t happen.</p>
        <p>Then, the coup d’etat:</p>
        <p>www:temp git$ git clone git@fe77.com:gitolite-admin<br />
Cloning into ‘gitolite-admin’…<br />
fatal: ‘gitolite-admin’ does not appear to be a git repository<br />
fatal: Could not read from remote repository.</p>
        <p>Please make sure you have the correct access rights<br />
and the repository exists.</p>
        <p>And yes, gitolite-admin is there, and I can pull it down via:</p>
        <p>git clone git@fe77.com:repositories/gitolite-admin</p>
        <p>But many sites mention that this is the WRONG way to do this since it bypasses Gitolite then – and yes, that is proven in that after pulling it down that way, I can’t push anything up – too numerous errors to describe.</p>
        <p>So, my question is this:  how do I get SSH on this machine to be setup correctly, etc? </p>
        <p>I have my .bashrc configured as such (and I know this is what’s called when I do that ssh call):</p>
        <p>www:~ git$ more .bashrc<br />
export GIT_EXEC_PATH=/usr/libexec/git-core</p>
        <p>PATH=$HOME/.rvm/bin:$HOME/gitolite/src:$HOME/bin:$PATH # Add RVM to PATH for scripting</p>
        <p>Any pointers or help would be fantastic.</p>
        <p>Thanks.</p>
        <p>Joe…</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-44420" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Lionel Abderemane" src="https://0.gravatar.com/avatar/cee55c1a175fdd1657671bf6e24682ff?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Lionel Abderemane</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, May 4th, 2013, 2:10 am">May 4, 2013</abbr> at <abbr class="comment-time" title="Saturday, May 4th, 2013, 2:10 am">2:10 am</abbr>
      </div>
      <div class="comment-text">
        <p>It is really clear Thank you</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-44872" class="pingback even thread-odd thread-alt depth-1 pingback reader">
      <img alt="Getting started with git ← Adm.in Berlin" src="https://0.gravatar.com/avatar/?d=https://www.bigfastblog.com/css/library/media/images/pingback.png&amp;s=80" class="avatar avatar-80 photo avatar-default" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://adminberlin.de/getting-started-with-git/">Getting started with git ← Adm.in Berlin</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, May 9th, 2013, 5:53 am">May 9, 2013</abbr> at <abbr class="comment-time" title="Thursday, May 9th, 2013, 5:53 am">5:53 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>[...] During the journey I was inspired by: https://www.bigfastblog.com/gitolite-installation-step-by-step [...]</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-45816" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="D Israel" src="https://0.gravatar.com/avatar/2d3cb0cd560fa69f41eca7f9ef417a77?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">D Israel</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, May 17th, 2013, 2:14 pm">May 17, 2013</abbr> at <abbr class="comment-time" title="Friday, May 17th, 2013, 2:14 pm">2:14 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>This is great!  However, I think I’m a little unclear about something.</p>
        <p>You create a key for each user, but how does gitolite know which user you are?  Do I have to do something else there?</p>
        <p>Thanks again for this.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-45819" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Friday, May 17th, 2013, 3:11 pm">May 17, 2013</abbr> at <abbr class="comment-time" title="Friday, May 17th, 2013, 3:11 pm">3:11 pm</abbr> | Permalink  </div>
          <div class="comment-text">
            <p>It knows based on which ssh key you authenticate with.</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-45820" class="comment odd alt depth-3 comment reader">
              <img alt="D Israel" src="https://0.gravatar.com/avatar/2d3cb0cd560fa69f41eca7f9ef417a77?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn">D Israel</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Friday, May 17th, 2013, 3:18 pm">May 17, 2013</abbr> at <abbr class="comment-time" title="Friday, May 17th, 2013, 3:18 pm">3:18 pm</abbr> | Permalink  </div>
              <div class="comment-text">
                <p>I’m new to these key settings.  Does this mean I change my ssh config file to the new username and identity file ?  I didn’t see where you made that change.</p>
                <p>Thanks again.</p>
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
    <li id="comment-45821" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="D Israel" src="https://0.gravatar.com/avatar/2d3cb0cd560fa69f41eca7f9ef417a77?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">D Israel</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, May 17th, 2013, 3:22 pm">May 17, 2013</abbr> at <abbr class="comment-time" title="Friday, May 17th, 2013, 3:22 pm">3:22 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Oh oh.  Is that what the comment (above) from Geoff Ford is about?  Just replace the Identity file with the newly created private key (User remains gitolite)?</p>
        <p>Thanks.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-45832" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Geoff Ford" src="https://0.gravatar.com/avatar/a1534fcf8cbea6b06f93f9b4cb57c1b9?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Geoff Ford</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, May 17th, 2013, 6:04 pm">May 17, 2013</abbr> at <abbr class="comment-time" title="Friday, May 17th, 2013, 6:04 pm">6:04 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>
          <img src="/media/images/smilies/icon_wink.gif" alt=";-)" class="wp-smiley" />
        </p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-46301" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Ovidiu Visan" src="https://0.gravatar.com/avatar/4c470a428265a3ef03584c1549629bad?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Ovidiu Visan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, May 21st, 2013, 1:05 pm">May 21, 2013</abbr> at <abbr class="comment-time" title="Tuesday, May 21st, 2013, 1:05 pm">1:05 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Thank you very much for this tutorial. It worked great!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-46608" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Amit Dixit" src="https://0.gravatar.com/avatar/4f33162114a37a3a689c11789ddb7356?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Amit Dixit</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, May 24th, 2013, 12:20 am">May 24, 2013</abbr> at <abbr class="comment-time" title="Friday, May 24th, 2013, 12:20 am">12:20 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>I did exactly the same as Geoff Ford described to add new user with new workstation.</p>
        <p>My .ssh/config file was as below:</p>
        <p>Host gitbox<br />
User gitolite<br />
Hostname new_host_ip<br />
Port 22<br />
IdentityFile ~/.ssh/new_user</p>
        <p>But every time it’s ask for password of gitolite user on my machine.<br />
gitolite@new_host_ip’s password: </p>
        <p>But once I changed the hostname to gitbox, it works fine. The changes I made is as:</p>
        <p>Host gitbox<br />
User gitolite<br />
Hostname gitbox<br />
Port 22<br />
IdentityFile ~/.ssh/new_user</p>
        <p>I believe hostname should be our local machine or new host that need to add, Can anyone explain to me why it’s happened.</p>
        <p>Thanks,<br />
Amit Dixit</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-50406" class="comment byuser comment-author-chinmay-hundekari even thread-odd thread-alt depth-1 comment role-subscriber user-chinmay-hundekari">
      <img alt="chinmay hundekari" src="https://0.gravatar.com/avatar/a5ffec0337476e283914938185abc7b3?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.google.com/profiles/111354133050015265445">chinmay hundekari</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, June 24th, 2013, 9:56 pm">June 24, 2013</abbr> at <abbr class="comment-time" title="Monday, June 24th, 2013, 9:56 pm">9:56 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Thanks a lot for this article.. I finally managed to set up gitolite. </p>
        <p>But I am not able to commit into a new repository. I added  a line in gitolite.conf and i was able to push it and creat a blank repo.<br />
But then when i try to push with the below command it asks for my password. and i am not able to proceed forward. I am not asked a password while doing a commit on gitolite repository.</p>
        <p>git push –all git@ubuntu3:ABCD</p>
        <p>ubuntu3 is hostname. and ABCD is the repository.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-51413" class="comment byuser comment-author-josivanps odd alt thread-even depth-1 comment role-subscriber user-josivanps">
      <img alt="Josivan Souza" src="https://0.gravatar.com/avatar/a64089a9f58239f20b084fcbb029d278?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.google.com/profiles/101861729150657836468">Josivan Souza</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, July 2nd, 2013, 6:44 pm">July 2, 2013</abbr> at <abbr class="comment-time" title="Tuesday, July 2nd, 2013, 6:44 pm">6:44 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>After create a repository following your blog. How can I clone it from a windows wosktation.</p>
        <p>Thanks</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-52027" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Ido" src="https://1.gravatar.com/avatar/132cfd78c9bce6ebcac20a0a755af6f6?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Ido</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, July 7th, 2013, 11:36 pm">July 7, 2013</abbr> at <abbr class="comment-time" title="Sunday, July 7th, 2013, 11:36 pm">11:36 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Hi Phil,</p>
        <p>I followed your blog (Which is great BTW) but I still face permission problem when I try:</p>
        <p>Idos-MacBook-Air:~ idofishler$ git clone amazongit:gitolite-admin.git<br />
Cloning into ‘gitolite-admin’…<br />
FATAL: R any gitolite-admin amazongit DENIED by fallthru<br />
(or you mis-spelled the reponame)<br />
fatal: Could not read from remote repository.</p>
        <p>Please make sure you have the correct access rights<br />
and the repository exists.</p>
        <p>I using an EC2 instance – so I suspect something to do with amazon restriction? Any idea about that?</p>
        <p>BR,<br />
Ido</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-52034" class="comment odd alt depth-2 comment reader">
          <img alt="Ido" src="https://1.gravatar.com/avatar/132cfd78c9bce6ebcac20a0a755af6f6?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">Ido</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, July 8th, 2013, 12:25 am">July 8, 2013</abbr> at <abbr class="comment-time" title="Monday, July 8th, 2013, 12:25 am">12:25 am</abbr> | Permalink  </div>
          <div class="comment-text">
            <p>I was able to solve this!</p>
            <p>I soon found that the user you login with let’s call it ‘git’ and the key file name – must be identical.<br />
In addition I had to edit the /home/git/.gitolite/conf/gitolite.conf file to add add the the user ‘git’ permissions on this gitolite-admin repository. Now it look like so:</p>
            <p>repo gitolite-admin<br />
    RW+     =   git</p>
            <p>repo testing<br />
    RW+     =   @all</p>
            <p>I also deleted all the other keys form /home/git/.gitolite/keydir<br />
Then run ‘/home/git/bin/gitolite setup again.</p>
            <p>Thanks again for this wonderful blog post!</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-52117" class="comment byuser comment-author-admin bypostauthor even depth-3 comment role-administrator user-admin entry-author">
              <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Monday, July 8th, 2013, 6:08 pm">July 8, 2013</abbr> at <abbr class="comment-time" title="Monday, July 8th, 2013, 6:08 pm">6:08 pm</abbr> | Permalink  </div>
              <div class="comment-text">
                <p>Great! Glad you solved it Ido. Thanks for also posting the resolution! I just got back from a trip to Japan, so was not up-to-date with comments.</p>
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
    <li id="comment-53205" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Shuaib" src="https://0.gravatar.com/avatar/0888b222c458d3343ff73af28f6f4332?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Shuaib</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, July 17th, 2013, 8:06 am">July 17, 2013</abbr> at <abbr class="comment-time" title="Wednesday, July 17th, 2013, 8:06 am">8:06 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Hi Phil,</p>
        <p>What’s the recommended setup when you want to host the repositories<br />
directory somewhere other than /home/git on your server?</p>
        <p>Thanks!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-53213" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Eshwar" src="https://0.gravatar.com/avatar/a3397c28170891359f35334ad05d09ee?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Eshwar</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, July 17th, 2013, 8:44 am">July 17, 2013</abbr> at <abbr class="comment-time" title="Wednesday, July 17th, 2013, 8:44 am">8:44 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Phil,</p>
        <p>I followed through your article, but i couldn’t clone gitolite:admin repo.<br />
I have created a user called ‘gitolitee’ as the user ‘gitolite’ is reserved. </p>
        <p>i have created gitolitee &amp; gitolitee.pub and copied to server through<br />
  scp .ssh/gitolitee.pub gitolitee@ubutu:</p>
        <p>then i cloned the repo and installed it. But when i clone admin repo in local machine it is asked for password.</p>
        <p>Could you help me in this.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-64491" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Mehran" src="https://0.gravatar.com/avatar/028a8a9e7e55f56926b79dcbc5e533fd?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Mehran</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, September 6th, 2013, 5:53 am">September 6, 2013</abbr> at <abbr class="comment-time" title="Friday, September 6th, 2013, 5:53 am">5:53 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Hey phil when i run this command.<br />
git clone gitbox:myserver</p>
        <p>system asks for<br />
gitosis:myserver’s password:</p>
        <p>I dont know which password …is it password of my server. I tried but it didnot worked.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-75894" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Shlomit" src="https://1.gravatar.com/avatar/d0a0866532551036bf55e3996d763690?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Shlomit</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, October 13th, 2013, 4:02 am">October 13, 2013</abbr> at <abbr class="comment-time" title="Sunday, October 13th, 2013, 4:02 am">4:02 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Hi,</p>
        <p>Your tutorial is so clear – Thanks,</p>
        <p>I feel something missing or I don’t understand something.<br />
I have git install and now I add installation of gitolite.<br />
How git know to go through gitolite? </p>
        <p>Thanks again for this wonderful tutorial.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-75966" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Shlomit" src="https://1.gravatar.com/avatar/d0a0866532551036bf55e3996d763690?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Shlomit</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, October 13th, 2013, 6:27 am">October 13, 2013</abbr> at <abbr class="comment-time" title="Sunday, October 13th, 2013, 6:27 am">6:27 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>It’s me again,<br />
I saw that even that I change the file conf/gitolite.conf and I run ‘gitolite setup’, when I run ssh to the git server I got the basic conf that came with the gitolite installation.<br />
How can I refresh the conf file?</p>
        <p>Thanka a lot.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-123122" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="hopsj" src="https://0.gravatar.com/avatar/6260fd63a04f073776af1920a4b8f53d?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">hopsj</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, November 21st, 2013, 1:20 am">November 21, 2013</abbr> at <abbr class="comment-time" title="Thursday, November 21st, 2013, 1:20 am">1:20 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Hello and thank you for your tutorial. One suggestion that may help make things more clear is to use usernames that are not exactly the same as the name of the program.  It can be hard for noobs such as myself to interpret the meaning of ‘gitolite’ especially when it appears 4 times in the same command (username, directory, file?)  I found Sitaram’s own tutorials to be difficult to follow for this reason as well.  I would suggest something like gitadmin to refer to the gitolite user, and real-world names such as bob to refer to individual users.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-124066" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, November 21st, 2013, 5:45 pm">November 21, 2013</abbr> at <abbr class="comment-time" title="Thursday, November 21st, 2013, 5:45 pm">5:45 pm</abbr> | Permalink  </div>
          <div class="comment-text">
            <p>Thanks! That’s great feedback and will keep it in mind for future posts.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-231334" class="comment even thread-even depth-1 comment reader">
      <img alt="yewin" src="https://0.gravatar.com/avatar/4073f537d4eb9f9655b4c59b42d28cc1?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://-">yewin</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, March 5th, 2014, 9:31 pm">March 5, 2014</abbr> at <abbr class="comment-time" title="Wednesday, March 5th, 2014, 9:31 pm">9:31 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Hi Phil,</p>
        <p>Thank you for your wonderful tutorial.</p>
        <p>But I feel little wrong in my workstation.</p>
        <p>I setting up gitolite server on linux and clone from eclipse windows.<br />
So I generate SSH key from elipse from windows side.<br />
Windows &gt; Preferences &gt; General &gt; Network Connections &gt; SSH 2 &gt; Key Management &gt; Generate RSA Key.<br />
I change the public key name “bob.pub” and copy to my linux server.</p>
        <p>And then I follow the “Everyone gets a key” section as the end of post.<br />
But I always face the following errors.</p>
        <p>ssh://gitolite@xx.xx.x.xx:22/home/gitolite/repositories/foo.git FATAL: R  home/gitolite/repositories/foo bob DENIED by fallthru.(or miss-spelled the repo name)</p>
        <p>I try the numerous repository path.:<br />
git+ssh://gitolite@xx.xxx.xx:22/foo<br />
git+ssh://gitolite@xx.xxx.xx:22/foo.git</p>
        <p>It not work and still errors.<br />
Any suggestion about that??<br />
I always appreciate your suggestion.</p>
        <p>Thank Again.<br />
yewin</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-259565" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="rballen" src="https://0.gravatar.com/avatar/0a41972bb2c209ddf959de8cf54d239e?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">rballen</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, April 4th, 2014, 6:54 am">April 4, 2014</abbr> at <abbr class="comment-time" title="Friday, April 4th, 2014, 6:54 am">6:54 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Thanks man! I’ve spent hours going round and round trying to install gitolite.  Then I found your article 20 minutes ago and now my pi gitbox is finally working perfectly. Awesome!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-275000" class="comment even thread-even depth-1 comment reader">
      <img alt="Maciej Cygan" src="https://0.gravatar.com/avatar/84deed6fb98039ae8af09837dabdfb26?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Maciej Cygan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, April 25th, 2014, 4:21 pm">April 25, 2014</abbr> at <abbr class="comment-time" title="Friday, April 25th, 2014, 4:21 pm">4:21 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Hello Phil</p>
        <p>I have tested this and it works great. However !<br />
I am now trying to integrate Gitolite with Redmine and its a pain in the ass and i can not get it to work .. is there a chance that you could amend this / or create new tutorial to show how to integrate gitolite with redmine ? I would really appreciate it !! </p>
        <p>Thanks</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

