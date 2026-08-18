---
layout: post
title:  "DFIR Challenges"
date:   2026-08-17 19:29:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /thryve-ctf-2026-dfir/
---
* TOC
{:toc}

## Monday Attack

**Category**: DFIR

**Author**: 𝔸𝕙𝕞𝕖𝕕 𝔸𝕝𝕨𝕖𝕕𝕪𝕒𝕟

**Description**: A SOC team detected unusual network activity originating from a workstation inside the environment.

The affected system appears to have communicated with an unknown host, accessed a suspicious resource, established repeated outbound connections, and potentially transferred sensitive data.

You have been provided with a packet capture collected during the incident.

Your task is to reconstruct the attack chain, identify the important Indicators of Compromise (IOCs), analyze the suspicious communication, and determine what data may have been transferred.

Not everything you need is inside the packets themselves. A previous analyst may have left something behind.

Q1 — Discovery

During the early stage of the incident, the compromised host discovered and later communicated with another system. What is the IP address of that system? Just IP

I get the challenge file:
```shell
unzip Monday-Attack.pcapng.zip  
Archive:  Monday-Attack.pcapng.zip  
  inflating: Monday-Attack.pcapng
```

I opened the pcap in wireshark. There was a lot of SSDP and MDNS packets.

I ran this `tshark -r Monday-Attack.pcapng -q -z conv,ip` in order to see which IPs talk to each other excluding multicast/broadcast. I discover the compromised host is `192.168.1.107`, and the system it discovered is `192.168.1.106`.

`192.168.1.106`

___

## Monday Attack - 2

**Category**: DFIR

**Author**: 𝔸𝕙𝕞𝕖𝕕 𝔸𝕝𝕨𝕖𝕕𝕪𝕒𝕟

**Description**: Q2 — Delivery

The compromised system made an HTTP request for a suspicious resource.

What URI was used to the suspicious request? Example: /test/

I found this request:
```http
GET /update/ HTTP/1.1
Host: 192.168.1.106:8080
User-Agent: curl/8.18.0
Accept: */*
```

`/update/`

___

## Monday Attack - 3

**Category**: DFIR

**Author**: 𝔸𝕙𝕞𝕖𝕕 𝔸𝕝𝕨𝕖𝕕𝕪𝕒𝕟

**Description**: Q3 — Command & Control

Following the suspicious HTTP activity, repeated connections were established with the remote system.

What destination port was used for Command and Control (C2) communication?

```shell
tshark -r Monday-Attack.pcapng -Y "tcp.port==4444" -T fields -e tcp.payload | xxd -r -p  
welcome with wedyan  
  
  
HOST=KALI-WS07  
STATUS=READY  
HELLO  
HOST=KALI-WS07  
STATUS=READY  
HELLO  
HOST=KALI-WS07  
STATUS=READY  
  
HELLO  
HOST=KALI-WS07  
STATUS=READY  
SESSION=8F21A  
ACTION=UPLOAD  
FILE=employee_backup.zip  
SIZE=2048  
STATUS=START  
ZHVtbXlfY3RmX2V4ZmlsdHJhdGlvbl9kYXRh  
STATUS=COMPLETE  

echo 'ZHVtbXlfY3RmX2V4ZmlsdHJhdGlvbl9kYXRh' | base64 -d  
dummy_ctf_exfiltration_data
```

`4444`

___

## Monday Attack - 4

**Category**: DFIR

**Author**: 𝔸𝕙𝕞𝕖𝕕 𝔸𝕝𝕨𝕖𝕕𝕪𝕒𝕟

**Description**: Q4 — Exfiltration

Further investigation of the C2 communication reveals what appears to be a file-transfer operation.

What was the name of the file targeted for exfiltration

I got the answer from the previous solution `tshark -r Monday-Attack.pcapng -Y "tcp.port==4444" -T fields -e tcp.payload | xxd -r -p`.

`employee_backup.zip`

___

## Monday Attack - 5

**Category**: DFIR

**Author**: 𝔸𝕙𝕞𝕖𝕕 𝔸𝕝𝕨𝕖𝕕𝕪𝕒𝕟

**Description**: 5 — Encoded Data

During the exfiltration stage, following encoded data transmitted through the suspicious, what is the data after decoding?

I got my answer for Q5 from the Q3 solution. 

`dummy_ctf_exfiltration_data`

___

## Monday Attack - 6

**Category**: DFIR

