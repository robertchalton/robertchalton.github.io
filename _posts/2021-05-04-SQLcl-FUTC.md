---
layout: post
published: false
title: 'SQLcl License Change'
date: '2021-05-04'

---

# SQLcl: The modern command line

SQLcl is the modern command line interface for the Oracle Database. It has been adding new features every release including recent additions such as comprehensive liquibase support for the Oracle Database and OCI Cloud Storage. The easiest way to see most of the additions are seen by issuing the help command which highlight the new features over the basics. The additions not seen are things like command completion, multi-line buffer editing, and robust history. The most powerful feature is the script command which enables the usage of javascript / Graal mixed in with sql scripts. It also allows for the creation of commands to make the tool truly customizable.

![](/img/sqlcl-help.png)


 # Access to SQLcl

 SQLcl is distributed via the product page on [oracle.com](https://www.oracle.com/tools/downloads/sqlcl-downloads.html) and bundled with [SQL Developer](https://www.oracle.com/database/technologies/appdev/sqldeveloper-landing.html), Oracle Database, OCI Cloud Shell, OCI Linux Yum Repository , and the [OCI Developer Image](https://console.us-ashburn-1.oraclecloud.com/marketplace/application/54030984/overview) in the marketplace.

 These channels are all subjected to either the Click-Thru license or the overall OCI Cloud license in the case of cloud shell and OCI Yum.


## SQLcl in Cloud Shell

SQLcl is out of box installed in cloud shell and ready to use.

![](/img/sqlcl-cloudshell.png)

## SQLcl in OCI Linux Yum

Any compute node in Oracle Cloud can issue a simple `yum install sqlcl` to have the tool installed. This also applies for Oracle REST Data Services with `yum install ords` The releases in this repository are updated as releases are loaded to Oracle.com

![](/img/sqlcl-oci-yum.png)



# SQLcl: Easiest Access

SQLcl has just changed it's licensing to the [Oracle Free Use Terms and Conditions](https://www.oracle.com/downloads/licenses/oracle-free-license.html).

That means the tool can now be downloaded without a manual click to agree. This change will make automated downloads possible.

The location on oracle.com will be unchanged to discover the links on [the download page](https://www.oracle.com/tools/downloads/sqlcl-downloads.html). The release that has gone live today is  
https://download.oracle.com/otn_software/java/sqldeveloper/sqlcl-21.1.1.113.1704.zip

There will also be a `sqlcl-latest.zip` which will always be the most current release. This will assist in making it as easy as possible to download the latest released version of the tool in an automated way. 

https://download.oracle.com/otn_software/java/sqldeveloper/sqlcl-latest.zip


![](/img/sqlcl-via-curl.png)
