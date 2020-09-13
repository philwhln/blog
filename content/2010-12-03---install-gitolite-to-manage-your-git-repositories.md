---
title: "Install Gitolite To Manage Your Git Repositories"
date: "2010-12-03T11:03:00-08:00"
template: "post"
draft: false
slug: "install-gitolite-to-manage-your-git-repositories"
category: "Software Engineering"
tags:
  - code repository
  - git
  - gitolite
  - install
  - installation
  - mac osx
  - ssh-keygen
description: "This post has been depreciated. A newer post is available. Gitolite Installation Step-By-Step Recently, netSIGN asked me to setup gitolite to give"
---
<div style="border: 1px #ddd solid; margin: 5px; padding: 5px; background-color: #FFFF99;">
<strong>This post has been depreciated. A newer post is available.<br/>
<a href="/gitolite-installation-step-by-step">Gitolite Installation Step-By-Step</a><br/>
</strong></div>

Recently, [netSIGN](https://www.netsign.com) asked me to setup [gitolite](https://github.com/sitaramc/gitolite) to give external developers controlled access to git repositories. Gitolite enables easy management of this access control. In this post I will detail how I set this up.

The first thing to note about the gitolite install is that the installer is run remotely. Therefore, you will want to download the gitolite installation code onto your local machine.

```
git clone git://github.com/sitaramc/gitolite
```

This will fetch the gitolite code from github.

```
Cloning into gitolite...
remote: Counting objects: 3156, done.
remote: Compressing objects: 100% (1438/1438), done.
remote: Total 3156 (delta 2149), reused 2479 (delta 1681)
Receiving objects: 100% (3156/3156), 699.61 KiB | 268 KiB/s, done.
Resolving deltas: 100% (2149/2149), done.
```

You will need to setup the git user account on the remote machine, under which gitolite will run, so login. 

```
ssh gitbox
```

_gitbox_ is the hostname of the remote machine I am using. You can replace this with your remote machine’s hostname or IP.

Now create the user. I’m calling my user “gitolite”, but you can use “git” or anything else.

```
sudo adduser \
  --system \
  --shell /bin/bash \
  --gecos 'git version control' \
  --group \
  --disabled-password \
  --home /home/gitolite gitolite
```

In this example above _/home/gitolite_ is where gitolite and your code repositories will live.

Now you can return to your local machine.

```
exit
```

Notice that when we created the user, we used _–disable-password_, which prevents us logging into the machine using a password. Therefore we’ll need to upload a ssh key for running the installer. Here, I will create a public and private keypair with the name _id\_rsa\_gitolite_.

```
cd ~/.ssh
ssh-keygen -t rsa -f id_rsa_gitolite
cd ~

```

Hit return at the prompts to create the key without passphrase authentication.

You public key can be found here.

```
~/.ssh/id_rsa_gitolite.pub
```

And the private key here.

```
~/.ssh/id_rsa_gitolite
```

Now you’ll need to upload the public key to gitolite user account, so that we can log into that account using our private key.

```
scp ~/.ssh/id_rsa_gitolite.pub gitbox
```

Now login to the remote machine

```
ssh gitbox
```

and copy the key to the gitolite account.

```
sudo cp id_rsa_gitolite.pub /home/gitolite
sudo chown gitolite:gitolite /home/gitolite/id_rsa_gitolite.pub
```

Become the _gitolite_ user

```
sudo su - gitolite
```

and add the gitolite public key to the list of authorized keys that can be used to login as this user.

```
mkdir .ssh
chmod 700 .ssh
cat id_rsa_gitolite.pub >> .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
rm id_rsa_gitolite.pub

```

Now it’s time to return to you local machine.

```
exit # from gitolite user
exit # from remote machine

```

To make things simple on ssh side I recommend adding the configuration for the gitolite account to you ssh config.

```
vim ~/.ssh/config
```

```
Host gitbox
   User gitolite
   Hostname gitbox
   Port 22
   IdentityFile ~/.ssh/id_rsa_gitolite

```

Now you should be able to login to the remote machine as the gitolite user using the following…

```
ssh gitbox
exit
```

The installer command _gl-easy-install_ takes the following arguments

```
gl-easy-install <user> <host> [ <port> ] <admin name> <host nickname>
```

If _port_ is not given it will default to _22_.

Now you can run the gitolite installer using the gitolite code we downloaded.

```
cd gitolite/src
./gl-easy-install gitolite gitbox gitadmin

```

If all went well you should have a checked-out gitolite-admin git repository in your home directory.

```
cd ~/gitolite-admin
```

This will be used for managing your users and git repositories. By simply editing _conf/gitolite.conf_ and pushing it to the gitolite server you can create new repositories. Adding new users will involve adding an ssh key to the _keydir_. I will cover more on these in a follow-up post.

## Comments

<div id="comments">
  <ol class="comment-list">
    <li id="comment-168" class="comment byuser comment-author-admin bypostauthor even thread-even depth-1 comment role-administrator user-admin entry-author">
      <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, December 3rd, 2010, 11:07 am">December 3, 2010</abbr> at <abbr class="comment-time" title="Friday, December 3rd, 2010, 11:07 am">11:07 am</abbr>
      </div>
      <div class="comment-text">
        <p>More great gitolite information can be found on the gitolite github page<br />
https://github.com/sitaramc/gitolite</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-849" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="airtonix" src="https://0.gravatar.com/avatar/454cea4c773e4e7ef6ceeb7ef0e18bd9?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">airtonix</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, January 25th, 2011, 4:29 am">January 25, 2011</abbr> at <abbr class="comment-time" title="Tuesday, January 25th, 2011, 4:29 am">4:29 am</abbr>
      </div>
      <div class="comment-text">
        <p>1. Instead of hardconding the customisable names you should use a step to customise them : </p>
        <p>$REMOTE_HOST=”gitbox”<br />
$REMOTE_USER=”Phil”</p>
        <p>2. you need to specify the home directory of the remote user when copying the public key to the remote host</p>
        <p>scp ~/.ssh/id_rsa_gitolite.pub gitbox</p>
        <p>needs to be </p>
        <p>scp ~/.ssh/id_rsa_gitolite.pub $REMOTE_HOST:/home/$REMOTE_USER</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-855" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Tuesday, January 25th, 2011, 9:39 am">January 25, 2011</abbr> at <abbr class="comment-time" title="Tuesday, January 25th, 2011, 9:39 am">9:39 am</abbr>
          </div>
          <div class="comment-text">
            <p>Excellent suggestion, Airtonix. I’m finding that to be a better way to go as I write posts, as it makes it easier for others to simply cut-and-paste the commands.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-1524" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Website design" src="https://1.gravatar.com/avatar/9a7cadee6e1855db142177d0453a3376?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.maastermedia.com">Website design</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, March 13th, 2011, 2:31 pm">March 13, 2011</abbr> at <abbr class="comment-time" title="Sunday, March 13th, 2011, 2:31 pm">2:31 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Nice fast tutorial. Thank you.</p>
        <p>One remark though. Regarding “cat id_rsa_gitolite.pub &gt;&gt; .ssh/authorized_keys” …</p>
        <p>In my case gitolite user’s authorized_keys file needs to have following format in them:</p>
        <p>command=”/usr/share/gitolite/gl-auth-command johndoe”,no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAIEAvwKWiIoF23S6TXMEr8H2U18hkpuPrt5nOsUhqvR7XB8Wpkf7Al5SKNpgpfb/4CGVrSSzDvwmTN/cO6SDO3td8h1NBVl0APaAmZ7x6RFyoN5NCco/raOfVK+0Ktwg1Yoq7S8TdUKRP1phDHnHnlSkwbhzk1TETOEiSZTboH6FMHs johndoe@hostname</p>
        <p>Only putting pub key file in it did not work.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-3638" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Pradeep Sharma" src="https://0.gravatar.com/avatar/0bb106b95fc10c738cab68c357558262?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Pradeep Sharma</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, July 7th, 2011, 4:33 am">July 7, 2011</abbr> at <abbr class="comment-time" title="Thursday, July 7th, 2011, 4:33 am">4:33 am</abbr>
      </div>
      <div class="comment-text">
        <p>it should be ssh gitbox and not ssh gitolite</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-3645" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, July 7th, 2011, 11:06 am">July 7, 2011</abbr> at <abbr class="comment-time" title="Thursday, July 7th, 2011, 11:06 am">11:06 am</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Pradeep! I’ve updated the text.</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-3686" class="comment odd alt depth-3 comment reader">
              <img alt="Pradeep Sharma" src="https://0.gravatar.com/avatar/0bb106b95fc10c738cab68c357558262?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn">Pradeep Sharma</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Monday, July 11th, 2011, 12:49 am">July 11, 2011</abbr> at <abbr class="comment-time" title="Monday, July 11th, 2011, 12:49 am">12:49 am</abbr>
              </div>
              <div class="comment-text">
                <p>This how-to is really great. Do you know of some other article(may be yours), where I can get details on setting up collaborators.</p>
              </div>
              <!-- .comment-text -->
              <ol class="children">
                <li id="comment-3693" class="comment byuser comment-author-admin bypostauthor even depth-4 comment role-administrator user-admin entry-author">
                  <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
                  <div class="comment-meta comment-meta-data">
                    <div class="comment-author vcard">
                      <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
                    </div>
                    <!-- .comment-author .vcard -->
                    <abbr class="comment-date" title="Monday, July 11th, 2011, 9:02 am">July 11, 2011</abbr> at <abbr class="comment-time" title="Monday, July 11th, 2011, 9:02 am">9:02 am</abbr>
                  </div>
                  <div class="comment-text">
                    <p>This is probably what you’re looking for…<br />
https://progit.org/book/ch4-8.html#config_file_and_access_control_rules</p>
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
    <li id="comment-3799" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Tim Orph" src="https://1.gravatar.com/avatar/967ec6191f9d954005e791c8afbc1e63?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Tim Orph</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, July 19th, 2011, 3:13 am">July 19, 2011</abbr> at <abbr class="comment-time" title="Tuesday, July 19th, 2011, 3:13 am">3:13 am</abbr>
      </div>
      <div class="comment-text">
        <p>Does anyone know if it is posible to use server where gitolite is installed, as client and edit repository files ? i don’t know where does even gitolite save this files on server so i could try “git status” or smth.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-3802" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Tuesday, July 19th, 2011, 9:06 am">July 19, 2011</abbr> at <abbr class="comment-time" title="Tuesday, July 19th, 2011, 9:06 am">9:06 am</abbr>
          </div>
          <div class="comment-text">
            <p>Curious why you want to do this?</p>
            <p>You can easily checkout a git repository as a separate user on the same machine.</p>
            <p>The repositories will be found under /home/gitolite/repositories if you followed the instructions above.</p>
            <p>Possibly look at installing gitweb, as this may do all the things you want.<br />
https://progit.org/book/ch4-6.html<br />
https://groups.google.com/group/gitolite/msg/e7579cbd35dc1b3d</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-3803" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Tim Orph" src="https://1.gravatar.com/avatar/967ec6191f9d954005e791c8afbc1e63?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Tim Orph</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, July 19th, 2011, 9:56 am">July 19, 2011</abbr> at <abbr class="comment-time" title="Tuesday, July 19th, 2011, 9:56 am">9:56 am</abbr>
      </div>
      <div class="comment-text">
        <p>Of course.. this is what I should do <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
          <br />
Thank you for making me realize.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-3806" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="ed" src="https://0.gravatar.com/avatar/aff2c092406ded6e7093ebd0bccdb0fc?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">ed</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, July 19th, 2011, 4:04 pm">July 19, 2011</abbr> at <abbr class="comment-time" title="Tuesday, July 19th, 2011, 4:04 pm">4:04 pm</abbr>
      </div>
      <div class="comment-text">
        <p>i believe the steps:</p>
        <p>cd gitolite/src<br />
./gl-easy-install gitolite gitbox gitadmin<br />
 is initiated from the client side. but who is gitadmin? what privileges does it have and on where? </p>
        <p>for absolute beginner, it is not clear in the article. can you add more details? thanks</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-4839" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="dale" src="https://1.gravatar.com/avatar/70ef5a873859d8225bc42fd2df6d165c?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">dale</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, September 28th, 2011, 7:47 am">September 28, 2011</abbr> at <abbr class="comment-time" title="Wednesday, September 28th, 2011, 7:47 am">7:47 am</abbr>
      </div>
      <div class="comment-text">
        <p>If you open the gl-easy-install file line 143 has a comment that “this *must* be run as “src/gl-easy-install”, not by cd-ing to src and then running “./gl-easy-install.  You may want to update your gl-easy-install instructions.  </p>
        <p>Excellent article, I have been struggling with the gitolite install, there’s many docs on it, but your’s made the most sense.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-12568" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Michal" src="https://1.gravatar.com/avatar/d178acdade435f84e8ff43af1b838c3a?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Michal</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, April 14th, 2012, 10:05 pm">April 14, 2012</abbr> at <abbr class="comment-time" title="Saturday, April 14th, 2012, 10:05 pm">10:05 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Unfortunately the file gl-easy-install is no longer distributed with gitolite and this “remote” installation is no longer possible. In short – when you use non-root installation method, you create your git user, clone gitolite, run src/gl-system-install and later gl-setup YourName.pub (on the server), clone your gitolite-admin repo (to the workstation)</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-19373" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Roger Ivy" src="https://1.gravatar.com/avatar/df7b8a8bc19dd3bc020708a8fd59e2be?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Roger Ivy</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, July 18th, 2012, 2:35 pm">July 18, 2012</abbr> at <abbr class="comment-time" title="Wednesday, July 18th, 2012, 2:35 pm">2:35 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Phil, do you have updated instructions for the most recent versions of both Gotolite and Ubuntu (12.04LTS)?</p>
        <p>It seems that this command is no longer valid: gl-easy-install</p>
        <p>I’ve tried these instructions but hit permission issues: https://github.com/sitaramc/gitolite</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-19533" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, July 23rd, 2012, 10:12 pm">July 23, 2012</abbr> at <abbr class="comment-time" title="Monday, July 23rd, 2012, 10:12 pm">10:12 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Roger. I’ve written a new post https://www.bigfastblog.com/gitolite-installation-step-by-step<br />
This covers Ubuntu 12.04LTS and changes to Gitolite.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-19511" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Charles Burns" src="https://0.gravatar.com/avatar/ae14185b22f65c974fca9c67d7e876cb?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Charles Burns</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Monday, July 23rd, 2012, 12:26 pm">July 23, 2012</abbr> at <abbr class="comment-time" title="Monday, July 23rd, 2012, 12:26 pm">12:26 pm</abbr>
      </div>
      <div class="comment-text">
        <p>I second what Roger Ivy says above: Love to see an updated version of thbis how-to for the most recent version of gitolite, since gl-easy-install doesn’t exist anymore…</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-19532" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Monday, July 23rd, 2012, 10:11 pm">July 23, 2012</abbr> at <abbr class="comment-time" title="Monday, July 23rd, 2012, 10:11 pm">10:11 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Just wrote new post https://www.bigfastblog.com/gitolite-installation-step-by-step</p>
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

