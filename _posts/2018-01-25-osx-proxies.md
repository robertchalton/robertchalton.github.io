---
layout: post
published: true
title: OSX Proxies
date: '2018-01-30'
---
## Swapping proxies 

I frequently switch from inside a network that needs a proxy to being on public internet.  Swapping the proxy on/off is less than easy for things like github so here's the script I use.

The script uses the osx utility scutil with --proxy to get the current proxy configuration. Then it's a simple if/then to set to unset various proxy settings.


{% highlight shell %}
  proxy=`scutil --proxy | grep ProxyAutoConfigEnable | awk -F: '{print $2}'` 
  echo Proxy  :$proxy:
  if [[ "$proxy" == " 1" ]] ; then
    echo Proxy On
     git config --global https.https://github.com.proxy http://ourcorpproxy.com:80
     git config --global http.https://github.com.proxy http://ourcorpproxy.com:80
     export http_proxy=http://ourcorpproxy.com:80/
     export https_proxy=http://ourcorpproxy.com:80/
     export MAVEN_OPTS="-Dhttp.proxyHost=ourcorpproxy.com -Dhttp.proxyPort=80 -Dhttps.proxyHost=ourcorpproxy.com -Dhttps.proxyPort=80"
  else
    echo Proxy Off
    git config --global --unset http.https://github.com.proxy
    git config --global --unset https.https://github.com.proxy
    export http_proxy=
    export https_proxy=
  fi
  
{% endhighlight %}
