---
title: "Map-Reduce With Ruby Using Hadoop"
date: "2010-12-31T09:38:00-08:00"
template: "post"
draft: false
slug: "map-reduce-with-ruby-using-hadoop"
category: "Data processing"
tags:
  - amazon ec2
  - bash
  - cloudera
  - data processing
  - hadoop
  - hadoop cluster
  - hadoop streaming
  - hdfs
  - jclouds
  - map-reduce
  - ruby
  - whirr
description: "Here I demonstrate, with repeatable steps, how to fire-up a Hadoop cluster on Amazon EC2, load data onto the HDFS (Hadoop Distributed File-System), write map-reduce scripts in Ruby and use them to run a map-reduce job on your Hadoop cluster. You will not need to ssh into the cluster, as all tasks are run from your local machine. Below I am using my MacBook Pro as my local machine, but the steps I have provided should be reproducible on other platforms running bash and Java."
socialImage:
  publicURL: "/media/images/2010/12/hadoop_ruby_sq.jpg"
---
<a href="/map-reduce-with-ruby-using-hadoop/hadoop-ruby" rel="attachment wp-att-605">

<img alt="Map-Reduce With Hadoop Using Ruby" class="alignleft size-full wp-image-605" height="189" src="/media/images/2010/12/hadoop-ruby.png" style="border: none" title="hadoop-ruby" width="202"/>

</a>  
Here I demonstrate, with repeatable steps, how to fire-up a Hadoop cluster on Amazon EC2, load data onto the HDFS (Hadoop Distributed File-System), write map-reduce scripts in Ruby and use them to run a map-reduce job on your Hadoop cluster. You will _not_ need to ssh into the cluster, as all tasks are run from your local machine. Below I am using my MacBook Pro as my local machine, but the steps I have provided should be reproducible on other platforms running bash and Java.