**Author**: 𝔸𝕙𝕞𝕖𝕕 𝔸𝕝𝕨𝕖𝕕𝕪𝕒𝕟

**Description**: Q5 — Final Analyst's The packets tell most of the story—but not all of it. During the original investigation, a SOC analyst left a flag. Can you retrieve it? `Thryve{...}`

```shell
exiftool Monday-Attack.pcapng  
ExifTool Version Number         : 13.25  
File Name                       : Monday-Attack.pcapng  
Directory                       : .  
File Size                       : 4.2 MB  
File Modification Date/Time     : 2026:08:06 20:08:18-04:00  
File Access Date/Time           : 2026:08:06 20:08:18-04:00  
File Inode Change Date/Time     : 2026:08:14 14:42:09-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PCAPNG  
File Type Extension             : pcapng  
MIME Type                       : application/vnd.tcpdump.pcap  
PCAP Version                    : PCAPNG 1.0  
Byte Order                      : Little-endian (Intel, II)  
Hardware                        : 13th Gen Intel(R) Core(TM) i5-1335U (with SSE4.2)  
User Application                : Dumpcap (Wireshark) 4.6.4  
Comment                         : Thryve{wedyan love you}  
Link Type                       : IEEE 802.3 Ethernet  
Device Name                     : eth0  
Time Stamp Resolution           : 1e-09  
Operating Sytem                 : Linux 6.18.12+kali-amd64  
Time Stamp                      : 2026:08:06 10:37:35.863824606-04:00
```

`Thryve{wedyan love you}`

___

## Signal Lost

**Category**: DFIR

**Author**: 𝐀𝐰𝐬 𝐀𝐟𝐚𝐧𝐞𝐡 — 𝐍𝐨𝐯𝐚𝟎𝐱

**Description**: A recovered bundle of corrupted surveillance stills was pulled from an abandoned relay node. Most of them are normal images. One of them is lying. Recover the final dispatch. Flag format: Thryve{...}

I get the challenge files:
```shell
unzip 'signal lost.zip'  
Archive:  signal lost.zip  
  inflating: _0f4d6d7c-8b1f-47d9-9c0d-1f1b7a5a1c41.jpg  
  inflating: _2bd3f369-46d0-4a60-9aa1-1f5d4e77d0fb.jpg  
  inflating: _4f7f51ac-6c9a-45f8-9904-a2a4550b4f58.jpg  
  inflating: _6b2a67aa-d3f4-45db-b4b8-986c7962b938.jpg  
  inflating: _8aa9f4d8-01c5-4385-b5ea-a87fb230fd26.jpg  
  inflating: _9d62c2a2-3b4c-4fb7-9f2d-16218b38f4ea.jpg  
  inflating: _c33758ae-dc47-4aab-a5bb-ef2d67db9d61.jpg  
  inflating: _e19a0c57-4c8f-4e2c-a113-5bf01af8e2dd.jpg  
  inflating: _f603cf07-1341-42f5-a58c-19743f8b182f.jpg  
  inflating: _fb8d2391-77b6-4916-96da-2ca2719c1517.jpg
```

`_e19a0c57-4c8f-4e2c-a113-5bf01af8e2dd.jpg` is a password protected pdf. Crack the password with `pdfcrack`:
```shell
pdfcrack -f _e19a0c57-4c8f-4e2c-a113-5bf01af8e2dd.jpg -w /usr/share/wordlists/rockyou.txt  
PDF version 1.4  
Security Handler: Standard  
V: 2  
R: 3  
P: -4  
Length: 128  
Encrypted Metadata: True  
FileID: 3938363066323932623532376361343136653131363133666130343831356463  
U: e143606359de954519387b6aecb7eb9328bf4e5e4e758a4164004e56fffa0108  
O: 37fa82c49f6ecd70c0896e50428697b6e016df22bc1e503d8b13aa577be82731  
found user-password: 'trustno1'
```

In the pdf I found this text:
```shell
Recovered pager fragment
The archive key is not written plainly.
Decode this signal three times and stop:
VFZWQ1RsZ3pVbTlOTVRsT1RraE9ja2xUUlQwPQ==
Nothing here is a screenshot. Nothing here is accidental.
```

Decode the base64 3 times:
```shell
echo "VFZWQ1RsZ3pVbTlOTVRsT1RraE9ja2xUUlQwPQ==" | base64 -d | base64 -d | base64 -d  
1@M_th3_M4sk!!
```

