---
layout: post
published: false
title: 'Other than JSON and ORDS '
---
## REST enabling PL/SQL

ORDS has a feature named auto-plsql enablement. While this is the fastest way to access and leverage existing PLSQL logic via HTTP it is not 'REST'. REST implies much more and there's many answers and explaination out there such as [Stack Overflow](https://stackoverflow.com/questions/4663927/what-is-rest-slightly-confused) , [REST API Tutorial](http://www.restapitutorial.com/lessons/whatisrest.html), [Wikipedia](https://en.wikipedia.org/wiki/Representational_state_transfer), and [lots more](http://lmgtfy.com/?q=what+is+REST).


# ORDS API

The easiest way to enable a PLSQL object is to use our PLSQL API. Here's an example of enabling a procedure that has 1 IN/OUT which is  TIMESTAMP WITH LOCAL TIME ZONE.

{% highlight sql %}
DECLARE
  PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN

    ORDS.ENABLE_OBJECT(p_enabled => TRUE,
                       p_schema => 'KLRICE',
                       p_object => 'TIMESTAMPLTZ_INOUT_PROC',
                       p_object_type => 'PROCEDURE',
                       p_object_alias => 'timestampltz_inout_proc',
                       p_auto_rest_auth => FALSE);

    commit;

END;
{% endhighlight %}

## OpenAPI 

A new feature in ORDS 17.3 is the generation of [OpenAPI definitions](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md).  This is format from https://swagger.io/ for documentation of a REST API.  There are many tools that support this standard and can simply import the definition and offer a way to test the endpoint automatically.

# Open API for this procedure


{% highlight JavaScript %}
{"swagger":"2.0","info":{"title":"ORDS generated API for TIMESTAMPLTZ_INOUT_PROC","version":"1.0.0"},"host":"localhost:9090","basePath":"/ords/klrice/timestampltz_inout_proc","schemes":["http"],"produces":["application/json"],"paths":{"/":{"post":{"produces":["application/json"],"responses":{"200":{"description":"output of the endpoint","schema":{"type":"object","properties":{}}}}}}}}
{% endhighlight %}


## 