*   [Fire-Up Your Hadoop Cluster](/map-reduce-with-ruby-using-hadoop#fire-up-your-hadoop-cluster)
*   [Setting Up Your Local Hadoop Client](/map-reduce-with-ruby-using-hadoop#setting-up-your-local-hadoop-client)
*   [Defining The Map-Reduce Task](/map-reduce-with-ruby-using-hadoop#defining-the-map-reduce-task)
*   [Uploading Your Data To HDFS (Hadoop Distributed FileSystem)](/map-reduce-with-ruby-using-hadoop#uploading-your-data-to-hdfs)
*   [Coding Your Map And Reduce Scripts in Ruby](/map-reduce-with-ruby-using-hadoop#coding-your-map-and-reduce-scripts-in-ruby)
*   [Running The Hadoop Job](/map-reduce-with-ruby-using-hadoop#running-the-hadoop-job)
*   [The Results](/map-reduce-with-ruby-using-hadoop#the-results)
*   [Conclusion](/map-reduce-with-ruby-using-hadoop#conclusion)
*   [Resources](/map-reduce-with-ruby-using-hadoop#resources)

<a id="fire-up-your-hadoop-cluster" name="fire-up-your-hadoop-cluster"></a>

## Fire-Up Your Hadoop Cluster

I choose the [Cloudera distribution of Hadoop](https://www.cloudera.com/hadoop/) which is still 100% Apache licensed, but has some additional benefits. One of these benefits is that it is released by [Doug Cutting](https://en.wikipedia.org/wiki/Doug_Cutting), who started Hadoop and drove it’s development at Yahoo! He also started [Lucene](https://lucene.apache.org/), which is another of my favourite Apache Projects, so I have good faith that he knows what he is doing. Another benefit, as you will see, is that it is simple to fire-up a Hadoop cluster.

I am going to use Cloudera’s [Whirr script](https://archive.cloudera.com/cdh/3/whirr/), which will allow me to fire up a production ready Hadoop cluster on [Amazon EC2](https://aws.amazon.com/ec2/) directly from my laptop. Whirr is built on [jclouds](https://code.google.com/p/jclouds/), meaning other cloud providers should be supported, but only Amazon EC2 has been tested. Once we have Whirr installed, we will configure a _hadoop.properties_ file with our Amazon EC2 credentials and the details of our desired Hadoop cluster. Whirr will use this _hadoop.properties_ file to build the cluster.

If you are on Debian or Redhat you can use either apt-get or yum to install whirr, but since I’m on Mac OS X, I’ll need to [download the Whirr script](https://www.apache.org/dyn/closer.cgi/incubator/whirr/).

The current version of Whirr 0.2.0, hosted on the Apache Incubator site, is not compatible with Cloudera’s Distribution for Hadoop (CDH), so I’m am downloading [version 0.1.0+23](https://archive.cloudera.com/cdh/3/whirr-0.1.0+23.tar.gz).

```
mkdir ~/src/cloudera
cd ~/src/cloudera
wget https://archive.cloudera.com/cdh/3/whirr-0.1.0+23.tar.gz
tar -xvzf whirr-0.1.0+23.tar.gz
```

To build Whirr you’ll need to install Java (version 1.6), [Maven](https://maven.apache.org/download.html) ( &gt;= 2.2.1) and [Ruby](https://www.ruby-lang.org/en/) ( &gt;= 1.8.7). If you’re running with the latest Mac OS X, then you should have the latest Java and I’ll assume, due to the title of this post, that you can manage the Ruby version. If you are not familiar with Maven, you can [install it via Homebrew on Mac OS X](/homebrew-intro-to-the-mac-os-x-package-installer) using the brew command below. On Debian use _apt-get install maven2_.

```
sudo brew update
sudo brew install maven
```

Once the dependencies are installed we can build the whirr tool.

```
cd whirr-0.1.0+23
mvn clean install
mvn package -Ppackage
```

In true Maven style, it will download a long list of dependencies the first time you build this. Be patient.

Ok, it should be built now and if you’re anything like me, you would have used the time to get a nice cuppa tea or a sandwich. Let’s sanity check the whirr script…

```
bin/whirr version
```

You should see something like “Apache Whirr 0.1.0+23″ output to the terminal.

Create a _hadoop.properties_ file with the following content.

```
whirr.service-name=hadoop
whirr.cluster-name=myhadoopcluster
whirr.instance-templates=1 jt+nn,1 dn+tt
whirr.provider=ec2
whirr.identity=<cloud-provider-identity>
whirr.credential=<cloud-provider-credential>
whirr.private-key-file=${sys:user.home}/.ssh/id_rsa
whirr.public-key-file=${sys:user.home}/.ssh/id_rsa.pub
whirr.hadoop-install-runurl=cloudera/cdh/install
whirr.hadoop-configure-runurl=cloudera/cdh/post-configure
```

Replace _&lt;cloud-provider-identity&gt;_ and _&lt;cloud-provider-credential&gt;_ with your Amazon EC2 Access Key ID and Amazon EC2 Secret Access Key (I will not tell you what mine is).

This configuration is a little boring with only two machines. One machine for the master and one machine for the worker. You can get more creative once you are up and running. Let’s fire up our “cluster”.

```
bin/whirr launch-cluster --config hadoop.properties
```

This is another good time to put the kettle on, as it takes a few minutes to get up and running. If you are curious, or worried that things have come to a halt then Whirr outputs a whirr.log in the current directory. Fire-up another terminal window and tail the log.

```
cd ~/src/cloudera/whirr-0.1.0+23
tail -F whirr.log
```

16 minutes (and several cups of tea) later the cluster is up and running. Here is the output I saw in my terminal.

```
Launching myhadoopcluster cluster
Configuring template
Starting master node
Master node started: [[id=us-east-1/i-561d073b, providerId=i-561d073b, tag=myhadoopcluster, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-d59d6bbc, os=[name=null, family=amzn-linux, version=2010.11.1-beta, arch=paravirtual, is64Bit=false, description=amzn-ami-us-east-1/amzn-ami-2010.11.1-beta.i386.manifest.xml], userMetadata={}, state=RUNNING, privateAddresses=[10.113.23.123], publicAddresses=[72.44.45.199], hardware=[id=m1.small, providerId=m1.small, name=m1.small, processors=[[cores=1.0, speed=1.0]], ram=1740, volumes=[[id=null, type=LOCAL, size=10.0, device=/dev/sda1, durable=false, isBootDevice=true], [id=null, type=LOCAL, size=150.0, device=/dev/sda2, durable=false, isBootDevice=false]], supportsImage=Not(is64Bit())]]]
Authorizing firewall
Starting 1 worker node(s)
Worker nodes started: [[id=us-east-1/i-98100af5, providerId=i-98100af5, tag=myhadoopcluster, name=null, location=[id=us-east-1d, scope=ZONE, description=us-east-1d, parent=us-east-1], uri=null, imageId=us-east-1/ami-d59d6bbc, os=[name=null, family=amzn-linux, version=2010.11.1-beta, arch=paravirtual, is64Bit=false, description=amzn-ami-us-east-1/amzn-ami-2010.11.1-beta.i386.manifest.xml], userMetadata={}, state=RUNNING, privateAddresses=[10.116.147.148], publicAddresses=[184.72.179.36], hardware=[id=m1.small, providerId=m1.small, name=m1.small, processors=[[cores=1.0, speed=1.0]], ram=1740, volumes=[[id=null, type=LOCAL, size=10.0, device=/dev/sda1, durable=false, isBootDevice=true], [id=null, type=LOCAL, size=150.0, device=/dev/sda2, durable=false, isBootDevice=false]], supportsImage=Not(is64Bit())]]]
Completed launch of myhadoopcluster
Web UI available at https://ec2-72-44-45-199.compute-1.amazonaws.com
Wrote Hadoop site file /Users/phil/.whirr/myhadoopcluster/hadoop-site.xml
Wrote Hadoop proxy script /Users/phil/.whirr/myhadoopcluster/hadoop-proxy.sh
Started cluster of 2 instances
HadoopCluster{instances=[Instance{roles=[jt, nn], publicAddress=ec2-72-44-45-199.compute-1.amazonaws.com/72.44.45.199, privateAddress=/10.113.23.123}, Instance{roles=[tt, dn], publicAddress=/184.72.179.36, privateAddress=/10.116.147.148}], configuration={fs.default.name=hdfs://ec2-72-44-45-199.compute-1.amazonaws.com:8020/, mapred.job.tracker=ec2-72-44-45-199.compute-1.amazonaws.com:8021, hadoop.job.ugi=root,root, hadoop.rpc.socket.factory.class.default=org.apache.hadoop.net.SocksSocketFactory, hadoop.socks.server=localhost:6666}}
```

Whirr has created a directory with some files in our home directory…

```
~/.whirr/myhadoopcluster/hadoop-proxy.sh
~/.whirr/myhadoopcluster/hadoop-site.xml
```

This hadoop-proxy.sh is used to access the web interface of Hadoop securely. When we run this it will tunnel through to the cluster and give us access in the web browser via a SOCKS proxy.

You need to configure the SOCKS proxy in either your web browser or, in my case, the Mac OS X settings menu.

<div class="wp-caption alignnone" id="attachment_508" style="width: 720px">
<a href="/map-reduce-with-ruby-using-hadoop/screen-shot-2010-12-28-at-3-15-55-pm" rel="attachment wp-att-508">
<img alt="Hadoop SOCKS Proxy Configuration for Mac OS X" class="size-full wp-image-508" height="360" src="/media/images/2010/12/Screen-shot-2010-12-28-at-3.15.55-PM.png" title="SOCKS Proxy Configuration" width="710"/>
</a>
<p class="wp-caption-text">Hadoop SOCKS Proxy Configuration for Mac OS X</p>
</div>

Now start the proxy in your terminal…  
_(Note: There has still been no need to ssh into the cluster. Everything in this post is done on our local machine)_

```
sh ~/.whirr/myhadoopcluster/hadoop-proxy.sh

   Running proxy to Hadoop cluster at
   ec2-72-44-45-199.compute-1.amazonaws.com.
   Use Ctrl-c to quit.

```

The above will output the hostname that you can access the cluster at. On Amazon EC2 it looks something like _https://ec2-72-44-45-199.compute-1.amazonaws.com:50070/dfshealth.jsp_. Use this hostname to view the cluster in your web browser.

```
https://<hostname>:50070/dfshealth.jsp
```

<div class="wp-caption alignnone" id="attachment_502" style="width: 588px">
<a href="/map-reduce-with-ruby-using-hadoop/screen-shot-2010-12-28-at-3-50-23-pm" rel="attachment wp-att-502">
<img alt="dfshealth.jsp" class="size-full wp-image-502" height="643" src="/media/images/2010/12/Screen-shot-2010-12-28-at-3.50.23-PM.png" title="Screen shot 2010-12-28 at 3.50.23 PM" width="578"/>
</a>
<p class="wp-caption-text">HDFS Health Dashboard</p>
</div>

If you click on the link to “Browse the filesystem” then you will notice the hostname changes. This will jump around the data-nodes in your cluster, due to HDFS’s distributed nature. You only currently have one data-node. On Amazon EC2 this new hostname will be the internal hostname of data-node server, which is visible because you are tunnelling through the SOCKS proxy.

<div class="wp-caption alignnone" id="attachment_505" style="width: 670px">
<a href="/map-reduce-with-ruby-using-hadoop/screen-shot-2010-12-28-at-3-36-08-pm" rel="attachment wp-att-505">
<img alt="browseDirectory.jsp" class="size-full wp-image-505" height="391" src="/media/images/2010/12/Screen-shot-2010-12-28-at-3.36.08-PM.png" title="Screen shot 2010-12-28 at 3.36.08 PM" width="660"/>
</a>
<p class="wp-caption-text">HDFS File Browser</p>
</div>

Ok! It looks as though our Hadoop cluster is up and running. Let’s upload our data.

<a id="setting-up-your-local-hadoop-client" name="setting-up-your-local-hadoop-client"></a>

## Setting Up Your Local Hadoop Client

To run a map-reduce job on your data, your data needs to be on the Hadoop Distributed File-System. Otherwise known as HDFS. You can interact with Hadoop and HDFS with the _hadoop_ command. We do not have Hadoop installed on our local machine. Therefore, we can either log into one of our Hadoop cluster machines and run the hadoop command from there, or install hadoop on our local machine. I’m going to opt for installing Hadoop on my local machine (recommended), as it will be easier to interact with the HDFS and start the Hadoop map-reduce jobs directly from my laptop.

Cloudera does not, unfortunately, provide a release of Hadoop for Mac OS X. Only debians and RPMs. They do provide a .tar.gz download, which we are going to use to install Hadoop locally. Hadoop is built with Java and the scripts are written in bash, so there should not be too many problems with compatibility across platforms that can run Java and bash.

Visit [Cloudera CDH Release](https://docs.cloudera.com/display/DOC/Downloading+CDH+Releases) webpage and select [CDH3 Patched Tarball](https://archive.cloudera.com/cdh/3/). I downloaded the same version [hadoop-0.20.2+737.tar.gz](https://archive.cloudera.com/cdh/3/hadoop-0.20.2+737.tar.gz ) that Whirr installed on the cluster.

```
tar -xvzf hadoop-0.20.2+737.tar.gz
sudo mv hadoop-0.20.2+737 /usr/local/
cd /usr/local
sudo ln -s hadoop-0.20.2+737 hadoop
echo 'export HADOOP_HOME=/usr/local/hadoop' >> ~/.profile
echo 'export PATH=$PATH:$HADOOP_HOME/bin' >> ~/.profile
source ~/.profile
which hadoop # should output "/usr/local/hadoop/bin/hadoop"
hadoop version # should output "Hadoop 0.20.2+737 ..."
cp ~/.whirr/myhadoopcluster/hadoop-site.xml /usr/local/hadoop/conf/
```

Now run your first command from your local machine to interact with HDFS. This following command is similar to “ls -l /” in bash.

```
hadoop fs -ls /
```

You should see the following output which lists the root on the Hadoop filesystem. 

```
10/12/30 18:19:59 WARN conf.Configuration: DEPRECATED: hadoop-site.xml found in the classpath. Usage of hadoop-site.xml is deprecated. Instead use core-site.xml, mapred-site.xml and hdfs-site.xml to override properties of core-default.xml, mapred-default.xml and hdfs-default.xml respectively
Found 4 items
drwxrwxrwx   - hdfs supergroup          0 2010-12-28 10:33 /hadoop
drwxrwxrwx   - hdfs supergroup          0 2010-12-28 10:33 /mnt
drwxrwxrwx   - hdfs supergroup          0 2010-12-28 10:33 /tmp
drwxrwxrwx   - hdfs supergroup          0 2010-12-28 10:33 /user
```

Yes, you will see a depreciation warning, since hadoop-site.xml configuration has been split into multiple files. We will not worry about this here.

<a id="defining-the-map-reduce-task" name="defining-the-map-reduce-task"></a>

## Defining The Map-Reduce Task

We are going write a map-reduce job that scans all the files in a given directory, takes the words found in those files and then counts the number of times words begin with any two characters.

For this we’re going to use a dictionary file found on my Mac OS X /usr/share/dict/words. It contains 234936 words, each on a newline. [Linux has a similar dictionary file](https://en.wikipedia.org/wiki/Words_(Unix)).

<a id="uploading-your-data-to-hdfs" name="uploading-your-data-to-hdfs"></a>

## Uploading Your Data To HDFS (Hadoop Distributed FileSystem)

```
hadoop fs -mkdir input
hadoop fs -put /usr/share/dict/words input/
hadoop fs -ls input
```

You should see output similar to the following, which list the _words_ file on the remote HDFS. Since my local user is “phil”, Hadoop has added the file under /user/phil on HDFS.

```
Found 1 items
-rw-r--r--   3 phil supergroup    2486813 2010-12-30 18:43 /user/phil/input/words
```

Congratulations! You have just uploaded your first file to the Hadoop Distributed File-System on your cluster in the cloud.

<a id="coding-your-map-and-reduce-scripts-in-ruby" name="coding-your-map-and-reduce-scripts-in-ruby"></a>

## Coding Your Map And Reduce Scripts in Ruby

Map-Reduce can actually be thought of as map-group-reduce. The “map” sucks in the raw data, cuts off the fat, removes the bones and outputs the smallest possible piece of output data for each piece of input data. The “map” also outputs the key of the data. Our key will be the two-letter prefix of each word. These keys are used by Hadoop to “group” the data together. The “reduce” then takes each group of data and “reduces” it. In our case the “reduce” will be the counting occurrences of the two-letter prefixes.

Hadoop will do much of the work for us. It will recurse the input directory, open the files and stream the files one line at a time into our “map” script via STDIN. We will output zero, one or many output lines to STDOUT for each line of input. Since we know that our input file has exactly one word per line, we can simplify our script and always output exactly one two-letter prefix for each input line. (EDIT: words with one letter will not result in any output).

The output of our “map” script to STDOUT will have to be Hadoop friendly. This means we will output our “key”, then a tab character then our value and then a newline. This is what the streaming interface expects. Hadoop needs to extract the key to be able to sort and organise the data based on this key.

```
<key><tab><value><newline>
```

Our value will always be “1″, since each line has only one word with only once instance of the two-letter prefix of that word.

For instance, if the input was “Apple” then we would output the key “ap” and value “1″. We have seen the prefix “ap” only once in this input.

You should note that the value can be anything that your reduce script can interpret. For instance, the value could be a string of JSON. Here, we are keeping it very simple.

```
ap<tab>1<newline>
```

Let’s code up the mapper as _map.rb_

```
# Ruby code for map.rb

ARGF.each do |line|

   # remove any newline
   line = line.chomp

   # do nothing will lines shorter than 2 characters
   next if ! line || line.length < 2

   # grab our key as the two-character prefix (lower-cased)
   key = line[0,2].downcase

   # value is a count of 1 occurence
   value = 1

   # output to STDOUT
   # <key><tab><value><newline>
   puts key + "\t" + value.to_s

end
```

Now we have our mapper script, let’s write the reducer.

Remember, the reducer is going to count up the occurences for each two-character prefix (our “key”). Hadoop will have already grouped our keys together, so even if the mapper output is in shuffled order, the reducer will now see the keys in sorted order. This means that the reducer can watch for when the key changes and know that it has seen all of the possible values for the previous key.

Here is an example of the STDIN and STDOUT that map.rb and reduce.rb might see. The data flow goes from left to right.

<table>
<tr>
<th>map.rb<br/>STDIN</th>
<th>map.rb<br/>STDOUT</th>
<th>Hadoop<br/>sorts<br/>keys</th>
<th>reduce.rb<br/>STDIN</th>
<th>reduce.rb<br/>STDOUT</th>
</tr>
<tr>
<td>Apple<br/>Monkey<br/>Orange<br/>Banana<br/>APR<br/>Bat<br/>appetite</td>
<td>ap 1<br/>mo 1<br/>or 1<br/>ba 1<br/>ap 1<br/>ba 1<br/>ap 1</td>
<td></td>
<td>ap 1<br/>ap 1<br/>ap 1<br/>ba 1<br/>ba 1<br/>mo 1<br/>or 1</td>
<td>ap 3<br/>ba 2<br/>mo 1<br/>or 1</td>
</tr>
</table>

Let’s code up the reducer as _reduce.rb_

```
# Ruby code for reduce.rb

prev_key = nil
key_total = 0

ARGF.each do |line|

   # remove any newline
   line = line.chomp

   # split key and value on tab character
   (key, value) = line.split(/\t/)

   # check for new key
   if prev_key && key != prev_key && key_total > 0

      # output total for previous key

      # <key><tab><value><newline>
      puts prev_key + "\t" + key_total.to_s

      # reset key total for new key
      prev_key = key
      key_total = 0

   elsif ! prev_key
      prev_key = key

   end

   # add to count for this current key
   key_total += value.to_i

end
```

You can test out your scripts on a small sample by using the “sort” command in replacement for Hadoop.

```
cat /usr/share/dict/words | ruby map.rb | sort | ruby reduce.rb
```

The start of this output looks like this…

```
aa  13
ab  666
ac  1491
ad  867
ae  337
af  380
```

<a id="running-the-hadoop-job" name="running-the-hadoop-job"></a>

## Running The Hadoop Job

I wrote this bash-based runner script to start the job. It uses Hadoop’s streaming service. This streaming service is what allows us to write our map-reduce scripts in Ruby. It _streams_ to our script’s STDIN and reads our script’s output from our script’s STDOUT.

```
#!/bin/bash

HADOOP_HOME=/usr/local/hadoop
JAR=contrib/streaming/hadoop-streaming-0.20.2+737.jar

HSTREAMING="$HADOOP_HOME/bin/hadoop jar $HADOOP_HOME/$JAR"

$HSTREAMING \
 -mapper  'ruby map.rb' \
 -reducer 'ruby reduce.rb' \
 -file map.rb \
 -file reduce.rb \
 -input '/user/phil/input/*' \
 -output /user/phil/output
```

We specify the command to run for the mapper and reducer and use the “-file” parameter twice to attach our two Ruby scripts. It is assumed that all other dependencies are already installed on the machine. In this case we are using no Ruby imports or requires and the Ruby interpreter is already installed on the machines in the Hadoop cluster (it came with the Cloudera Amazon EC2 image). Things become more complicated when you start to run jobs with more dependencies that are not already installed on the Hadoop cluster. This is a topic for another post.

“-input” and “-output” specify which files to read from for input and the directoty to send the output to. You can also specify a deeper level of recursion with more wildcards (e.g. “/user/phil/input/\*/\*/\*”).

Once again, it is important that our SOCKS proxy is running, as this is the secure way that we communicate through to our Hadoop cluster.

```
sh ~/.whirr/myhadoopcluster/hadoop-proxy.sh
    Running proxy to Hadoop cluster at ec2-72-44-45-199.compute-1.amazonaws.com. Use Ctrl-c to quit.
```

Now we can start the Hadoop job by running our above bash script. Here is the output the script gave me at the terminal.

```
packageJobJar: [map.rb, reduce.rb, /tmp/hadoop-phil/hadoop-unjar3366245269477540365/] [] /var/folders/+Q/+QReZ-KsElyb+mXn12xTxU+++TI/-Tmp-/streamjob5253225231988397348.jar tmpDir=null
10/12/30 21:45:32 INFO mapred.FileInputFormat: Total input paths to process : 1
10/12/30 21:45:37 INFO streaming.StreamJob: getLocalDirs(): [/tmp/hadoop-phil/mapred/local]
10/12/30 21:45:37 INFO streaming.StreamJob: Running job: job_201012281833_0001
10/12/30 21:45:37 INFO streaming.StreamJob: To kill this job, run:
10/12/30 21:45:37 INFO streaming.StreamJob: /usr/local/hadoop/bin/hadoop job  -Dmapred.job.tracker=ec2-72-44-45-199.compute-1.amazonaws.com:8021 -kill job_201012281833_0001
10/12/30 21:45:37 INFO streaming.StreamJob: Tracking URL: https://ec2-72-44-45-199.compute-1.amazonaws.com:50030/jobdetails.jsp?jobid=job_201012281833_0001
10/12/30 21:45:38 INFO streaming.StreamJob:  map 0%  reduce 0%
10/12/30 21:45:55 INFO streaming.StreamJob:  map 42%  reduce 0%
10/12/30 21:45:58 INFO streaming.StreamJob:  map 100%  reduce 0%
10/12/30 21:46:14 INFO streaming.StreamJob:  map 100%  reduce 88%
10/12/30 21:46:19 INFO streaming.StreamJob:  map 100%  reduce 100%
10/12/30 21:46:22 INFO streaming.StreamJob: Job complete: job_201012281833_0001
10/12/30 21:46:22 INFO streaming.StreamJob: Output: /user/phil/output
```

This is reflected if you visit the job tracker console in web browser.

<div class="wp-caption alignnone" id="attachment_577" style="width: 996px">
<a href="/map-reduce-with-ruby-using-hadoop/screen-shot-2010-12-30-at-10-12-46-pm" rel="attachment wp-att-577">
<img alt="jobTracker after successful run" class="size-full wp-image-577" height="783" src="/media/images/2010/12/Screen-shot-2010-12-30-at-10.12.46-PM.png" title="Screen shot 2010-12-30 at 10.12.46 PM" width="986"/>
</a>
<p class="wp-caption-text">jobTracker after successful run</p>
</div>

If you click on the job link you can see lots of information on this job. This job is completed in these images, but with a longer running job you would see the progress as the job runs. I have split the job tracker page into the following three images.

<div class="wp-caption alignnone" id="attachment_578" style="width: 762px">
<a href="/map-reduce-with-ruby-using-hadoop/screen-shot-2010-12-30-at-10-15-55-pm" rel="attachment wp-att-578">
<img alt="Map-Reduce Job Tracker Page (part 1)" class="size-full wp-image-578" height="361" src="/media/images/2010/12/Screen-shot-2010-12-30-at-10.15.55-PM.png" title="Screen shot 2010-12-30 at 10.15.55 PM" width="752"/>
</a>
<p class="wp-caption-text">Map-Reduce Job Tracker Page (part 1)</p>
</div>

<div class="wp-caption alignnone" id="attachment_579" style="width: 720px">
<a href="/map-reduce-with-ruby-using-hadoop/screen-shot-2010-12-30-at-10-16-17-pm" rel="attachment wp-att-579">
<img alt="Map-Reduce Job Tracker Page (part 2)" class="size-full wp-image-579" height="616" src="/media/images/2010/12/Screen-shot-2010-12-30-at-10.16.17-PM.png" title="Screen shot 2010-12-30 at 10.16.17 PM" width="710"/>
</a>
<p class="wp-caption-text">Map-Reduce Job Tracker Page (part 2)</p>
</div>

<div class="wp-caption alignnone" id="attachment_580" style="width: 772px">
<a href="/map-reduce-with-ruby-using-hadoop/screen-shot-2010-12-30-at-10-16-44-pm" rel="attachment wp-att-580">
<img alt="Map-Reduce Job Tracker Page (part 3) Graphs" class="size-full wp-image-580" height="551" src="/media/images/2010/12/Screen-shot-2010-12-30-at-10.16.44-PM.png" title="Screen shot 2010-12-30 at 10.16.44 PM" width="762"/>
</a>
<p class="wp-caption-text">Map-Reduce Job Tracker Page (part 3) Graphs</p>
</div>

<a id="the-results" name="the-results"></a>

## The Results

Our map-reduce job has run successfully using Ruby. Let’s have a look at the output.

```
hadoop fs -ls output

Found 3 items
-rw-r--r--   3 phil supergroup          0 2010-12-30 21:46 /user/phil/output/_SUCCESS
drwxrwxrwx   - phil supergroup          0 2010-12-30 21:45 /user/phil/output/_logs
-rw-r--r--   3 phil supergroup       2341 2010-12-30 21:46 /user/phil/output/part-00000
```

Hadoop output is written in chunks to sequential files part-00000, part-00001, part-00002 and so on. Our dataset is very small, so we only have one 2kb file called part-00000.

```
hadoop fs -cat output/part-00000 | head
aa  13
ab  666
ac  1491
ad  867
ae  337
af  380
ag  507
ah  46
ai  169
aj  14
```

Our map-reduce script counted 13 words starting with “aa”, 666 words starting with “ab” and 1491 words starting with “ac”.

<a id="conclusion" name="conclusion"></a>

## Conclusion

Yes, it is an overkill to use Hadoop and a (very small) cluster of cloud-based machines for this example, but I think it demonstrates how you can quickly get your Hadoop cluster up and running map-reduce jobs written in Ruby. You can use the same procedure to fire-up a much larger and more powerful Hadoop cluster with a bigger dataset and more complex Ruby scripts.

__Please post any questions or suggestions you have in the comments below. They are always highly appreciated.__

<a id="resources" name="resources"></a>

## Resources

*   [Apache Hadoop](https://hadoop.apache.org/)
*   [Cloudera’s Distribution for Apache Hadoop (CDH)](https://www.cloudera.com/hadoop/)
*   [Cloudera Hadoop Training VMWare Image](https://www.cloudera.com/downloads/virtual-machine/)
*   [Map-Reduce Using Perl](https://www.slideshare.net/philwhln/map-reduce-using-perl)
*   [jclouds ](https://code.google.com/p/jclouds/)
*   [Words file on Unix-like operating systems](https://en.wikipedia.org/wiki/Words_(Unix))
*   [Ruby Weekly](https://rubyweekly.com/)

## Posted in : [ Ruby](/category/ruby-2)

## Related Books

If you are interested in learning more about Hadoop, then I recommend reading  
[Hadoop: The Definitive Guide (2nd Edition)](https://www.amazon.com/gp/product/1449389732?ie=UTF8&amp;tag=getafil-20&amp;linkCode=as2&amp;camp=1789&amp;creative=9325&amp;creativeASIN=1449389732)

<img alt="" border="0" height="1" src="https://www.assoc-amazon.com/e/ir?t=getafil-20&amp;l=as2&amp;o=1&amp;a=1449389732" style="border:none !important; margin:0px !important;" width="1"/>

  
by Tom White.

<a href="https://www.amazon.com/gp/product/1449389732?ie=UTF8&amp;tag=getafil-20&amp;linkCode=as2&amp;camp=1789&amp;creative=9325&amp;creativeASIN=1449389732"><img border="0" src="https://images-na.ssl-images-amazon.com/media/images/I/41VfwSSarkL._SL160_.jpg"/></a>

<img alt="" border="0" height="1" src="https://www.assoc-amazon.com/e/ir?t=getafil-20&amp;l=as2&amp;o=1&amp;a=1449389732" style="border:none !important; margin:0px !important;" width="1"/>

Here are some other good books related to this blog post… 

<object classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" codebase="https://fpdownload.macromedia.com/get/flashplayer/current/swflash.cab" height="200px" id="Player_2ddaa026-5f1d-4b7b-a5bc-7d610d9369dc" width="600px"> 

<param name="movie" value="https://ws.amazon.com/widgets/q?ServiceVersion=20070822&amp;MarketPlace=US&amp;ID=V20070822%2FUS%2Fgetafil-20%2F8010%2F2ddaa026-5f1d-4b7b-a5bc-7d610d9369dc&amp;Operation=GetDisplayTemplate"/>

<param name="quality" value="high"/>

<param name="bgcolor" value="#FFFFFF"/>

<param name="allowscriptaccess" value="always"/>

<embed align="middle" allowscriptaccess="always" bgcolor="#ffffff" height="200px" id="Player_2ddaa026-5f1d-4b7b-a5bc-7d610d9369dc" name="Player_2ddaa026-5f1d-4b7b-a5bc-7d610d9369dc" quality="high" src="https://ws.amazon.com/widgets/q?ServiceVersion=20070822&amp;MarketPlace=US&amp;ID=V20070822%2FUS%2Fgetafil-20%2F8010%2F2ddaa026-5f1d-4b7b-a5bc-7d610d9369dc&amp;Operation=GetDisplayTemplate" type="application/x-shockwave-flash" width="600px"/>

</object> 

<noscript><a href="https://ws.amazon.com/widgets/q?ServiceVersion=20070822&amp;MarketPlace=US&amp;ID=V20070822%2FUS%2Fgetafil-20%2F8010%2F2ddaa026-5f1d-4b7b-a5bc-7d610d9369dc&amp;Operation=NoScript">Amazon.com Widgets</a></noscript>

<div class="hentry post publish post-1 odd author-admin category-cassandra-nosql-2 category-web-development post_tag-amazon-ec2 post_tag-cassandra post_tag-cluster post_tag-homebrew post_tag-mac-os-x post_tag-nosql post_tag-whirr" id="post-1051">
<h2 class="entry-title"><a href="/quickly-launch-a-cassandra-cluster-on-amazon-ec2" rel="bookmark" title="Quickly Launch A Cassandra Cluster On Amazon EC2">Quickly Launch A Cassandra Cluster On Amazon EC2</a></h2>
<p class="byline"><span class="byline-prep byline-prep-author">By</span> <span class="author vcard"><span class="fn">Phil Whelan</span></span> <span class="byline-prep byline-prep-published">on</span> <abbr class="published" title="Friday, January 21st, 2011, 6:58 pm">January 21, 2011</abbr></p>
<div style="float: left;"><img alt="launch_cassandra_amazon_ec2_sq" class="attachment-post-thumbnail wp-post-image" height="180" src="/media/images/2011/01/launch_cassandra_amazon_ec2_sq.jpg" title="launch_cassandra_amazon_ec2_sq" width="180"/></div>
<div class="entry-content">
<p>If you have read my previous post, “Map-Reduce With Ruby Using Hadoop“, then you will know that firing up a Hadoop cluster is really simple when you use Whirr. Without even ssh’ing on the machines in the cloud you can start-up your cluster and interact with it. In this post I’ll show you that it [...]</p>
</div>
<p><!-- .entry-content --></p>
<div class="continue-reading"><a href="/quickly-launch-a-cassandra-cluster-on-amazon-ec2"><strong>Continue reading…</strong></a></div>
</div>

If you have read my previous post, “Map-Reduce With Ruby Using Hadoop“, then you will know that firing up a Hadoop cluster is really simple when you use Whirr. Without even ssh’ing on the machines in the cloud you can start-up your cluster and interact with it. In this post I’ll show you that it \[...\]

<div class="hentry post publish post-3 odd author-admin category-ruby-2 category-web-development post_tag-file-server post_tag-gem post_tag-io post_tag-iosplice post_tag-java post_tag-linux post_tag-ruby post_tag-splice post_tag-zero-copy" id="post-836" style="padding-top: 40px;">
<h2 class="entry-title"><a href="/zero-copy-transfer-data-faster-in-ruby" rel="bookmark" title="Zero-Copy. Transfer Data Faster In Ruby">Zero-Copy. Transfer Data Faster In Ruby</a></h2>
<p class="byline"><span class="byline-prep byline-prep-author">By</span> <span class="author vcard"><span class="fn">Phil Whelan</span></span> <span class="byline-prep byline-prep-published">on</span> <abbr class="published" title="Friday, January 14th, 2011, 1:27 pm">January 14, 2011</abbr> </p>
<div style="float: left;"><img alt="zero_copy_ruby_sq" class="attachment-post-thumbnail wp-post-image" height="180" src="/media/images/2011/01/zero_copy_ruby_sq.jpg" title="zero_copy_ruby_sq" width="180"/></div>
<div class="entry-content">
<p>In this post I will explain the concept behind “zero-copy”, which is feature of the Linux allowing for faster transfer of data between pipes, file-descriptors and sockets. I will demonstrate how you can use this functionality in your Ruby projects using a code example. This functionality has been implemented in C, Java, Ruby, Perl and nameless other languages, but in this blog I will focus on the Ruby usage.</p>
</div>
<p><!-- .entry-content --></p>
<div class="continue-reading"><a href="/zero-copy-transfer-data-faster-in-ruby"><strong>Continue reading…</strong></a></div>
</div>

In this post I will explain the concept behind “zero-copy”, which is feature of the Linux allowing for faster transfer of data between pipes, file-descriptors and sockets. I will demonstrate how you can use this functionality in your Ruby projects using a code example. This functionality has been implemented in C, Java, Ruby, Perl and nameless other languages, but in this blog I will focus on the Ruby usage.

<div id="comments">
  <h3 id="comments-number" class="comments-header">29 responses to “Map-Reduce With Ruby Using Hadoop”</h3>
  <ol class="comment-list">
    <li id="comment-665" class="pingback even thread-even depth-1 pingback reader">
      <img alt="The Apache Projects – The Justice League Of Scalability | Phil Whelan's Blog" src="https://0.gravatar.com/avatar/?d=https://www.bigfastblog.com/css/library/media/images/pingback.png&amp;s=80" class="avatar avatar-80 photo avatar-default" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.bigfastblog.com/the-apache-projects-the-justice-league-of-scalability">The Apache Projects – The Justice League Of Scalability | Phil Whelan's Blog</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, January 11th, 2011, 3:48 pm">January 11, 2011</abbr> at <abbr class="comment-time" title="Tuesday, January 11th, 2011, 3:48 pm">3:48 pm</abbr>
      </div>
      <div class="comment-text">
        <p>[...] and manages the running of distributed Map-Reduce jobs. In an previous post I gave an example using Ruby with Hadoop to perform Map-Reduce [...]</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-679" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="andy" src="https://1.gravatar.com/avatar/10a8337cf8879d37affd54e1cab4d79e?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">andy</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, January 12th, 2011, 3:49 pm">January 12, 2011</abbr> at <abbr class="comment-time" title="Wednesday, January 12th, 2011, 3:49 pm">3:49 pm</abbr>
      </div>
      <div class="comment-text">
        <p>kudos on the good article</p>
        <p>I was able to follow along and everything worked (after correcting for my typos).</p>
        <p>One thing that wasn’t clear, one can be in any dir for the ruby &amp; bash file work.  I had done this originally in the hadoop folder.</p>
        <p>thanks</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-680" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, January 12th, 2011, 4:07 pm">January 12, 2011</abbr> at <abbr class="comment-time" title="Wednesday, January 12th, 2011, 4:07 pm">4:07 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thank you for your comment Andy! I’m glad you followed it all the way through and it worked.</p>
            <p>In this example the Ruby scripts for Map and Reduce are in the same directory as the bash script that I created for running the job. They do not need to be. You just need to tell the job starter where to find them. For instance, for the map.rb script I have…</p>
            <p>-file map.rb</p>
            <p>This is a relative path, meaning it’s in the same directory. It could also be relative in another directory..</p>
            <p>-file ../someotherdirectory/map.rb</p>
            <p>or we could give an absolute path…</p>
            <p>-file /Users/phil/someotherdirectory/map.rb</p>
            <p>The bash script can also be written is any directory. The path to the JAR file is absolute, so the only other dependencies are the Map and Reduce scripts as I mentioned above.</p>
            <p>I hope that answers your question.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-690" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="andy" src="https://1.gravatar.com/avatar/10a8337cf8879d37affd54e1cab4d79e?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">andy</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Thursday, January 13th, 2011, 10:45 am">January 13, 2011</abbr> at <abbr class="comment-time" title="Thursday, January 13th, 2011, 10:45 am">10:45 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hi Phil, </p>
        <p>one more question,</p>
        <p>is there a way to use s3 buckets for the input and output?</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-693" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Thursday, January 13th, 2011, 11:07 am">January 13, 2011</abbr> at <abbr class="comment-time" title="Thursday, January 13th, 2011, 11:07 am">11:07 am</abbr>
          </div>
          <div class="comment-text">
            <p>That’s a great question Andy and the answer is “yes”. You will need to modify your <em>hadoop-site.xml</em> to replace HDFS with S3.</p>
            <p>Check out this wiki page…<br />
https://wiki.apache.org/hadoop/AmazonS3</p>
            <p>This would make a good follow post, so keep watching this space!</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-719" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Navin" src="https://1.gravatar.com/avatar/35bae7b4a1c41762e6cc252ee5198fff?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Navin</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, January 16th, 2011, 2:45 am">January 16, 2011</abbr> at <abbr class="comment-time" title="Sunday, January 16th, 2011, 2:45 am">2:45 am</abbr>
      </div>
      <div class="comment-text">
        <p>Phil – thanks very much for taking the time to write this up!  I am new to AWS and EC2 and am running into a problem at the point where I first try and launch clusters – wondering if you can please help?  I have my Access Key ID and Secret Access Key in my hadoop.properties, and the output I get is as follows:</p>
        <pre>
          <small>
[~/src/cloudera/whirr-0.1.0+23]$ bin/whirr launch-cluster --config hadoop.properties                                                                                                     rvm:ruby-1.8.7-p299
Launching myhadoopcluster cluster
Exception in thread "main" com.google.inject.CreationException: Guice creation errors:
1) No implementation for java.lang.String annotated with @com.google.inject.name.Named(value=jclouds.credential) was bound.
  while locating java.lang.String annotated with @com.google.inject.name.Named(value=jclouds.credential)
    for parameter 2 at org.jclouds.aws.filters.FormSigner.(FormSigner.java:91)
  at org.jclouds.aws.config.AWSFormSigningRestClientModule.provideRequestSigner(AWSFormSigningRestClientModule.java:66)
1 error
  at com.google.inject.internal.Errors.throwCreationExceptionIfErrorsExist(Errors.java:410)
  at com.google.inject.internal.InternalInjectorCreator.initializeStatically(InternalInjectorCreator.java:166)
  at com.google.inject.internal.InternalInjectorCreator.build(InternalInjectorCreator.java:118)
  at com.google.inject.InjectorBuilder.build(InjectorBuilder.java:100)
  at com.google.inject.Guice.createInjector(Guice.java:95)
  at com.google.inject.Guice.createInjector(Guice.java:72)
  at org.jclouds.rest.RestContextBuilder.buildInjector(RestContextBuilder.java:141)
  at org.jclouds.compute.ComputeServiceContextBuilder.buildInjector(ComputeServiceContextBuilder.java:53)
  at org.jclouds.aws.ec2.EC2ContextBuilder.buildInjector(EC2ContextBuilder.java:101)
  at org.jclouds.compute.ComputeServiceContextBuilder.buildComputeServiceContext(ComputeServiceContextBuilder.java:66)
  at org.jclouds.compute.ComputeServiceContextFactory.buildContextUnwrappingExceptions(ComputeServiceContextFactory.java:72)
  at org.jclouds.compute.ComputeServiceContextFactory.createContext(ComputeServiceContextFactory.java:114)
  at org.apache.whirr.service.ComputeServiceContextBuilder.build(ComputeServiceContextBuilder.java:41)
  at org.apache.whirr.service.hadoop.HadoopService.launchCluster(HadoopService.java:84)
  at org.apache.whirr.service.hadoop.HadoopService.launchCluster(HadoopService.java:61)
  at org.apache.whirr.cli.command.LaunchClusterCommand.run(LaunchClusterCommand.java:61)
  at org.apache.whirr.cli.Main.run(Main.java:65)
  at org.apache.whirr.cli.Main.main(Main.java:91)
</small>
        </pre>
        <p>PS: The rvm:ruby-1.8.7-p299 is part of my prompt and not in the output.</p>
        <p>Thanks in advance for any pointers on getting this resolved.</p>
        <p>Regards,<br />
Navin</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-721" class="comment even thread-even depth-1 comment reader">
      <img alt="Adrian Cole" src="https://0.gravatar.com/avatar/08fdc408ec6d07b292b8fdf46b1e3e1e?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.jclouds.org">Adrian Cole</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, January 16th, 2011, 11:24 am">January 16, 2011</abbr> at <abbr class="comment-time" title="Sunday, January 16th, 2011, 11:24 am">11:24 am</abbr>
      </div>
      <div class="comment-text">
        <p>Hi, Navin.</p>
        <p>There’s probably an issue with the property syntax inside your hadoop.properties.  Have a look at https://incubator.apache.org/projects/whirr.html and https://cwiki.apache.org/WHIRR/configuration-guide.html</p>
        <p>If you still have issues, send your query to the user list, you’ll get on track quickly! </p>
        <p>whirr-user@incubator.apache.org</p>
        <p>Cheers,<br />
-Adrian</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-722" class="comment odd alt thread-odd thread-alt depth-1 comment reader">
      <img alt="Navin" src="https://1.gravatar.com/avatar/35bae7b4a1c41762e6cc252ee5198fff?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Navin</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, January 16th, 2011, 11:57 am">January 16, 2011</abbr> at <abbr class="comment-time" title="Sunday, January 16th, 2011, 11:57 am">11:57 am</abbr>
      </div>
      <div class="comment-text">
        <p>Um, yes, sorry! A problem with “whirr.credentials” getting truncated to “whirr.cred” while copying keys across.  How embarrassing!</p>
        <p>Apologies – and thanks for taking the time to reply …</p>
        <p>Navin</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-723" class="comment byuser comment-author-admin bypostauthor even depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, January 16th, 2011, 12:04 pm">January 16, 2011</abbr> at <abbr class="comment-time" title="Sunday, January 16th, 2011, 12:04 pm">12:04 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Excellent! Glad you got if figured out. When you get through to the end of the example, let us know your thoughts are of using Whirr + Hadoop + Ruby.</p>
          </div>
          <!-- .comment-text -->
          <ol class="children">
            <li id="comment-950" class="comment odd alt depth-3 comment reader">
              <img alt="Navin" src="https://1.gravatar.com/avatar/35bae7b4a1c41762e6cc252ee5198fff?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
              <div class="comment-meta comment-meta-data">
                <div class="comment-author vcard">
                  <cite class="fn" title="https://novemberkilo.com">Navin</cite>
                </div>
                <!-- .comment-author .vcard -->
                <abbr class="comment-date" title="Thursday, February 3rd, 2011, 2:23 pm">February 3, 2011</abbr> at <abbr class="comment-time" title="Thursday, February 3rd, 2011, 2:23 pm">2:23 pm</abbr>
              </div>
              <div class="comment-text">
                <p>Worked my way through the tutorial soon after my last comment and have been meaning to come back and thank you for writing this up!  I am inspired to pick something from this list </p>
                <p>https://www.quora.com/What-are-some-good-toy-problems-in-data-science </p>
                <p>and play some more.  </p>
                <p>Not all is well though – I received a bill from AWS for $100 this morning!  While following your tutorial I fumbled my way through signing up for AWS and assumed that I would only ever be operating within a free tier – boy was I wrong!  Some three weeks later and I realise that I’ve had “small” EC2 servers going all this time … </p>
                <p>So, may I suggest that you please augment your post to direct the reader to shut down (and terminate) their EC2 instances after they are done with the tute, and, perhaps, include directions on how to ensure that they use Amazon’s free https://aws.amazon.com/free/ tier?</p>
                <p>It is evident that I’ve made some silly mistakes while following this but I hope I am not atypical of your garden variety newbie that is curious about big data.  I hope that I find a way to get Amazon to reverse charges (don’t like my chances though).  In any case, my enthusiasm is not dampened – thanks again for getting me started <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
                </p>
                <p>Navin</p>
              </div>
              <!-- .comment-text -->
              <ol class="children">
                <li id="comment-980" class="comment even depth-4 comment reader">
                  <img alt="Navin" src="https://1.gravatar.com/avatar/35bae7b4a1c41762e6cc252ee5198fff?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
                  <div class="comment-meta comment-meta-data">
                    <div class="comment-author vcard">
                      <cite class="fn">Navin</cite>
                    </div>
                    <!-- .comment-author .vcard -->
                    <abbr class="comment-date" title="Saturday, February 5th, 2011, 11:51 pm">February 5, 2011</abbr> at <abbr class="comment-time" title="Saturday, February 5th, 2011, 11:51 pm">11:51 pm</abbr>
                  </div>
                  <div class="comment-text">
                    <p>Further update: Amazon replied to my request within 12 hours and, in fact, did reverse charges.  Phew!</p>
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
    <li id="comment-824" class="comment byuser comment-author-admin bypostauthor odd alt thread-even depth-1 comment role-administrator user-admin entry-author">
      <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.bigfastblog.com">Phil Whelan</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, January 21st, 2011, 7:04 pm">January 21, 2011</abbr> at <abbr class="comment-time" title="Friday, January 21st, 2011, 7:04 pm">7:04 pm</abbr>
      </div>
      <div class="comment-text">
        <p>I have just posted “Quickly Launch A Cassandra Cluster On Amazon EC2″, which follows a very similar process. https://www.bigfastblog.com/quickly-launch-a-cassandra-cluster-on-amazon-ec2</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-1140" class="comment byuser comment-author-cjbottaro even thread-odd thread-alt depth-1 comment role-subscriber user-cjbottaro">
      <img alt="cjbottaro" src="https://1.gravatar.com/avatar/7ac9652c07e2ce83b8288663460f5fa8?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.google.com/profiles/106840081710342094498">cjbottaro</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, February 18th, 2011, 9:35 pm">February 18, 2011</abbr> at <abbr class="comment-time" title="Friday, February 18th, 2011, 9:35 pm">9:35 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Great post, thank you very much for that.</p>
        <p>Next question though… how do you use Cassandra for input/output (while still using Ruby)?</p>
        <p>I know you can run Hadoop jobs against Cassandra in Java with the InputFormat they provide, but how to do so using streaming?</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-1939" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Sid" src="https://0.gravatar.com/avatar/84d25496d54d37643490909dfdc7b75c?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://truenorth.gb.com">Sid</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, April 12th, 2011, 3:44 am">April 12, 2011</abbr> at <abbr class="comment-time" title="Tuesday, April 12th, 2011, 3:44 am">3:44 am</abbr>
      </div>
      <div class="comment-text">
        <p>What a great post! Really clearly written and keeps to it’s promise of being able to demonstrate through repeatable steps. Particularly liked the cues for tea breaks for the lengthy download/install steps. Nice work!</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-3064" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Allen" src="https://0.gravatar.com/avatar/e67a987a5b092841127f1545a441401b?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Allen</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, May 28th, 2011, 12:19 am">May 28, 2011</abbr> at <abbr class="comment-time" title="Saturday, May 28th, 2011, 12:19 am">12:19 am</abbr>
      </div>
      <div class="comment-text">
        <p>When i would like fire up a customized ec2 instance, i added following parameters into hadoop.properties:</p>
        <p>whirr.image-id= us-west-1/ami-**********<br />
jclouds.ec2.ami-owners=******************<br />
whirr.hardware-id= m1.large<br />
whirr.location-id=us-west-1</p>
        <p>But it doesn’t work. Any thought? thanks</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-3071" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Adrian Cole" src="https://0.gravatar.com/avatar/08fdc408ec6d07b292b8fdf46b1e3e1e?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://code.google.com/p/jclouds">Adrian Cole</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, May 28th, 2011, 11:09 am">May 28, 2011</abbr> at <abbr class="comment-time" title="Saturday, May 28th, 2011, 11:09 am">11:09 am</abbr>
      </div>
      <div class="comment-text">
        <p>I think you would be best taking this to the whirr user list, where you can let us know what didn’t work:<br />
   https://cwiki.apache.org/confluence/display/WHIRR/MailingLists</p>
        <p>There are recipes in the latest version of whirr here, as well:<br />
   https://svn.apache.org/repos/asf/incubator/whirr/trunk/recipes/hadoop-ec2.properties</p>
        <p>Unrelated, but if you don’t mind trying 0.5.0, voting it up can get the new rev, including recipes released!<br />
   https://people.apache.org/~tomwhite/whirr-0.5.0-incubating-candidate-1/</p>
        <p>Cheers,<br />
A</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-3079" class="comment even depth-2 comment reader">
          <img alt="Allen" src="https://0.gravatar.com/avatar/e67a987a5b092841127f1545a441401b?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">Allen</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, May 29th, 2011, 1:07 am">May 29, 2011</abbr> at <abbr class="comment-time" title="Sunday, May 29th, 2011, 1:07 am">1:07 am</abbr>
          </div>
          <div class="comment-text">
            <p>Cool, I will try both, post the questions and new Whirr, will update here later on if i get any solution. Thx <img src="/media/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />
            </p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
        <li id="comment-3081" class="comment odd alt depth-2 comment reader">
          <img alt="Allen" src="https://0.gravatar.com/avatar/e67a987a5b092841127f1545a441401b?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn">Allen</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Sunday, May 29th, 2011, 3:54 am">May 29, 2011</abbr> at <abbr class="comment-time" title="Sunday, May 29th, 2011, 3:54 am">3:54 am</abbr>
          </div>
          <div class="comment-text">
            <p>I tried whirr 0.5.0, and it still doesn’t work. </p>
            <p>It worked well if my hadoop.properties as below:</p>
            <p>whirr.service-name=hadoop<br />
whirr.cluster-name=myhadoopcluster<br />
whirr.instance-templates=1 jt+nn,1 dn+tt<br />
whirr.provider=ec2<br />
whirr.identity=<br />
whirr.credential=<br />
whirr.private-key-file=${sys:user.home}/.ssh/id_rsa<br />
whirr.public-key-file=${sys:user.home}/.ssh/id_rsa.pub<br />
whirr.hardware-id= m1.large<br />
whirr.location-id=us-west-1<br />
whirr.hadoop-install-runurl=cloudera/cdh/install<br />
whirr.hadoop-configure-runurl=cloudera/cdh/post-configure</p>
            <p>But once i added two more lines into hadoop.properties file, it went wrong:<br />
whirr.image-id= us-west-1/ami-***** (my ami)<br />
jclouds.ec2.ami-owners=(my owner id&gt;</p>
            <p>I have posted a question on whirr forum and see if i could get any solution. Will update here if i get anything. thx</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-3204" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Jack Veenstra" src="https://1.gravatar.com/avatar/b99708c5a96497dd952ad6d24c4da74c?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Jack Veenstra</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Wednesday, June 8th, 2011, 6:22 pm">June 8, 2011</abbr> at <abbr class="comment-time" title="Wednesday, June 8th, 2011, 6:22 pm">6:22 pm</abbr>
      </div>
      <div class="comment-text">
        <p>There’s a bug in your reduce script.  You output the total only when you get a new key.  So the last key’s total will never be included in the output.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-3208" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Wednesday, June 8th, 2011, 11:21 pm">June 8, 2011</abbr> at <abbr class="comment-time" title="Wednesday, June 8th, 2011, 11:21 pm">11:21 pm</abbr>
          </div>
          <div class="comment-text">
            <p>Thanks Jack! You’re right. Well spotted.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-4107" class="comment even thread-even depth-1 comment reader">
      <img alt="threecuptea" src="https://0.gravatar.com/avatar/8aeccde7f8556f3b36f0be2c98bafc96?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">threecuptea</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, August 6th, 2011, 6:37 pm">August 6, 2011</abbr> at <abbr class="comment-time" title="Saturday, August 6th, 2011, 6:37 pm">6:37 pm</abbr>
      </div>
      <div class="comment-text">
        <p>I started using EC2 yesterday and I got this working today thanks to your article.  However, it’s not without twist and turn.<br />
1.  I got https://whirr.s3.amazonaws.com/0.3.0-cdh3u1/util/configure-hostnames not found error when I run ‘whirr launch-cluster’. They haven’t put configure-hostnames for cdh3u1 in s3 yet.   I workaround by adding –run-url-base https://whirr.s3.amazonaws.com/0.3.0-cdh3u0/util.  I got this advice from https://www.cloudera.com/blog/2011/07/cdh3u1-released/</p>
        <p>2. I followed the instruction to set up Hadoop client in the the host initiating whirr and got the following error when it tried to connect to name node.<br />
11/08/07 01:03:48 INFO ipc.Client: Retrying connect to server: ec2-107-20-60-75.<br />
compute-1.amazonaws.com/10.116.78.198:8020. Already tried 9 time(s).<br />
Bad connection to FS. command aborted. exception: Call to ec2-107-20-60-75.compu<br />
te-1.amazonaws.com/10.116.78.198:8020 failed on local exception: java.net.Socket<br />
Exception: Connection refused</p>
        <p>It has to do with security.  I checked the security group jclouds#myhadoopcluster3#us-east-1.  It allows inbound on 80, 50070, 50030 only from the host initiating whirr launch-cluster and allow inbound on 8020, 8021 only from the name node host.   I added rules to allow inbound on 8020, 8021 from the host initiating whirr and apply the rule change.  That doesn’t help.   In my case, the host initiating whirr launch-cluster is a EC2 instance too.</p>
        <p>3.  I can ssh to cluster hosts from the host initiating whirr without any key.  iptable is empty and selinux is disabled.  Network rules seems set up outside the linux box. No luck.</p>
        <p>4.  I ends up transferring files to name nodes and run map reduce job there. Whirr script create /user/hive/warehouse but no /user/ec2-user.  Need to create that directory and input sub-directory.  You might also add  -jobconf mapred.reduce.tasks=1 since the default is 10 in this case.</p>
        <p>Thanks.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-4141" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Tuesday, August 9th, 2011, 11:39 am">August 9, 2011</abbr> at <abbr class="comment-time" title="Tuesday, August 9th, 2011, 11:39 am">11:39 am</abbr>
          </div>
          <div class="comment-text">
            <p>Hi threecuptea,</p>
            <p>Do you mind re-posting this to whirr’s mailing list?</p>
            <p>https://incubator.apache.org/whirr/mail-lists.html</p>
            <p>There’s a release almost finished which should support cdh much better (v0.6)</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-4111" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Harit" src="https://0.gravatar.com/avatar/2a0b2959ff20e8654aabee4b7e49a6ed?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://hhimanshu.wordpress.com">Harit</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, August 6th, 2011, 11:08 pm">August 6, 2011</abbr> at <abbr class="comment-time" title="Saturday, August 6th, 2011, 11:08 pm">11:08 pm</abbr>
      </div>
      <div class="comment-text">
        <p>I guess the reducer is missing the code,because the last line when it completes, has to put the result to the output.<br />
I ran the same logic in Java and then in Ruby using hadoop and realized that my last node is missing in the result data. so I added the following line at the very end of reducer.rb</p>
        <p>puts prev_key + separator + key_total.to_s</p>
        <p>and it worked.</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-4657" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Daniel" src="https://1.gravatar.com/avatar/9b5c88970dfb4863b33738e06603f07c?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Daniel</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, September 13th, 2011, 1:43 pm">September 13, 2011</abbr> at <abbr class="comment-time" title="Tuesday, September 13th, 2011, 1:43 pm">1:43 pm</abbr>
      </div>
      <div class="comment-text">
        <p>Hi Phil, </p>
        <p>I am using Hadoop Mapreduce to predict secondary structure of a given long sequence. The idea is, I have a chunks of segments of a sequence and they are written into a single file input where each line is one segment. I have used one of the programs for secondary structure predictions as my mapper code (Hadoop Streaming).<br />
The out put of the mapper was successful that it produces the predicted structures in terms of dot-bracket notation. I want to use a simple reducer that glue all the outputs from the mapper in an orderly manner.<br />
For Example, If my input was like</p>
        <p>….<br />
And my mapper output is a predicted structure but not in order</p>
        <p>What I am looking is a reducer code that sorts and Glue and outputs in a form similar to the following:<br />
……</p>
        <p>Any help…Thanks</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-4787" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Dr. SHyam Sarkar" src="https://0.gravatar.com/avatar/ed7021750051605f968da939d8c1c9c4?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://www.ayushnet.com">Dr. SHyam Sarkar</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Friday, September 23rd, 2011, 7:47 pm">September 23, 2011</abbr> at <abbr class="comment-time" title="Friday, September 23rd, 2011, 7:47 pm">7:47 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Hello,</p>
        <p>We have following properties set :</p>
        <p>whirr.service-name=hadoop<br />
whirr.cluster-name=myhadoopcluster<br />
whirr.instance-templates=1 jt+nn,1 dn+tt<br />
whirr.provider=ec2<br />
whirr.credential=mYar/KSbx+UL+nqGr9hSgGHIOqXC9tjNcuO9UwF/<br />
whirr.identity=AKIAJCDTYGREJYIECQZA<br />
whirr.private-key-file=${sys:user.home}/.ssh/id_rsa<br />
whirr.public-key-file=${sys:user.home}/.ssh/id_rsa.pub<br />
whirr.hadoop-install-runurl=cloudera/cdh/install<br />
whirr.hadoop-configure-runurl=cloudera/cdh/post-configure<br />
whirr.image-id=ami-3bc9997e<br />
whirr.hardware-id=i-bffa23f8<br />
whirr.location-id=us.west-1c</p>
        <p>But we are getting following error:</p>
        <p>[ec2-user@ip-10-170-103-243 ~]$ ./whirr-0.3.0-cdh3u1/bin/whirr launch-cluster –config hadoop.properties –run-url-base https://whirr.s3.amazonaws.com/0.3.0-cdh3u0/util<br />
Bootstrapping cluster<br />
Configuring template<br />
Exception in thread “main” java.util.NoSuchElementException<br />
        at com.google.common.collect.AbstractIterator.next(AbstractIterator.java:147)<br />
        at com.google.common.collect.Iterators.find(Iterators.java:679)<br />
        at com.google.common.collect.Iterables.find(Iterables.java:555)<br />
        at org.jclouds.compute.domain.internal.TemplateBuilderImpl.locationId(TemplateBuilderImpl.java:492)<br />
        at org.apache.whirr.service.jclouds.TemplateBuilderStrategy.configureTemplateBuilder(TemplateBuilderStrategy.java:41)<br />
        at org.apache.whirr.service.hadoop.HadoopTemplateBuilderStrategy.configureTemplateBuilder(HadoopTemplateBuilderStrategy.java:31)<br />
        at org.apache.whirr.cluster.actions.BootstrapClusterAction.buildTemplate(BootstrapClusterAction.java:144)<br />
        at org.apache.whirr.cluster.actions.BootstrapClusterAction.doAction(BootstrapClusterAction.java:94)<br />
        at org.apache.whirr.cluster.actions.ScriptBasedClusterAction.execute(ScriptBasedClusterAction.java:74)<br />
        at org.apache.whirr.service.Service.launchCluster(Service.java:71)<br />
        at org.apache.whirr.cli.command.LaunchClusterCommand.run(LaunchClusterCommand.java:61)<br />
        at org.apache.whirr.cli.Main.run(Main.java:65)<br />
        at org.apache.whirr.cli.Main.main(Main.java:91)</p>
        <p>Can we get any help ?  What shold we do ?</p>
        <p>Thanks,<br />
S.Sarkar</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-9136" class="comment odd alt thread-even depth-1 comment reader">
      <img alt="Joao Salcedo" src="https://0.gravatar.com/avatar/26eee1500d078ca201158cd5771eda56?s=80&amp;d=https%3A%2F%2F0.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">Joao Salcedo</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Sunday, February 12th, 2012, 12:52 am">February 12, 2012</abbr> at <abbr class="comment-time" title="Sunday, February 12th, 2012, 12:52 am">12:52 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Nice tutorial, Everything work just how it should be !!!</p>
        <p>Just a small question , what if I wanna connect to the instance , where I can find the key in order to connect to it.</p>
        <p>Cheers,</p>
        <p>Joao</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
    <li id="comment-14745" class="comment even thread-odd thread-alt depth-1 comment reader">
      <img alt="Andrii Vozniuk" src="https://1.gravatar.com/avatar/f62cb86de3076e13d16a4722e631e91f?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://andrii.vozniuk.com">Andrii Vozniuk</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Tuesday, May 15th, 2012, 9:27 am">May 15, 2012</abbr> at <abbr class="comment-time" title="Tuesday, May 15th, 2012, 9:27 am">9:27 am</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>Phil, thanks for the detailed tutorial!<br />
I had my custom MapReduce application up and running on an EC2 cluster just in a few hours.<br />
I reproduced the steps with whirr-0.7.1 and hadoop-0.20.2-cdh3u4.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-14782" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Tuesday, May 15th, 2012, 3:40 pm">May 15, 2012</abbr> at <abbr class="comment-time" title="Tuesday, May 15th, 2012, 3:40 pm">3:40 pm</abbr> | Permalink  </div>
          <div class="comment-text">
            <p>Great! I’m glad the process still working and the blog post is still valid. Thanks for the comment.</p>
          </div>
          <!-- .comment-text -->
        </li>
        <!-- .comment -->
      </ol>
    </li>
    <!-- .comment -->
    <li id="comment-213930" class="comment even thread-even depth-1 comment reader">
      <img alt="bodla dharani kumar" src="https://1.gravatar.com/avatar/1e9164a8ada74fc8faf55ecebfeed87b?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn">bodla dharani kumar</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, February 15th, 2014, 10:58 pm">February 15, 2014</abbr> at <abbr class="comment-time" title="Saturday, February 15th, 2014, 10:58 pm">10:58 pm</abbr> | Permalink  </div>
      <div class="comment-text">
        <p>hi to all,<br />
Good Morning,<br />
I had a set of 22documents in the form of text files of size 20MB and loaded in hdfs,when running hadoop streaming map/reduce funtion from command line of hdfs ,it took 4mins 31 secs for streaming the 22 text files.How to increase the map/reduce process as fast as possible so that these text files should complete the process by 5-10 seconds.<br />
What changes I need to do on ambari hadoop.<br />
And having cores = 2<br />
Allocated 2GB of data for Yarn,and 400GB for HDFS<br />
default virtual memory for a job map-task = 341 MB<br />
default virtual memory for a job reduce-task = 683 MB<br />
MAP side sort buffer memory = 136 MB<br />
And when running a job ,Hbase error with Region server goes down,Hive metastore status service check timed out.</p>
        <p>Note:[hdfs@s ~]$ hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming-2.2.0.2.0.6.0-76.jar -D mapred.map.tasks=2 -input hdfs:/apps/*.txt -output /home/ambari-qa/8.txt -mapper /home/coartha/mapper.py -file /home/coartha/mapper.py -reducer /home/coartha/reducer.py -file /home/coartha/reducer.py</p>
        <p>Thanks &amp; regards,<br />
Bodla Dharani Kumar,</p>
      </div>
      <!-- .comment-text -->
    </li>
    <!-- .comment -->
  </ol>
  <!-- .comment-list -->
</div>