I ran strings in the image files and found this:
```shell
BEGIN_SIGNAL_BLOB  
Mzc3YWJjYWYyNzFjMDAwNDYwOWRjNTkyNjAwMTAwMDAwMDAwMDAwMDNmMDAwMDAwMDAwMDAwMDBmZmI0ZmQ3MmM5NWFkMTc4ZWE4NDlkOWIzYWQ1YTEyODM4ZTI1ZTViZTA2ZGEwODc0NTJkZjM4MjYwMmVlZGEwZTQ3NmMwMzYwZjk4OWVjYmZhOGMyMjExYzA2ODk4YzdlNmI5NDk3YjdjMDc3ZjYzNDE3YjQ1MWM2YzAxYTFhNzgyMjMyZWU3N2RjZTQ4MTIyMTQ0YjJmYTI3N2IyZDA0NWYwNjcwNWNhOTg5ODlmOGZhZGIzZGIwZmM1OGM1N2JhMzllNTg4MzcwZDI0MTE1YTFmZWI5OWY4NGYxODY1ZTg5YjAxMzU5NjllNDEwMGNkZDljOTFiMTg0MzhhZDUxMTJkNjk0ZDJjM2E3ZjY0OTFiMzdjYzE1MGNmOTM2YWQwNmJlMDk3ZmI1Mzk2NzQ4MmVhY2I1YWY2OTI3YmQzMzdlYzEwOWVjZmMxMmE1NmY2OTY5Mzg5NGJhZjViYTEzZDRjMGJlYWRlOTgwZWJkNGRlN2IyZWQyMjk5YmYxZDM0MjM1NjJlODUxZTI2MTg2N2VjMGI2ZWIxNzE1ODVkN2FiNWFkNDgyOTA2ZTUxMWY4MmRkNTNhZmQ3OTZjYjkyZjQ2OTY1MzNiZWI3YjcyZGEyZjNhYWQ4YmVmMmM4ODZiY2I5Njg4YWIyZWFhZjY0YmEwNzA0NDBlMTE0NDI4M2MyMmRlYTJmYjU2ZGVjMzYzYTQ3OWZjZGVlMzQxMTcxZTE1ODhhYTRhOWFkNTEwZWQxOTk5MzUxYjk1Zjc1ZWIwNGU2ZTc5MDQ2Njk2Mzc2MGZlMDJhN2I0NTc1NzFlZDAwMzc3MGEwNTgwMGY2MzFmODVjMGU2NThmMzEyOTNiNGVmZmUzN2MyODEwNGFjNTc1ODBiMTUwMDhlZjU4OWI2MDI1NzcwOWI3Zjg3NzU2ZWI1ZjllZTNlYTI5ZjhmYWNhNGExYWFlMTcwNjgwODAwMTA5ODBlMDAwMDcwYjAxMDAwMjI0MDZmMTA3MDExMjUzMGZiMDZjYjZjZDE5ZDk2NmUzYmM5NTgxMTNhMWFjNTZmNzIzMDMwMTAxMDU1ZDAwMTAwMDAwMDEwMDBjODBkMTgxOGUwYTAxOTRjZWMxZmQwMDAw  
END_SIGNAL_BLOB
```

This is a password protected `7z` file. I put it into cyberchef and download the file:
```shell
file randomthing.7z  
randomthing.7z: 7-zip archive data, version 0.4  

exiftool randomthing.7z  
ExifTool Version Number         : 13.25  
File Name                       : randomthing.7z  
Directory                       : .  
File Size                       : 447 bytes  
File Modification Date/Time     : 2026:08:14 15:28:08-04:00  
File Access Date/Time           : 2026:08:14 15:28:08-04:00  
File Inode Change Date/Time     : 2026:08:14 15:28:18-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : 7Z  
File Type Extension             : 7z  
MIME Type                       : application/x-7z-compressed  
File Version                    : 7z v0.04  
Warning                         : File is encrypted.
```
The password for this file is `1@M_th3_M4sk!!`.

Get the flag from the new files:
```shell
cd output  

cd thryve_forensics_signal_lost_20260813  

cd work  

cd vault  

file *  
fsociety_final.txt: ASCII text, with CRLF line terminators  
README.txt:         ASCII text, with CRLF line terminators  

cat README.txt  
Recovered relay cache. The final line matters.  

cat fsociety_final.txt  
final dispatch:  
Thryve{d0nt_trust_th3_f1l3_ext3ns10n_follow_th3_s1gnal}
```

`Thryve{d0nt_trust_th3_f1l3_ext3ns10n_follow_th3_s1gnal}`
