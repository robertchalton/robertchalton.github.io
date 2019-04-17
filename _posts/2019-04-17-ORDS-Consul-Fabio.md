---
layout: post
published: true
title: 'Load Balancing ORDS with Consul and Fabiolb'
date: '2018-10-22'

---
# Load Balancing 

Load balancing serves a few purposes the most common is redudnancy in case one node goes down the other other(s) are still present to take over the load. Another is to scale out and have more smaller resources vs scale up to have less but larger resources.  Oracle and customers have deployed ORDS in multiple ways with load balancers that are software based solutions such as WLS clustering or NGINX to hardware based solutions such as Big-IP F5 which is specialized hardware. 


# ORDS

For this excercise, there is nothing special done to ORDS at all. The following steps will work on any ORDS  configuration.

# Consul

[Consul](https://www.hashicorp.com/products/consul) is an open source project from HashiCorp for tracking services and their health across the datacenter.  This could be used to track any service from sqlnet to http.  It will then perform health checks to monitor when a service is available or not. There's a lot more features in the product which could be applied to ORDS such as a key/value store for configuration but this focus is on the service and it's health.

It may help to take a quick look over the [architecture](https://www.consul.io/docs/internals/architecture.html) of Consul. This example is not going to take full advantage as it will be on a single node ( my laptop ) and in development mode. 

Since this example is on an OSX machine installing consul is made easy with `brew`. Other install options are available on Hashicorp's site on the [Packaged Binaries](https://www.consul.io/downloads.html) page.

`brew install consul`

Next consul is going to run in developer mode which is purely for development. 

`consul agent -dev`


The first thing printed is how to access consul. The default is on port 8500.

```

kriss-MacBook-Pro:~ klrice$ consul agent -dev
==> Starting Consul agent...
==> Consul agent running!
           Version: 'v1.4.4'
           Node ID: '4bf186f5-79ac-b447-2101-eee6a0fd2f70'
         Node name: 'kriss-MacBook-Pro.local'
        Datacenter: 'dc1' (Segment: '<all>')
            Server: true (Bootstrap: false)
       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600)
      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false

```

There is a comprehensive REST API available on that port and this post will focus solely on the [service](https://www.consul.io/api/agent/service.html) API which is used to register and deregister the ORDS services. 

The payload of registering contains and ID which must be unique across all consul services. The first ords registered will use an ID following service-host-port or `ords-local-9090`  The service address, port, and health check are also in the payload.  The health check will be by accessing a rest end point of `/health/check` which is simple REST API made in ORDS that does a `select 1 from dual`.  The health check expects any HTTP 2xx range status code to be deemed healthy and any other status is considered unhealthy.  The only exception is a HTTP 429 which is a warning of too frequent of health checks.  If a service is marked unhealthy, it will not be included in any inventory of services for example in the next step the FabioLB would not use that un healthy service.


The payload will be >
```
{
  "ID": "ords-local-9090",
  "Name": "ords",
  "Tags": [
    "primary",
    "v1",
    "urlprefix-/ords",
    "urlprefix-/i"
  ],
  "Address": "127.0.0.1",
  "Port": 9090,
  "Meta": {
    "ords_version": "19.2.0"
  },
  "Check": {
    "DeregisterCriticalServiceAfter": "5m",
    "HTTP": "http://localhost:9090/ords/klrice/health/check",
    "Interval": "5s"
  }
}
```

Once the payload is defined, it can be registered via the Consul REST API as follows:

`curl --verbose  --request PUT     --data @payload.json     http://127.0.0.1:8500/v1/agent/service/register`

The ORDS service is now registered and viewable in the Consul web UI.

![](/img/ords_consul.png)

Consul now has an inventory of all ORDS services.  These steps were repeated for another ORDS running on port `9091`.

# FabioLB

[FabioLB](https://github.com/fabiolb/fabio) auto configures from the services and metadata registered in Consul.  The key is the `Tags` from the service registration in the last step.  

The `urlprefix-` tags are read by FabioLB which for ords is /ords for all database access and /i for APEX's images. The full configuration options are in the Fabio [documentation](https://fabiolb.net/quickstart/) which include hostname,protocol, ssl certificates,non-http,....

Again I used brew to install for simplicity and there are many other [options](https://github.com/fabiolb/fabio/releases) on the project Release page.


`brew install fabio`


Start fabio by running it directly on the command line:
`fabio`

The ADMIN port and the HTTP port are listed in the console at startup.  The defaults are to have ADMIN on port `9998` and the HTTP LB/Proxy on `9999`



Upon starting the configuration is printed to the console. The output below show there are 2 ORDS nodes and each have /ords and /i published via the urlprefix in the tags during registration.

```
2019/04/17 14:19:03 [INFO] Config updates
+ route add ords /ords http://127.0.0.1:9090/ tags "primary,v1"
+ route add ords /i http://127.0.0.1:9090/ tags "primary,v1"
2019/04/17 14:19:18 [INFO] Config updates
+ 1/ tags "primary,v1"
+ route add ords /ords http://127.+ .0.1:9090/ tags "primary,v1"
+ route add ords /i http://127.0.0.1:9091

```

This can also be seen in Fabio's Web UI. The 2 ORDS instances are loaded and given equal weight for load balancing purposes seen in the last column. 

![](/img/ords_fabio.png)

# All together

The result is now there is a port 9999 being serviced by Fabio and auto configured to the 2 registered ORDS servers running on ports 9090 and 9091. 

![](/img/ords_consul_fabio.png)


# Next Steps

- Make a docker image that has ords and consul in every image. The start process will register with the local Consul agent which would join overall Consul Cluster.
- SSL  https://fabiolb.net/feature/certificate-stores/
- SQL*NET proxy  https://fabiolb.net/feature/tcp-proxy/ 

