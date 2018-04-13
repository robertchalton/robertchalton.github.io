---
layout: post
published: true
title: 'Open Sourcing dbtools-commons, SQLcl, and more'
date: '2018-04-12'
---
## What are we Open Sourcing ?

This library of code is the core sql scripting and db access for SQL Developer and SQLcl. It is also used in other places for example in the new Autonomous Warehouse's Machine Learning feature. 

Here's what's being open sourced.

```

dhcp-10-10-183-239:dbtools-commons klrice$ cloc .
    2004 text files.
    1940 unique files.                                          
     738 files ignored.

github.com/AlDanial/cloc v 1.74  T=10.75 s (118.0 files/s, 39885.5 lines/s)
--------------------------------------------------------------------------------
Language                      files          blank        comment           code
--------------------------------------------------------------------------------
Java                           1086          28984          74179         174195
XML                             118           1200           3143         124988
yacc                              1           1641           1839           9632
Maven                            10            191            245           1688
XSLT                              2            132            203           1214
JSON                              3              0              0            975
xBase                            11            109            300            699
Markdown                          9            264              0            644
SQL                               9             55            106            522
Bourne Again Shell                1             37            134            318
JavaScript                        1             33             54            261
XSD                               3             27             72            229
HTML                             14             46            209            111
Ant                               1             23             25             64
--------------------------------------------------------------------------------
SUM:                           1269          32742          80509         315540
--------------------------------------------------------------------------------
```


Just in case that's not obvious enough..

The [https://github.com/oracle/dbtools-commons]() is organized into a few folders.

- common - base library 
- http - thin wrapper over apache http client that we use
- hudsonplugin - example this library as a hudson build target
- jdbcrest - the JDBC Driver that talks REST to the ORDS SQL over REST feature
- sqlcl - The glue from the cmd line to the common library


## Why open source?

There's a few reasons we're open sourcing this library of code. The first is simply so others can use this as a library into existing java programs. I've noticed on StackOverflow and other places developers using various techniques to run a sql script from a java program. 

It takes a small amount of java to add the ability to run a sql script. Here's the minimum required which is to setup a connection, create an output buffer, create a scripting context, and run.
![Example 1](https://krisrice.io/img/xWarq.png)

Next is to open the option up for folks to customize a build of SQLcl themselves.  This can range from a new output formatter by starting from the built in [CSVFormatter](https://github.com/oracle/dbtools-commons/blob/master/common/src/main/java/oracle/dbtools/raptor/format/CSVFormatter.java) to a whole new command by looking at how the "[apex](https://github.com/oracle/dbtools-commons/blob/master/common/src/main/java/oracle/dbtools/raptor/newscriptrunner/apex/APEXExport.java)" command works

There's also a lot of non-script running APIs that can be leveraged such as the SQL/PLSQL formatter, the [Abori](https://vadimtropashko.wordpress.com/2017/02/11/arbori-the-missing-manuals/) parser.

[Javascript/Nashorn Scripting](https://github.com/oracle/oracle-db-tools/blob/master/sqlcl/SCRIPTING.md) will be easier now as well. There's a lot that is in SQLcl but not covered in the scripting examples yet. With full sources, everything is available to be leveraged in some client side javascript for control flow in sql scripting.

## Getting Started


If you liked the movie Inception, you'll like this.

Step 1 Setting up the environment. The code uses maven 3.5.2+. There are a lot of required libraries ship with SQLcl so the easiest thing we could do was add a pom.xml file to [SQLcl 18.1.1](http://www.oracle.com/technetwork/developer-tools/sqlcl/downloads/index.html).  This will add all the required libraries to the local maven repository. Once SQLcl is downloaded and unzipped:

```
$ cd sqlcl
$ cd lib
$ ls pom.xml
pom.xml
$ mvn validate
[INFO] Scanning for projects...
....

```

Step 2 Clone from [https://github.com/oracle/dbtools-commons](https://github.com/oracle/dbtools-commons)

Step 3 Build

```
$ mvn -U clean install
..
..... One Eternity Later....
..
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] DBTools Resource Generation maven plugin ........... SUCCESS [  2.050 s]
[INFO] DBTools Common Project Parent POM .................. SUCCESS [  0.436 s]
[INFO] DBTools Common Library ............................. SUCCESS [ 12.235 s]
[INFO] DBTools HTTP Library ............................... SUCCESS [  0.291 s]
[INFO] DBTools REST JDBC Driver ........................... SUCCESS [  1.577 s]
[INFO] DBTools SQLDeveloper SQLcl Library and Distribution  SUCCESS [  7.307 s]
[INFO] DBTools SQLcl Plugin for Hudson .................... SUCCESS [  4.394 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 29.224 s
[INFO] Finished at: 2018-04-12T13:52:02-04:00
[INFO] Final Memory: 73M/1064M
[INFO] ------------------------------------------------------------------------
```

Step 4 Built targets
A copy of SQLcl is now located here->

**sqlcl/target/dbtools-sqlcl-18.2.0-SNAPSHOT-sqlcl/dbtools-sqlcl-18.2.0-SNAPSHOT**

Step 5 Build something and tell us about it.

## Example Code

Included is an example of making a [SQLcl Build Target plugin](https://github.com/oracle/dbtools-commons/tree/master/hudsonplugin) for Hudson-CI. A lot of places now are doing Continuous Integration / Deployment using hudson. There are plenty of ways to do this today most of which include shelling out to exec something. This adds a native SQLcl builder right into Hudson.

Be sure to check blogs of [Barry](http://barrymcgillin.blogspot.com/) and [myself](http://krisrice.io/) for more examples.

We will make more examples over time. If there's something specific you are looking for, log an issue on the GitHub repo or find me on twitter [@krisrice](https://twitter.com/krisrice)


## So now What?

This base of code is very actively developed and will be maintained. We will be pushing this repository with every release.
