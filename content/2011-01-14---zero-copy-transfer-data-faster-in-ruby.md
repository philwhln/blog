---
title: "Zero-Copy. Transfer Data Faster In Ruby"
date: "2011-01-14T13:27:00-08:00"
template: "post"
draft: false
slug: "zero-copy-transfer-data-faster-in-ruby"
category: "Ruby"
tags:
  - file-server
  - gem
  - io
  - io_splice
  - java
  - linux
  - ruby
  - splice
  - zero-copy
description: "In this post I will explain the concept behind zero-copy, which is feature of the Linux allowing for faster transfer of data between pipes, file-descriptors and sockets. I will demonstrate how you can use this functionality in your Ruby projects using a code example. This functionality has been implemented in C, Java, Ruby, Perl and nameless other languages, but in this blog I will focus on the Ruby usage."
socialImage:
  publicURL: "/media/images/2011/01/zero_copy_ruby_sq.jpg"
---
<a href="https://www.flickr.com/photos/zitona/3749382854/" title="McDreamy by Zitona on Flickr"><img align="left" alt="Zero-Copy In Ruby Image" height="323" src="/media/images/2011/01/zero_copy_ruby.jpg" style="margin-right: 15px" width="270"/></a>

In this post I will explain the concept behind “zero-copy”, which is feature of the Linux allowing for faster transfer of data between pipes, file-descriptors and sockets. I will demonstrate how you can use this functionality in your Ruby projects using a code example. This functionality has been implemented in C, Java, Ruby, Perl and nameless other languages, but in this blog I will focus on the Ruby usage.

## What Is “Zero-Copy”?

The name “zero-copy” comes from the fact that the data is not copied from kernel-space to user-space as it usually would be. If you are serving a file, receiving a file, writing to a socket or reading from a socket or pipe and have no interest in programatically looking at the data then you can let the kernel do all the work. Your application runs in user-space and your application defines what you want to do with data as it comes in from various filehandles or sockets and out through other filehandles and sockets. Therefore, it is understandable that the kernel usually needs to pass the data your application, in user-space memory. Zero-copy cuts out the middle-man, your application, when your application is not interested in seeing the data. Roughly speaking, with zero-copy, you can give the kernel two file-descriptors and tell it kernel to “pass the data from one to the other and let me know when you are done”.

## How Is This Faster?

### Less Context-Switching

If you are reading this, I’ll assume you are a software developer or someone who can imagine what life would be like if you were a developer (the money, the women, the fame…). Have you ever tried writing code and being the person on support at the same? It is not easy. Every time you roll up your sleeves and get stuck into fixing that infinite for-loop, you get a call and your brain has start thinking about the problem of person on the other end of line. A day of this and you will get little done. It would be better to just be on support or just fixing infinite for-loops.

When you serve up a static file to the web, the data is read from a file into your application and then your application writes it out to the socket. Even if you are using the lowest-level API in your chosen language for doing this, it is a two-step process and your application has made two requests to the kernel. “Read this”. “Write this”. Between these two steps the kernel is switching control back to your application and asking, “what next?”.

With zero-copy you are saying to kernel, “read from _here_ and write to _there_“. This is can be a single call to the kernel, which means less time passing control back to your application and saying, “what next?”.

### Less Copying

