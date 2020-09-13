---
title: "Hosting Images With Google Storage Manager"
date: "2011-01-14T16:32:00-08:00"
template: "post"
draft: false
slug: "hosting-images-google-storage-manager"
category: "Software Engineering"
tags:
  - dfs
  - distributed file storage
  - file uploader
  - file-server
  - gfs
  - google
  - google i/o
  - google storage
  - google storage manager
  - hosting
  - images
  - storage
  - upload
description: "Today I received my invite from Google Google Storage for Developers. Yes, like most of us, my life if wrapped in layers of googliness. In this post I'm going to review briefly what Google Storage is and upload the image files, for the images you see in this post, using the Google Storage Manager."
socialImage:
  publicURL: "/media/images/2011/01/google_storage_sq.jpg"
---
<a href="/hosting-images-google-storage-manager"><img alt="Google Storage" class="size-full wp-image-887" height="212" src="/media/images/google_storage.png" title="Google Storage" width="380"/></a>

Today I received my invite from Google “Google Storage for Developers”. Yes, like most of us, my life if wrapped in layers of googliness. In this post I’m going to review briefly what Google Storage is and upload the image files, for the images you see in this post, using the Google Storage Manager. 

## Overview

First, here is an overview from [Google I/O 2010 presentation](https://www.youtube.com/watch?v=GLbRvcbkAwE#t=12m56s) on the subject.

>  
> __Google Storage Overview__
> 
> *   Fast, scalable, highly available object store
> 
> *   Objects of any type and practically any size
> *   Data is replicated across multiple US data centers
> *   Read-your-writes data consistency
> 
> *   Easy, flexible authentication and sharing
> 
> *   Key-based authentication
> *   Authenticated downloads from a web browser
> *   Sharing with individuals and groups
> 
> *   Google products and 3rd party tools/services
> 
> 

## Storage Hierarchy

At the top-level of the storage system you have buckets and inside a bucket you have folders and files. You can nest folders, but you cannot nest buckets. You can think of buckets like your “C:” or “D:” drive on Windows, with all your files and folders under it. Google says, in it’s [Google I/O presentation](https://www.youtube.com/watch?v=GLbRvcbkAwE), that bucket names are unique across Google Storage, so if I call my bucket “blog”, then you cannot call your bucket “blog”.

## Uploading Files

I jumped right in and went to the [Google Storage Manager](https://sandbox.google.com/storage/#), which is a web interface for the storage.

<div class="wp-caption alignnone" id="attachment_879" style="width: 530px">
<a href="/media/images/no_buckets.png">
<img alt="Google Storage Manager" class="size-full wp-image-879" height="167" src="/media/images/no_buckets.png" title="Google Storage Manager" width="520"/>
</a>
<p class="wp-caption-text">Google Storage Manager</p>
</div>

Entered a my new bucket name “blog”…

<a href="/media/images/bucket_called_blog.png"><img alt='Google Storage Manager. New bucket called "blog"' class="size-full wp-image-880" height="155" src="/media/images/bucket_called_blog.png" title='Google Storage Manager. New bucket called "blog"' width="369"/></a>

Doh!

<blockquote style="color: red">
<p>Sorry, that name is not available. Please try a different one. (provided name: ‘blog’)</p>
</blockquote>

According to Google’s presentation, buckets are name-spaced on domain names that you own, but I think that functionality is not there yet. Instead, I tried a bucket that others were unlikely to have, “philwhln”. Bingo! I now have my first Google Storage bucket. Unlike adding a bucket, adding a folder is a little less like pinning a donkey with a blindfold on. You can call your folder anything you want, because it’s in a bucket that you own. I called mine “blog” and a sub-folder under that called “images”.

>  
> This folder is empty. Drag and drop files here to upload them.
> 

Google Storage seems pretty hungry for some files, so I dragged in some images for this post from my desktop. Once your files are uploaded you can click the tick-box to the right of each filename to share those files publicly with the Internet. I selectively chose the ones I wanted to share.

<div class="wp-caption alignnone" id="attachment_884" style="width: 659px">
<a href="/media/images/shared_images.png">
<img alt="Shared Images" class="size-full wp-image-884" height="293" src="/media/images/shared_images.png" title="Shared Images" width="649"/>
</a>
<p class="wp-caption-text">Use the tick-box to the right of the filename to select the images you wish to share</p>
</div>

The URL to these file look like this…  
[/media/images/google\_storage.png](/media/images/google_storage.png)  
[/media/images/no\_buckets.png](/media/images/no_buckets.png)

Apart from the rather long “commondatastorage” subdomain, those URLs a very nice. Clean and simple.

### Replacing A File

You can replace a file quite easily. You will get a prompt asking if you want to replace it. You should note that you will have to re-click the “share” tick next to the image name if you still want to share this, as it will become unshared when you replace it. Luckily, the URL of the file will not change.

## Conclusion

In this post I have shown my first encounter with Google Storage Manager and the ease of uploading some image files. If you notice any problems with the images loading in this post (slow loading, not loading) then please leave a comment below.

## Comments

<div id="comments">
  <ol class="comment-list">
    <li id="comment-963" class="comment even thread-even depth-1 comment reader">
      <img alt="Anton Babenko" src="https://1.gravatar.com/avatar/fc9fce3c16a287d672ec5433430f11ca?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://imagepush.to">Anton Babenko</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, February 4th, 2011, 1:22 pm">February 4, 2011</abbr> at <abbr class="comment-time" title="Friday, February 4th, 2011, 1:22 pm">1:22 pm</abbr>
      </div>
      <div class="comment-text">
        <p>It looks like an answer to my question here – https://www.quora.com/What-is-free-or-cheap-storage-for-public-images . Thanks for blog post.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-964" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Friday, February 4th, 2011, 1:26 pm">February 4, 2011</abbr> at <abbr class="comment-time" title="Friday, February 4th, 2011, 1:26 pm">1:26 pm</abbr>
          </div>
          <div class="comment-text">
            <p>You will find that many of the images on this blog are now hosted with Google Storage. I have had no problems [that I have noticed], so far.</p>
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

