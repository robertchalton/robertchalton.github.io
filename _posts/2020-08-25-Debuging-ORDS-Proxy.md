---
layout: post
published: true
title: 'Debugging ORDS Proxy Connection Issues'
date: '2020-08-25'

---

## ORA-28150: proxy not authorized to connect as client 

This is the message from the db for a number of things that could have gone wrong with the ability of ORDS to perform it's job. OR it could also be `ORA-01017: invalid username password; logon denied`

The way ORDS works is there's the connection pool user `ORDS_PUBLIC_USER` this is a common db account for all connections. The alternative would be to have a connection pool PER schema that wants to use ORDS' features. This would not scale very well in a large system

Enabling a schema for ords is done via `begin ords.enable_schema; end;` This performs the grant to allow the ords_public_user to proxy to say `KLRICE`.  The command issued would be 

```
ALTER USER KLRICE GRANT CONNECT THROUGH ORDS_PUBLIC_USER;
```

With the revoke being

```
ALTER USER KLRICE REVOKE CONNECT THROUGH ORDS_PUBLIC_USER;
```

When this goes wrong for various reasons there could be a couple ORA- errors which make it a tad difficult to unravel.


Now anyone that open the Oracle Documentation will see using proxy is as simple as `conn ORDS_PUBLIC_USER[klrice]`. This will test that ORDS_PUBLIC_USER can authenticate and proxy to klrice. However it's never that simple. This is a different code path in the client and in the db server side. ORDS uses a JDBC function call named `openProxySession`. The key differece is in the name as it's opening a new session not a new connection. This is a new [OCISession](https://docs.oracle.com/database/121/LNOCI/oci16rel001.htm#LNOCI17114) inside the same ords_public_user's db process. The full doc of JDBC using Proxy is [here](https://docs.oracle.com/en/database/oracle/oracle-database/19/jjdbc/proxy-authentication.html#GUID-07E0AF7F-2C9A-42E9-8B99-F2716DC3B746) 


## Enter SQLcl ( again..)

Since SQLcl is java based and we have the scripting support this is quite easy to test. 

This .sql script does 

* 	A normal connect to ORDS_PUBLIC_USER
* [	select to show the session user ] 
*  The jdbc/java code required to open the proxy
* [	select to show the session user ] 
* The jdbc/java code required to close the proxy


```
conn ords_public_user/oracle

select user from dual
/

REM the test for openProxySession
script
    var Properties = Java.type("java.util.Properties");
    var OracleConnection= Java.type("oracle.jdbc.OracleConnection");

    var prop = new Properties();
    prop.put(OracleConnection.PROXY_USER_NAME, "KLRICE");
    conn.openProxySession(OracleConnection.PROXYTYPE_USER_NAME, prop);

    // post proxy
    ctx.write('proxied\n');
/


select user from dual
/

script
    var OracleConnection= Java.type("oracle.jdbc.OracleConnection");
  conn.close(OracleConnection.PROXY_SESSION);

  // un-proxy
  ctx.write('un-proxied');

/

select user from dual
/

```


### Run it

I saved the script above and named it `simple_ord_public_user_proxy_test.sql` 

The output is quite easy to follow with a display of the session's username at each step.

```

KLRICE@xe🍻🍺 >@simple_ord_public_user_proxy_test.sql
Connected.

               USER
___________________
ORDS_PUBLIC_USER

proxied

     USER
_________
KLRICE

un-proxied

               USER
___________________
ORDS_PUBLIC_USER

```