The second reason that zero-copy is faster is because the kernel is not giving you the data. If it did it would have to copy the data from kernel-space to user-space, since kernel-space is for the kernel’s eyes only. Instead, the kernel knows what to do with that data. It transfers data from one filehandle to the other filehandle without bothering your application unless there is an error, it has completed, or the application has requested the transfer to be [non-blocking](https://en.wikipedia.org/wiki/Asynchronous_I/O).

### “Splice”

This system call that does the magic is called [splice](https://www.kernel.org/doc/man-pages/online/pages/man2/splice.2.html). You can read the [Linux manual page here](https://www.kernel.org/doc/man-pages/online/pages/man2/splice.2.html).

```
ssize_t splice(int fd_in, loff_t *off_in, int fd_out,
               loff_t *off_out, size_t len, unsigned int flags);
```

The key to note here is that _one of the end-points must be a pipe_. If neither end-points are a pipe then you can create an intermediary pipe in kernel-space memory and tell the kernel to _write to_ and _read from_ that pipe. You can see [an example of this below](/zero-copy-transfer-data-faster-in-ruby#ruby-example-copy-file-to-file), written in Ruby, where we copy data from one file to another without moving any of the data in user-space memory.

## Server-Side Data Validation

The down-side to this is when you want to check the data before you send it out. It may make the server snappier, but if the data on disk is prone to corruption, then you would have to rely on the clients letting you know, since your application does not get chance to inspect the data.

## Other Factors

While you may take some of the burden off the CPU and there is less time spent copying memory, depending on your system and your usage you may not see much difference. Disk seek time and network latency often leave the CPU twiddling it’s thumbs. Memory copying is very fast. Therefore, you should weigh up the cost and benefits to this. The cost would be the possibly limiting your code to the Linux platform and an additional dependency of the extra code needed to do this (a gem in Ruby’s case).

##  Performance

If you’re interested in getting deeper into this or would like to approach it from more of a Java angle, then I recommend checking out “[Efficient data transfer through zero copy](https://www.ibm.com/developerworks/library/j-zerocopy/index.html)” by [Sathish K. Palaniappan](https://www.ibm.com/developerworks/library/j-zerocopy/index.html#author1) and [Pramod B. Nagaraja](https://www.ibm.com/developerworks/library/j-zerocopy/index.html#author2). It’s from here I take the following performance results.

<table>
<tr>
<th>File size</th>
<th>Non zero-copy</th>
<th>Zero-copy</th>
<th>How much faster?</th>
</tr>
<tr>
<td>7 Mb</td>
<td>156 ms</td>
<td>45ms</td>
<td>346 % faster</td>
</tr>
<tr>
<td>200 Mb</td>
<td>2124 ms</td>
<td>1150 ms</td>
<td>184 % faster</td>
</tr>
<tr>
<td>1 Gb</td>
<td>18399 ms</td>
<td>8537 ms</td>
<td>215 % faster</td>
</tr>
</table>

It is hard to make a clear opinion on the exact speed increase, since the performance change is non-linear, but saying that you get a 2x performance would not be unreasonable.

If you want to run your own tests to replicate this then [the Java source code](https://www.ibm.com/developerworks/apps/download/index.jsp?contentid=333543&amp;filename=j-zerocopy.zip&amp;method=http&amp;locale=worldwide) is available on their article.

Some other recent benchmarks by [Iñaki Baz Castillo](https://dev.sipdoc.net), which were written in Ruby, can be found [here](https://librelist.com/browser//ruby.io.splice/2010/12/22/some-benchmarks/). I use code from these benchmarks in [the example below](/zero-copy-transfer-data-faster-in-ruby#ruby-example-copy-file-to-file).

## Ruby Implementation

In Ruby there is an implementation of zero-copy called [io\_splice](https://bogomips.org/ruby_io_splice). This is not supported on Mac OS X, since zero-copy is Linux specific. You will get installation problems if you install this on anything less than Linux Kernel 2.6.17.

### Installing The Gem

You can install the gem in the same way you install other gems.

```
gem install io_splice
```

The usual gem install output…

```
Building native extensions.  This could take a while...
Successfully installed io_splice-2.2.0
1 gem installed
Installing ri documentation for io_splice-2.2.0...
Installing RDoc documentation for io_splice-2.2.0...
```

<a id="ruby-example-copy-file-to-file"></a>

### Ruby Example – Copy File To File

In this example I will copy a file from one location to another.

Let’s create an input file first, called _input\_file.txt_

```
ruby -e '1_000_000.times { puts rand.to_s }' > input_file.txt
```

Now, I’m going to write a short Ruby script called _test\_filecopy.rb_. I have taken the meat of this script from [Iñaki Baz Castillo](https://dev.sipdoc.net)‘s benchmark code. 

```
#!/usr/bin/ruby

require 'io/splice'

SRC_FILE = "input_file.txt"
DST_FILE = "output_file.txt"

source = File.open(SRC_FILE, 'rb')
dest = File.open(DST_FILE, 'wb')
source_fd = source.fileno
dest_fd = dest.fileno

# We use a pipe as a ring buffer in kernel space.
# pipes may store up to IO::Splice::PIPE_CAPA bytes
pipe = IO.pipe
rfd, wfd = pipe.map { |io| io.fileno }

while true

  nread = begin
    # first pull as many bytes as possible into the pipe
    IO.splice(source_fd, nil, wfd, nil, IO::Splice::PIPE_CAPA, 0)
  rescue EOFError
    # The end of the file has been reached
    break
  end

  # now move the contents of the pipe buffer into the destination file
  # the copied data never enters userspace
  nwritten = IO.splice(rfd, nil, dest_fd, nil, nread, 0)

end

```

Sure enough, it copied the file. Although, if you were paying attention, you will realize that even though we did not read the data into user-space memory, we still copied it into an intermediary pipe in kernel-space memory. We also had an extra context switch back to our code between the reading from one file and writing to the other. This is because we cannot use zero-copy to transfer directly from one file-descriptor to another file-descriptor without one of the end-points being a pipe. To get around this we created the in-kernel-memory pipe as an intermediary.

## Conclusion

Zero-copy is way to increase performance when transferring static data between two end-points on your system. A good use-case for this is serving files to the web. Even though the disk is still a major bottle-neck, benchmarks do show it to be faster. If validating the data going out of your system is important to you then zero-copy may not be the solution you are looking for, as the data never enters user-space, so there is no chance to inspect it.

## Resources

*   [Zero-Copy On Wikipedia](https://en.wikipedia.org/wiki/Zero-copy)
*   [Zero Copy I: User-Mode Perspective](https://www.linuxjournal.com/article/6345)
*   [“Ruby IO Splice” Documentation](https://bogomips.org/ruby_io_splice/IO.html)
*   [Splice man-page](https://www.kernel.org/doc/man-pages/online/pages/man2/splice.2.html)
*   [Efficient data transfer through zero copy](https://www.ibm.com/developerworks/library/j-zerocopy/index.html)
*   [Ruby IO Splice Homepage](https://bogomips.org/ruby_io_splice/)
*   [How to create PI sequentially in Ruby](https://stackoverflow.com/questions/3137594/how-to-create-pi-sequentially-in-ruby)

<div id="comments">
  <h3 id="comments-number" class="comments-header">2 responses to “Zero-Copy. Transfer Data Faster In Ruby”</h3>
  <ol class="comment-list">
    <li id="comment-807" class="comment even thread-even depth-1 comment reader">
      <img alt="iq9" src="https://0.gravatar.com/avatar/092ff97a9cf1189c671baecc59f8bf65?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://russbrooks.com">iq9</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, January 20th, 2011, 8:08 am">January 20, 2011</abbr> at <abbr class="comment-time" title="Thursday, January 20th, 2011, 8:08 am">8:08 am</abbr>
      </div>
      <div class="comment-text">
        <p>It would be interesting to see straight calls to `cp` in their benchmarks. https://librelist.com/browser//ruby.io.splice/2010/12/22/some-benchmarks/</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-813" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Carl Youngblood" src="https://0.gravatar.com/avatar/eca15b2b601e7e577d38bd5210a753ac?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Carl Youngblood</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, January 20th, 2011, 11:51 pm">January 20, 2011</abbr> at <abbr class="comment-time" title="Thursday, January 20th, 2011, 11:51 pm">11:51 pm</abbr>
      </div>
      <div class="comment-text">
        <p>I wonder if someone more familiar with OSX is aware of another construct in BSD land for doing the same thing. Any takers?</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

