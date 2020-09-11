---
title: "SQLShell. A Cross-Database SQL Tool With NoSQL Potential"
date: "2011-01-18T17:07:00-08:00"
template: "post"
draft: false
slug: "sqlshell-a-cross-database-sql-tool-with-nosql-potential"
category: "DevOps"
tags:
  - clapper.org
  - couchdb
  - google storage
  - hbase
  - hive
  - homebrew
  - jdbc
  - mongodb
  - mysql
  - nosql
  - oracle
  - postgresql
  - psql
  - rdbms
  - redis
  - scala
  - sqlcmd
  - sqlite
  - sqlshell
description: "In this blog post I will introduce SQLShell and demonstrate, step-by-step, how to install it and start using it with MySQL. I will also reflect on the possibilites of using this with NoSQL technologies, such as HBase, MongoDB, Hive, CouchDB, Redis and Google BigQuery. SQLShell is a cross-platform, cross-database command-line tool for SQL, much like psql for PostgreSQL or the mysql command-line tool for MySQL."
socialImage:
  publicURL: "/media/images/2011/01/sqlshell_nosql_potential_sq.jpg"
---
<a href="https://www.flickr.com/photos/pvera/114420037/" title="Will code for food by pvera, on Flickr"><img align="left" alt="Will code for food" height="180" src="https://farm1.static.flickr.com/45/114420037_f08201d9b8_m.jpg" style="border: none; padding: none; padding-right: 50px; padding-bottom: 20px;" width="240"/></a> 

In this blog post I will introduce SQLShell and demonstrate, step-by-step, how to install it and start using it with MySQL. I will also reflect on the possibilites of using this with NoSQL technologies, such as HBase, MongoDB, Hive, CouchDB, Redis and Google BigQuery.

