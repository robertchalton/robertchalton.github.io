---
layout: post
published: true
title: Introducing SQL Developer - web
date: '2018-05-09'
image: /img/lets__encrypt.png
---

## Let's Encrypt

Probably the easiest way to get an SSL Certificate these days is with the Free Let's Encrypt. Let's Encrypt is currently creating ~600k SSL certificates a day according to their [statistics](https://letsencrypt.org/stats/)

## ORDS Standalone and SSL

ORDS standalone mode will create a self-signed certificate upon install just to get something in place. This certificate should clearly never be used other than development. When going to production, there should be a real certificate aquired and installed.

Installing a real certificate is quite straight forward and in the [documentation](https://docs.oracle.com/en/cloud/paas/database-dbaas-cloud/csdbi/administer-ords.html#GUID-CFF2853B-1375-4F71-9600-69347A47A291)

1. Aquire a Certificate

2. (optional) If the key file is in .pem format, create a .der file 

		```
		openssl pkcs8 -topk8 -inform PEM -outform DER -in example.com.pem -out example.com.der -nocrypt
		```


3. Edit the ords/standalone/standalone.properties

		``` 
		ssl.cert=/u01/app/oracle/product/ords/conf/ords/standalone/example.com.crt
		ssl.cert.key=/u01/app/oracle/product/ords/conf/ords/standalone/example.com.der
		```
> This is the path in Oracle Cloud DBCS that is the ORDS configuration /u01/app/oracle/product/ords/conf/ords/standalone/

4. Restart ORDS. 

## Let's Encrypt Setup Options

There's lots of prebuilt integrations that Let's Encrypt. For example, if the ORDS install is being front ended with Nginx or Apache httpd there's a couple choices with the implementations listed on the [client-optiosn page](https://letsencrypt.org/docs/client-options/) or EFF's [Certbot](https://certbot.eff.org/) 


## GetSSL
Since there's no prebuilt tool for dealing with ORDS. I'll be using  [GetSSL](https://github.com/srvrco/getssl) to aquiring a cert. 

These are just my steps and probably other ways to get it done.

* Install   

	```
	curl --silent https://raw.githubusercontent.com/srvrco/getssl/master/getssl > getssl ; chmod 700 getssl
	```

* Initialize the configuration files

	```./getssl -c demo.krisrice.io```

* Edit the configuration to site specifics.

	In the file ~/.getssl/demo.krisrice.io/getssl.cfg , These are the only settings I adjusted.

	```
	PRIVATE_KEY_ALG="rsa"
	
	ACL=('/u01/app/oracle/product/ords/conf/ords/standalone/doc_root/.well-known/acme-challenge')
	```

> This is the path in Oracle Cloud DBCS that is the docroot for ORDS /u01/app/oracle/product/ords/conf/ords/standalone/doc_root/

* Clean up the self-signed certificates  

	```
	rm /u01/app/oracle/product/ords/conf/ords/standalone/self-signed.key	
	rm /u01/app/oracle/product/ords/conf/ords/standalone/self-signed.pem
	```

* Convert the Let's Encrypt .key file to a .der that ORDS can use

	```
	openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in /u01/app/oracle/product/ords/conf/ords/standalone/example.com.key -out /u01/app/oracle/product/ords/conf/ords/standalone/example.com.pkcs8.key
	
	openssl pkcs8 -topk8 -inform PEM -outform DER -in /u01/app/oracle/product/ords/conf/ords/standalone/example.com.pkcs8.key -out /u01/app/oracle/product/ords/conf/ords/standalone/example.com.pkcs8.der -nocrypt
	
	# remove temp file
	rm /u01/app/oracle/product/ords/conf/ords/standalone/example.com.pkcs8.key
	
	```	


* Configure ORDS. This is now the same as setting ORDS to using any SSL certificate as mentioned in the documentation.	

	Edit /u01/app/oracle/product/ords/conf/ords/standalone/standalone.properties
	
	```
	ssl.cert=/u01/app/oracle/product/ords/conf/ords/standalone/example.com.crt
	ssl.cert.key=/u01/app/oracle/product/ords/conf/ords/standalone/example.com.pkcs8.der
	```




## End

The end result is a valid SSL certificate in the Standalone mode of ORDS

![](/img/lets__encrypt.png)
