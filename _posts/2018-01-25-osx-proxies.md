---
layout: post
published: false
title: OSX Proxies
---
## A New Post


{% highlight shell %}
  proxy=`scutil --proxy | grep ProxyAutoConfigEnable | awk -F: '{print $2}'`
 23 
 24 echo Proxy  :$proxy:
 25 if [[ "$proxy" == " 1" ]] ; then
 26   echo Proxy On
 27    git config --global https.https://github.com.proxy http://www-proxy.us.oracle.com:80
 28    git config --global http.https://github.com.proxy http://www-proxy.us.oracle.com:80
 29    export http_proxy=http://www-proxy.us.oracle.com:80/
 30    export https_proxy=http://www-proxy.us.oracle.com:80/
 31    export MAVEN_OPTS="-Dhttp.proxyHost=www-proxy.us.oracle.com -Dhttp.proxyPort=80 -Dhttps.proxyHost=www-proxy.us.oracle.com -Dhttps.proxyPort=80"
 32 else
 33   echo Proxy Off
 34   git config --global --unset http.https://github.com.proxy
 35   git config --global --unset https.https://github.com.proxy
 36   export http_proxy=
 37   export https_proxy=
 38 fi
 39 

{% endhighlight %}