[SQLShell](https://software.clapper.org/sqlshell) is a cross-platform, cross-database command-line tool for SQL, much like _psql_ for PostgreSQL or the _mysql_ command-line tool for MySQL.

## Why Use It?

If you already use only one command-line tool, such as PostgreSQL’s _psql_ or MySQL’s _mysql_ tool and are happy then you may not need to. If you find yourself jumping between several of these tools and would like common functionality, or you are a fan of the many NoSQL technologies out there, then this is worth keeping an eye on.

### JDBC Driver Support

SQLShell is built in [Scala](https://www.scala-lang.org/). Scala is a scalable programming language that compiles to Java byte code, can run on the JVM and can be used alongside Java code. Therefore, SQLShell can interface to any database for which there is a JDBC (Java Database Connectivity) driver. Encase you do not know, there are many.

#### JDBC Drivers Available

<li>RDBMS Databases</li>

*   [Oracle](https://www.oracle.com/technetwork/database/features/jdbc/index-091264.html)
*   [MySQL](https://www.mysql.com/downloads/connector/j/)
*   [PostgreSQL](https://jdbc.postgresql.org/)
*   [SQLite](https://www.zentus.com/sqlitejdbc/)

There are several JDBC drivers for many NoSQL technologies, but none of these implement JDBC fully enough to SQLShell, _yet_.

*   [HBase](https://www.hbql.com/examples/jdbc.html)
*   [MongoDB](https://github.com/erh/mongo-jdbc)
*   [Hive ](https://aws.typepad.com/aws/2011/01/elastic-mapreduce-updates-hive-multipart-upload-jdbc-squirrel-sql.html)
*   [CouchDB](https://github.com/fellix/couchdb-j)
*   [Redis](https://code.google.com/p/jdbc-redis/)

Something to note about Google’s recent [BigQuery](https://code.google.com/apis/bigquery/) (part of [Google Storage](https://code.google.com/apis/storage/)) is that, while they do not have a JDBC driver, yet, they do internally make use of [sqlcmd](https://github.com/bmc/sqlcmd/) with BigQuery. Sqlcmd is also created by [Brian M. Clapper at Clapper.org](https://software.clapper.org/), but he has discontinued development of it in favor of the new SQLShell. Therefore, I think there is a good chance that Google may soon develop a JDBC driver compatible with SQLShell. That said, Google does like Python and sqlcmd is written in Python.

Brian M. Clapper, author of SQLShell, writes about the move from Python to Scala in the [SQLShell user-guide](https://software.clapper.org/sqlshell/users-guide.html).

>  
> SQLShell is a SQL command line tool, similar in concept to tools like Oracle’s SQL Plus, the PostgreSQL psql command, and the MySQL mysql tool.
> 
> SQLShell is a Scala rewrite of my Python sqlcmd tool (rewritten because, as it turns out, I think JDBC is more consistent and portable than Python’s DB API).
> 
> [ – Brian M. Clapper, author of SQLShell](https://software.clapper.org/sqlshell/users-guide.html)
> 

## Installing SQLShell

You can [download a pre-compiled installation JAR file for SQLShell](https://software.clapper.org/sqlshell/#binary_releases) or [compile the binary yourself](https://software.clapper.org/sqlshell/#building_from_source). Downloading the JAR is recommended and I have download [version 0.7.1](https://github.com/downloads/bmc/sqlshell/sqlshell-0.7.1-install.jar)

```
curl -O https://cloud.github.com/downloads/bmc/sqlshell/sqlshell-0.7.1-install.jar
```

Running this installer can be done with the _java_ command, which will launch a graphical installer. The graphical installer uses [IzPack](https://izpack.org/), which is a cross-platform installation framework. Therefore, even though I am using Mac OS X, you should not have any problems installing on Windows or Linux.

```
java -jar sqlshell-0.7.1-install.jar
```

First thing you see if all goes well is the language prompt. I am speak the English.

<img alt="SQLShell Installer language prompt" height="230" src="https://commondatastorage.googleapis.com/philwhln/blog/media/images/sqlshell_nosql/sqlshell_install_prompt.png" width="304"/>

Following this you will see a welcome screen, then a intro and info page, and then the license page. Be sure to read the license carefully – especially the part about “your first born child”.

Here is what you will see for the intro screen. 

<img alt="SQLShell Installer intro screen" height="515" src="https://commondatastorage.googleapis.com/philwhln/blog/media/images/sqlshell_nosql/sqlshell_install_intro_screen.png" width="670"/>

When you get to the end, you will see the following message…

>  
> You’ve successfully installed SQLShell.
> 
> For your convenience, a command-line wrapper script has been provided in  
> the “bin” directory under “/Applications/clapper.org/sqlshell”.
> 

That tells you where sqlshell is installed. Under this directory is the _bin_ directory, which contains _sqlshell_. It’s a command-line tool, and since we will be using it often, I’ll add the path of that _bin_ directory to my PATH.

```
echo 'export PATH=$PATH:/Applications/clapper.org/sqlshell/bin' >> ~/.profile
source ~/.profile
which sqlshell || echo "Not found in path" # This tests that it's found in your PATH
```

We can see what parameters it expects, by running _sqlshell_ with the _–help_ argument.

```
sqlshell --help

SQLShell, version 0.7.1 (2010/11/10 17:27:55)

Usage: sqlshell [OPTIONS] db [@file]

OPTIONS

-?
-h
--help                Show this usage message.

-V
--version             Show version and exit.

-c config_file
--config config_file  Specify configuration file. Defaults to:
                      /Users/phil/.sqlshell/config

-n
--no-ansi
--noansi              Disable the use of ANSI terminal sequences. This option
                      just sets the initial value for this setting. The value
                      can be changed later from within SQLShell itself.

-r lib_name
--readline lib_name   Specify readline libraries to use. Legal values:
                      editline, getline, gnu, jline, simple. (May be specified
                      multiple times.)

-s
--stack               Show all exception stack traces.

-v
--verbose             Enable various verbose messages. This option just sets
                      the initial verbosity value. The value can be changed
                      later from within SQLShell itself.

PARAMETERS

db     Name of database to which to connect, or an on-the-fly database
       specification, of the form:

           driver,url,[user[,password]]

       If the name of a database is specified, SQLShellwill look in the
       configuration file for the corresponding connection parameters. If a
       database specification is used, the specification must one argument. The
       driver can be a full driver class name, or a driver alias from the
       configuration file. The user and password are optional, since some
       databases (like SQLite) don't require them at all.

@file  Path of file of commands to run

```

## SQLShell With MySQL

### Install MySQL’s JDBC Driver

Download the driver from [MySQL’s website](https://www.mysql.com/downloads/connector/j/).

```
curl -O 
```

Above, we copied the _mysql-connector-java-5.1.14-bin.jar_ file to SQLShell’s _lib_. SQLShell will load all the JAR files found in that directory at start-up, and will then reference them by package and class name. We will use the class name of the MySQL driver to configure a “mysql” alias in the SQLShell configuration.

To configure the “mysql” alias we will edit the default configuration file _~/.sqlshell/config_, which currently does not exist.

```
mkdir ~/.sqlshell
vim ~/.sqlshell/config # I use vim to edit files

```

Add the following configuration…

```
[drivers]
mysql = com.mysql.jdbc.Driver

```

### Connect To MySQL Using SQLShell

The format of your connection string for the MySQL JDBC driver should be

```
jdbc:mysql://[host][,failoverhost...][:port]/[database][?propertyName1][=propertyValue1][&propertyName2][=propertyValue2]...
```

I’m just going to connect to the local mysql database and look at the pre-installed MySQL database called “test”. You should have the same database, unless you deleted it.

```
sqlshell mysql,jdbc:mysql://localhost/test?user=root
```

```
SQLShell, version 0.7.1 (2010/11/10 17:27:55)
Copyright (c) 2009-2010 Brian M. Clapper
Using JLine
Type "help" for help. Type ".about" for more information.

sqlshell>

```

Ok, we are connected to MySQL and can start running some queries.

We can see the commands that we can run using the _help_ command.

```
sqlshell> help
Help is available for the following commands:
-------------------------------------------------------------------------------
.about    .capture  .desc     .echo     .run      .set      .show
alter     begin     commit    create    delete    drop      exit
help      history   insert    r         rollback  select    update

```

Let’s see what databases we have…

```
sqlshell> show databases;
Execution time: 0.26 seconds
Retrieval time: 0.21 seconds
2 rows returned.

SCHEMA_NAME
------------------
information_schema
test

sqlshell>

```

We can create an new database.

```
sqlshell> create database sqlshell_test;
1 row affected.
Execution time: 0.5 seconds

```

SQLShell passes SQL statements in full to the database, so you can use any commands that your database understands.

<table style="margin: none; border: none" width="100%">
<tr>
<td align="left" style="border: none">
<h2>Conclusion</h2>
<p>The initial purpose of this blog post was to demonstrate SQLShell with a NoSQL database, but unfortunately I failed to find a JDBC driver that implemented all the features that SQLShell requires. I’m hoping that development of such drivers will continue, as it will be good to have standard inferface to all these NoSQL technologies directly from the command-line. I look forward to seeing how SQLShell and these JDBC develop over time.</p>
</td>
<td align="left" style="width: 300px; border: none"><script type="text/javascript">&lt;!--
google_ad_client = "pub-5260338827095335";
/* philwhln sqlshell sq cnclsn */
google_ad_slot = "3689646111";
google_ad_width = 336;
google_ad_height = 280;
//--&gt;
</script><br/>
<script src="https://pagead2.googlesyndication.com/pagead/show_ads.js" type="text/javascript">
</script></td>
</tr>
</table>

<div id="comments">
  <h3 id="comments-number" class="comments-header">2 responses to “SQLShell. A Cross-Database SQL Tool With NoSQL Potential”</h3>
  <ol class="comment-list">
    <li id="comment-21155" class="comment even thread-even depth-1 comment reader">
      <img alt="Endre" src="https://1.gravatar.com/avatar/3f3b0ce9b2bad9e2181945e64e85bc48?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
      <div class="comment-meta comment-meta-data">
        <div class="comment-author vcard">
          <cite class="fn" title="https://starschema.net/">Endre</cite>
        </div>
        <!-- .comment-author .vcard -->
        <abbr class="comment-date" title="Saturday, September 1st, 2012, 9:48 am">September 1, 2012</abbr> at <abbr class="comment-time" title="Saturday, September 1st, 2012, 9:48 am">9:48 am</abbr>
      </div>
      <div class="comment-text">
        <p>Very nice and thorough article Phil,</p>
        <p>I have only one addition – actually its an update:</p>
        <p>there is a JDBC driver for Google BigQuery available for a few days now:<br />
https://code.google.com/p/starschema-bigquery-jdbc/</p>
        <p>This might give many people new perspectives for their data visualization needs.</p>
      </div>
      <!-- .comment-text -->
      <ol class="children">
        <li id="comment-21156" class="comment byuser comment-author-admin bypostauthor odd alt depth-2 comment role-administrator user-admin entry-author">
          <img alt="Phil Whelan" src="https://1.gravatar.com/avatar/5f357d996da96ccd36d3374e3728bf29?s=80&amp;d=https%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D80&amp;r=PG" class="avatar avatar-80 photo" height="80" width="80" />
          <div class="comment-meta comment-meta-data">
            <div class="comment-author vcard">
              <cite class="fn" title="https://www.google.com/profiles/101358683928607234715">Phil Whelan</cite>
            </div>
            <!-- .comment-author .vcard -->
            <abbr class="comment-date" title="Saturday, September 1st, 2012, 9:54 am">September 1, 2012</abbr> at <abbr class="comment-time" title="Saturday, September 1st, 2012, 9:54 am">9:54 am</abbr>
          </div>
          <div class="comment-text">
            <p>Great info the JDBC driver, Endre. Thanks for sharing!</p>
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

