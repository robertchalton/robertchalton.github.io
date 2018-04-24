---
layout: post
published: true
title: ORDS Standalone and URI Rewrites
date: '2018-04-24'

---
# Adding a Simple URL Rewrite

My last post How to add an [NCSA style Access Log](http://krisrice.io/2017-01-12-how-to-add-ncsa-style-access-log-to/) to ORDS Standalone explained what the ORDS standalone is and that is based on Eclipse Jetty.  Jetty offers far more than ORDS exposed in it's standalone.  There's a long list of all the features and configuration options listed in the documentation, [http://www.eclipse.org/jetty/documentation/9.2.21.v20170120/](http://www.eclipse.org/jetty/documentation/9.2.21.v20170120/)

A recent question came up for doing URL rewrites.  Jetty does offer this as well.  To take advantage of it the same jetty-http.xml file from my last post just needs a few more lines of xml added.

This example will be just a simple one that rewrites /catalog to /ords/klrice/metadata-catalog/ The better usage of this would be to have / redirect into an APEX application or some home page of the application.

The full list of options are listed in the Jetty Documentation 
[http://www.eclipse.org/jetty/documentation/9.2.21.v20170120/rewrite-handler.html
](http://www.eclipse.org/jetty/documentation/9.2.21.v20170120/rewrite-handler.html)


```
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">
<Configure id="Server" class="org.eclipse.jetty.server.Server">
<!-- REWRITE -->
  <Get id="oldhandler" name="handler"/>
  <Set name="handler">
   <New id="Rewrite" class="org.eclipse.jetty.rewrite.handler.RewriteHandler">
    <Set name="handler"><Ref refid="oldhandler"/></Set>
    <Set name="rewriteRequestURI"><Property name="rewrite.rewriteRequestURI" default="true"/></Set>
    <Set name="rewritePathInfo"><Property name="rewrite.rewritePathInfo" default="false"/></Set>
    <Set name="originalPathAttribute"><Property name="rewrite.originalPathAttribute" default="requestedPath"/></Set>
   </New>
  </Set>
  <Ref refid="Rewrite">
<!-- REWRITE of /catalog ---- /ords/klrice/metadata-catalog/ -->
<Call name="addRule">          <Arg>            <New class="org.eclipse.jetty.rewrite.handler.RewritePatternRule">              <Set name="pattern">/catalog</Set>              <Set name="replacement">/ords/klrice/metadata-catalog/</Set>            </New>          </Arg>        </Call> </Ref>
<!-- HTTP ACCESS LOGS -->
  <Ref id="Handlers">
    <Call name="addHandler">
      <Arg>
        <New id="RequestLog" class="org.eclipse.jetty.server.handler.RequestLogHandler">
          <Set name="requestLog">
            <New id="RequestLogImpl" class="org.eclipse.jetty.server.NCSARequestLog">
              <Set name="filename"><Property name="jetty.logs" default="/tmp/"/>ords-access-yyyy_mm_dd.log</Set>
              <Set name="filenameDateFormat">yyyy_MM_dd</Set>
              <Set name="retainDays">90</Set>
              <Set name="append">true</Set>
              <Set name="extended">false</Set>
              <Set name="logCookies">false</Set>
              <Set name="LogTimeZone">GMT</Set>
            </New>
          </Set>
        </New>
      </Arg>
    </Call>
  </Ref>
</Configure>

```


The result is as expected.  This could be used for anything from shorter REST points like this catalog example or nicer entry points into APEX application.

![ScreenShot](https://4.bp.blogspot.com/-K9h2K80cAxM/WO-bIpvUlZI/AAAAAAAABPs/T_at8DO3ue0spwrTywhIBiqBKWkD6gaZwCK4B/s640/Screen%2BShot%2B2017-04-13%2Bat%2B11.36.30%2BAM.png)



The example for an APEX URL would be:

```
<!-- REWRITE of /awesomeapp ---- /ords/f?p=105:2:::::: -->

<Call name="addRule">
 <Arg>
   <New class="org.eclipse.jetty.rewrite.handler.RewritePatternRule">
     <Set name="pattern">/awesomeapp</Set>
     <Set name="replacement">/ords/f?p=105:2::::::/</Set>
   </New>
 </Arg>
</Call>
```

