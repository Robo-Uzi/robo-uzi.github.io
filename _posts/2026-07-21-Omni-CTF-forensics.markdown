---
layout: post
title:  "QuackQuackDiriDiriDuck"
date:   2026-07-21 17:49:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /omni-CTF-2026-QuackQuackDiriDiriDuck/
---

Title: QuackQuackDiriDiriDuck

Author: From the Romanian folklore

Category: Forensics

Description: Forensics/Malware all the files are present in the zip. DISCLAIMER: This is real MALWARE don't run it on your computer without the proper tools!! Password: `infected`

___

How many emails have malicious PDFs? (Example: `OmniCTF{67}`)

Using cyberchef I took the 5 email files (`Email<1-5>.eml`) and looked at the attached pdfs. They each contained a download button which lead to a sketchy domain which downloads a zip file:

![Alt text](/images/omniforensics1.png)

`OmniCTF{5}`

___

Arrange the emails the attacker used to send the PDFs alphabetically from a -> z? (Example: `OmniCTF{abc@andydwdhhdd.net+test@omnictf.com}`)

Get the emails from the headers of the `.eml` files.

`OmniCTF{anderson@hexli.com+bounce-77291@mailer-node.example+michellemay@aryanproo.co.ke+ventas@castillo-lane.example+xaney@emplo.com}`

___

What is the first malicious HTTP download URL in the PCAP? (Example: `OmniCTF{url}`)

```shell
tshark -r 2023-05-24-obama264-Qakbot-infection.pcap -Y "http"  
    8   0.186265  10.5.24.101 → 160.153.53.37 HTTP 518 GET /ewukhyqpjz/ewukhyqpjz.zip HTTP/1.1  
   36   0.832587 160.153.53.37 → 10.5.24.101  HTTP 672 HTTP/1.1 200 OK  (application/zip)  
   91  74.141492  10.5.24.101 → 45.76.58.72  HTTP 384 GET /aKUVYL8o0uv.dat HTTP/1.1  
... etc
```

`OmniCTF{http://adubuildersco.com/ewukhyqpjz/ewukhyqpjz.zip}`

___

How many unique remote IP-and-port combinations use the Qakbot TLS client fingerprint? (Example: `OmniCTF{67}`)

Eventually I found something suspicious:
```shell
tshark -r 2023-05-24-obama264-Qakbot-infection.pcap -Y "tls.handshake.type == 1" -T fields -e tls.handshake.ja3 -e ip.dst -e tcp.dstport | sort -u | grep "2222"  
43016d7f7f9336b17c884650d0d2545d        142.118.221.248 2222
```

I looked for `tls.handshake.ja3 == 43016d7f7f9336b17c884650d0d2545d` in wireshark and found 4 unique IP, port combos. 

`OmniCTF{4}`

___

Identify all unique Qakbot TLS C2 endpoints, separated by `+`, in ascending order. (Example: `OmniCTF{192.168.11.0:80+192.168.11.3:443}`)

```shell
tshark -r 2023-05-24-obama264-Qakbot-infection.pcap -Y "tls.handshake.ja3 == 43016d7f7f9336b17c884650d0d2545d" -T fields -e ip.dst -e tcp.dstport | sort -u  
142.118.221.248 2222  
185.81.114.188  443  
188.28.19.84    443  
201.130.154.90  443
```
`OmniCTF{142.118.221.248:2222+185.81.114.188:443+188.28.19.84:443+201.130.154.90:443}`

___

At what timestamp is the malicious ZIP requested? (Example: `OmniCTF{YYYY-MM-DD HH:MM:SS}`)

This is the request:
```http
GET /ewukhyqpjz/ewukhyqpjz.zip HTTP/1.1
Host: adubuildersco.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36 Edg/113.0.1774.42
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9

  
HTTP/1.1 200 OK
Date: Wed, 24 May 2023 16:34:14 GMT
Server: Apache
X-Powered-By: PHP/7.1.33
Connection: keep-alive, Keep-Alive
Accept-Ranges: bytes
Expires: 0
Cache-Control: no-cache, no-store, must-revalidate
Content-Disposition: attachment; filename="ewukhyqpjz.zip"
Upgrade: h2,h2c
Connection: Upgrade
Content-Length: 28178
Vary: Accept-Encoding
Keep-Alive: timeout=5
Content-Type: application/zip

... etc zip data
```

I find this:
```shell
tshark -r 2023-05-24-obama264-Qakbot-infection.pcap -Y 'http.request.uri contains "ewukhyqpjz.zip"' -T fields -e frame.time  
May 24, 2023 12:34:13.441341000 EDT  
May 24, 2023 12:34:14.087663000 EDT
```
Convert to UTC by adding 4 hours.

`OmniCTF{2023-05-24 16:34:13}`

___

At what timestamp is the Qakbot DLL requested? (Example: `OmniCTF{YYYY-MM-DD HH:MM:SS}`)

```shell
tshark -r 2023-05-24-obama264-Qakbot-infection.pcap -Y 'http.request.uri contains "aKUVYL8o0uv.dat"' -T fields -e frame.time  
May 24, 2023 12:35:27.396568000 EDT  
May 24, 2023 12:35:28.454315000 EDT
```

`OmniCTF{2023-05-24 16:35:27}`

___

How many seconds pass between DLL GET and first C2 ClientHello? (Example: `OmniCTF{6.767}`)

```shell
tshark -r 2023-05-24-obama264-Qakbot-infection.pcap -Y "tls.handshake.type == 1 and (ip.dst == 142.118.221.248 or ip.dst == 185.81.114.188 or ip.dst == 188.28.19.84 or ip.dst == 201.130.154.90)" -T fields -e frame.time_relative -e ip.dst -e tcp.dstport | sort -n | head  
414.529394000   188.28.19.84    443  
420.250981000   142.118.221.248 2222  
426.238154000   142.118.221.248 2222  
491.980525000   142.118.221.248 2222  
492.578483000   142.118.221.248 2222  
495.353309000   142.118.221.248 2222  
676.412845000   142.118.221.248 2222  
856.755408000   142.118.221.248 2222  
1036.952322000  142.118.221.248 2222  
1217.701002000  142.118.221.248 2222  

tshark -r 2023-05-24-obama264-Qakbot-infection.pcap -Y 'http.request.uri contains "aKUVYL8o0uv.dat"' -T fields -e frame.time_relative | head -1  
74.141492000
```

```shell
414.529-74.141=340.388
```

`OmniCTF{340.388}`

___

What malware family is represented? (Example: `OmniCTF{Fam}`)

`OmniCTF{QAKBOT}`

___

What is the malware campaign/variant identifier? (Example: `OmniCTF{Mal}`)

I found info on the campaign here: [https://www.malware-traffic-analysis.net/2023/05/24/index.html](https://www.malware-traffic-analysis.net/2023/05/24/index.html)

`OmniCTF{OBAMA264}`