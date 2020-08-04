---
layout: post
published: true
title: 'SQLcl and OCI Cloud Shell'
date: '2020-08-04'

---

# Cloud Shell w/ SQLcl

SQLcl is now included with OCI Cloud Shell. 


## Autonomous DB wallets

Autonomous DB requires a wallet to connect. This can be done via the `oci` cli which also included with the cloud shell default.

The wallet can be generated quickly with the cli as follows. 

```

oci db autonomous-database generate-wallet --autonomous-database-id ocid1.autonomousdatabase.oc1.iad..... --file <filename> --password <password>

```

The OCID of the database is required which is easily found in the OCI Console.


### Automated wallet generation

The better way is to generate the wallets without having to go to a web console. 

This script prints all DBs in a compartment. which requires a compartment-id. Luckily for the "root" compartment this is the tenancy and the tenancy is an `env` variable auto populated in cloud shell.

` oci db autonomous-database list --compartment-id $OCI_TENANCY`

That list is a nicely formatted json document which has all the needed information. 

```

oci db autonomous-database list --compartment-id $OCI_TENANCY
{
  "data": [
    {
      "autonomous-container-database-id": null,
      "available-upgrade-versions": [],
      
```

To avoid the scrolling over a large json doc, I wrote a tiny `jq` format of the output and place it into `db.jq` 


```
# Print current instances for autonomous db
# 
#
cat > db.jq <<EOF
.data[] |
    ."db-name" ,
    "\t" + .id ,
    "\t" + ."db-name" + "_low" ,
    "\n\tSDW: " + ."connection-urls"."sql-dev-web-url",
    "\tAPEX: " + ."connection-urls"."apex-url",
    "\n\toci db autonomous-database generate-wallet --autonomous-database-id " +  .id + " --file mywallet.zip --password <YOURPASSWORD>",
    "\n\t sql -cloudconfig mywallet.zip admin@" +."db-name" + "_low"    
EOF
```

To use this simply pipe the output as follows

```
oci db autonomous-database list --compartment-id $OCI_TENANCY | jq  -r -f db.jq
```

The output is much easier to consume

```
$oci db autonomous-database list --compartment-id $OCI_TENANCY | jq  -r -f db.jq

DB202006231608
        ocid1.autonomousdatabase.oc1.iad.axxxxxxxxxxxkacqrs6ehh2tokwq
        DB202006231608_low

        SDW: https://Y3xxxxxXLO3Y-DB202006231608.adb.us-ashburn-1.oraclecloudapps.com/ords/admin/_sdw/?nav=worksheet
        APEX: https://Y3xxxxXLO3Y-DB202006231608.adb.us-ashburn-1.oraclecloudapps.com/ords/apex

        oci db autonomous-database generate-wallet --autonomous-database-id ocid1.autonomousdatabase.oc1. iad.axxxxxxxxxxxkacqrs6ehh2tokwq --file mywallet.zip --password <YOURPASSWORD>

         sql -cloudconfig mywallet.zip admin@DB202006231608_low
         
```         

Now just copy/paste the wallet generation sample and fill in the password and use the sql sample to connect.

```
$  oci db autonomous-database generate-wallet --autonomous-database-id ocid1.autonomousdatabase.oc1. iad.axxxxxxxxxxxkacqrs6ehh2tokwq --file mywallet.zip --password $WALLET_PASS
Downloading file  [####################################]  100%
kris_rice@cloudshell:~ (us-ashburn-1)$  sql -cloudconfig mywallet.zip admin@DB202006231608_low

SQLcl: Release 20.2 Production on Tue Aug 04 15:52:31 2020

Copyright (c) 1982, 2020, Oracle.  All rights reserved.

Operation is successfully completed.
Operation is successfully completed.
Using temp directory:/tmp/oracle_cloud_config557139830701085193
Password? (**********?) *****************
Last Successful login time: Tue Aug 04 2020 15:52:39 +00:00

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.5.0.0.0


SQL> 
```
