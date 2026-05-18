---
layout: post
title:  "OSINT Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RAMunchers-CTF-2026-OSINT/
---
* TOC
{:toc}

## Data Centre Hijack

**Author**: not given

**Description**: Gibson has attacked one of the Data Center’s of a significant company. Before their server was hijacked, the employees managed to send a drone picture of the building. Your job is to identify the location of this data server and the company that owns it.

The flag is in format: RMCTF{City-Country-Company}

I get the challenge file:

![Alt text](/images/data-center.jpg)

I find coordinates in the metadata:
```shell
exiftool data-center.jpg | tail  
Profile Copyright               : Google Inc. 2016  
Image Width                     : 1000  
Image Height                    : 416  
Encoding Process                : Baseline DCT, Huffman coding  
Bits Per Sample                 : 8  
Color Components                : 3  
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)  
Image Size                      : 1000x416  
Megapixels                      : 0.416  
GPS Position                    : 50 deg 28' 22.82", 3 deg 52' 3.34"
```

The location on google maps is at: [https://www.google.com/maps/place/Belgium+Google+data+center/@50.464142,3.7844992,11z/data=!4m12!1m5!3m4!2zNTDCsDI4JzIyLjgiTiAzwrA1MicwMy4zIkU!8m2!3d50.473006!4d3.867594!3m5!1s0x47c250c9f9463265:0x7528c15b02e15eaa!8m2!3d50.472747!4d3.867703!16s%2Fg%2F1td4cq35?entry=ttu&g_ep=EgoyMDI2MDUxMC4wIKXMDSoASAFQAw%3D%3D](https://www.google.com/maps/place/Belgium+Google+data+center/@50.464142,3.7844992,11z/data=!4m12!1m5!3m4!2zNTDCsDI4JzIyLjgiTiAzwrA1MicwMy4zIkU!8m2!3d50.473006!4d3.867594!3m5!1s0x47c250c9f9463265:0x7528c15b02e15eaa!8m2!3d50.472747!4d3.867703!16s%2Fg%2F1td4cq35?entry=ttu&g_ep=EgoyMDI2MDUxMC4wIKXMDSoASAFQAw%3D%3D)

`RMCTF{Saint-Ghislain-Belgium-Google}`

___

## AI or Real?

**Author**: not given

**Description**: An employee of the identified company (author of the picture from challenge 1) has recently reported an impersonator account stalking them on social media – we believe it may be Gibson posting AI versions of their posts. Using their real username: starring1367 can you find the fake account and the name of the event taking place in the picture of their most recent holiday.

The flag is in the format: RMCTF{username-Event Name}

I found the employees name in the metadata from the photo in the `Data Centre Hijack` OSINT challenge:
```shell
exiftool data-center.jpg | grep "Artist"  
Artist                          : Charlie Jacobs
```

By looking up `Charlie Jacobs` I find his instagram account `@starring1367`. I found an instagram account who tagged them, `@starr0256`.

They shared this photo about their holiday:

![Alt text](/images/005103_Instagram.jpg)

I put this photo into reverse image search and find the event is the famous `Brussels Flower Carpet`.

`RMCTF{starr0256-Flower Carpet}`

___

## Gibson makes a Mistake

**Author**: not given

**Description**: Gibson has disabled the employee account and is posting fake pictures of themselves at the AI-IM global summit 2026 - but they have made mistakes in each image of their event post leaving clues to their real location. Your job is to identify the country and region they are based in.

The flag is in the format: RMCTF{Region-Country}

They shared a picture of a car with the liscense plate number `CK-923-EJ`. There is also a `74` in the bottom right hand corner of the plate. 

The format `XX-000-XX` (two letters, three numbers, two letters) is the standard for plates in France. Searching for `france department code 74` returns the region `Haute-Savoie`.

`RMCTF{Haute-Savoie-France}`

___

## Office Switching

**Author**: Randomtoob & Jonathans

**Description**: An IT company allowed Gibson to "optimise" their office locations. Your goal is to find the flag that shows where the Infrastructure team has been moved to.

[https://bluepeakcyber.co.uk/](https://bluepeakcyber.co.uk/)

The flag is in the format of `RMCTF{XXXXXXXXXXXX}`

I find an html comment pointing to `internalit.bluepeakcyber` on `bluepeakcyber.co.uk`:
```shell
curl https://bluepeakcyber.co.uk/ | tail  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  8498    0  8498    0     0  10938      0 --:--:-- --:--:-- --:--:-- 11452  
</main>  
<!-- synced footers from internalit.bluepeakcyber -->  
	<footer>  
		<div class="container">  
			© 2026 BluePeak Cyber. All rights reserved.  
		</div>  
	</footer>  
</body>  
</html>
```

On `https://internalit.bluepeakcyber.co.uk/robots.txt`:
```shell
curl https://internalit.bluepeakcyber.co.uk/robots.txt  
User-agent: *  
Disallow: /memo.pdf
```

On `https://internalit.bluepeakcyber.co.uk/memo.pdf` I download the pdf and run `pdftotext`:
```shell
pdftotext memo.pdf

cat memo.txt  
BluePeak Cyber  
Internal Memo  
  
Internal IT Notice – DNS Record Review  
Date: 11 May 2026  
From: Internal IT Team  
Subject: DNS Record Notes and Temporary Retention  
  
The Internal IT team would like to advise staff that we have recently experienced a number of DNS-related issues affecting parts of our internal infrastructure and service resolution.  
As part of ongoing troubleshooting and remediation work, several legacy and temporary DNS records have intentionally been left in place within the environment. These records are currently being retained as operational notes and reference points to assist with diagnostics, rollback procedures, and service continuity during the review period.  
Please note the following:  
•  
  
Some DNS entries may appear duplicated, outdated, or non-standard.  
  
•  
  
Certain records are being preserved temporarily for documentation and audit  
purposes.  
  
•  
  
Internal systems should continue to function normally while remediation work is  
completed.  
  
•  
  
IT staff should avoid removing or modifying DNS entries unless specifically  
authorised by the Infrastructure or Network team.  
  
We are continuing to monitor the environment closely and will remove or consolidate unnecessary records once the review has been fully completed.  
If you encounter any unusual connectivity or name resolution issues, please report them through the Internal IT support process.  
Thank you for your cooperation and patience while these improvements are carried out.  
- Internal IT Team
```

First I look at TXT records for `bluepeakcyber.co.uk`:
```shell
dig TXT bluepeakcyber.co.uk  
  
; <<>> DiG 9.20.21-1~deb13u1-Debian <<>> TXT bluepeakcyber.co.uk  
;; global options: +cmd  
;; Got answer:  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3793  
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; MBZ: 0x0005, udp: 1232  
;; QUESTION SECTION:  
;bluepeakcyber.co.uk.           IN      TXT  
  
;; ANSWER SECTION:  
bluepeakcyber.co.uk.    5       IN      TXT     "v=spf1 +a +mx +ip4:149.255.62.126 include:relay.thundermail.uk ~all"  
bluepeakcyber.co.uk.    5       IN      TXT     "Legacy systems have been left running while the new infrastructure is under maintenance. To contact the infrastructure team get in contact with support@coventry.r032.bluepeakcyber.co.uk"  
  
;; Query time: 33 msec  
;; SERVER: 192.168.126.2#53(192.168.126.2) (UDP)  
;; WHEN: Wed May 13 01:22:58 EDT 2026  
;; MSG SIZE  rcvd: 326
```

Check records for `coventry.r032.bluepeakcyber.co.uk`:
```shell
dig TXT coventry.r032.bluepeakcyber.co.uk  
  
; <<>> DiG 9.20.21-1~deb13u1-Debian <<>> TXT coventry.r032.bluepeakcyber.co.uk  
;; global options: +cmd  
;; Got answer:  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33510  
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; MBZ: 0x0005, udp: 1232  
;; QUESTION SECTION:  
;coventry.r032.bluepeakcyber.co.uk. IN  TXT  
  
;; ANSWER SECTION:  
coventry.r032.bluepeakcyber.co.uk. 5 IN TXT     "RMCTF{DN5_1S_PUBLIC}"  
  
;; Query time: 40 msec  
;; SERVER: 192.168.126.2#53(192.168.126.2) (UDP)  
;; WHEN: Wed May 13 01:23:12 EDT 2026  
;; MSG SIZE  rcvd: 95
```

`RMCTF{DN5_1S_PUBLIC}`
