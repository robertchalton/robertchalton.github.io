---
layout: post
published: true
title: IP Address lookups with MaxMind and SQL
date: '2018-01-31'
---

# Lookup who owns an IP address

[Oracle APEX](http://apex.oracle.com) and [Oracle REST Data Services](http://oracle.com/rest) on a busy system can generate a lot of access data. This data has of course the remote ip address that is accessing it. Those IP addresses are known and assigned to companies. My thought was to find that data and get it into the database where SQL could be used to join to the access data.


# MaxMind - ASN database

[Autonomous System Number](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)) simply is the number assigned to companies that own ip address ranges. The full details are on the wiki page. The important thing is MaxMind has a [csv version](https://dev.maxmind.com/geoip/geoip2/geolite2/) of all this data which they make public and under [Creative Commons Attribution-Share A like 4.0 ](https://creativecommons.org/licenses/by-sa/4.0/)


<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Great news GeoLite2 customers! We just added a GeoLite2 ASN database. You can find details here: <a href="https://t.co/n5hTJJhSEj">https://t.co/n5hTJJhSEj</a>.</p>&mdash; MaxMind (@maxmind) <a href="https://twitter.com/maxmind/status/861964844441927684?ref_src=twsrc%5Etfw">May 9, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


# Github

Here's the github repo where I started this [oracle-maxmind](https://github.com/krisrice/maxmind-oracledb). The project's README has the details all outlined. There's a small install script that uses SQLcl's LOAD command to load up the CSV file.  Then a plsql package to convert the IPs into numbers to make it easier to lookup and join. 


<div class="github-widget" data-username="krisrice"></div>
<script src="https://unpkg.com/github-card@1.2.1/dist/widget.js"></script>

# Putting it to use with the APEX_ACTIVITY_LOG
![apex_mindmax_lookup.png]({{site.baseurl}}/img/apex_mindmax_lookup.png)


# Next Steps

The next things to do is add Country,City. and IPv6 which is in the csv files
