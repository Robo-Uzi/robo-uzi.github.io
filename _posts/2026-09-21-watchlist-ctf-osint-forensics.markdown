---
layout: post
title:  "OSINT and Forensics Challenges"
date:   2026-09-21 17:27:00 -0400
author: uzi
tags: [CTF]
permalink: /watchlist-ctf-2026-osint-forensics/
---
* TOC
{:toc}
{% raw %}
## Carter's Note

**Category**: forensics

**Description**: Detective Joss Carter was killed in November of last year. Her death is filed as a closed homicide. It is not.

Her effects included a USB drive. I have imaged it. It contains the public surface of her life: case notes, family photographs, her son's school papers.

It also contains four files she deleted before she was killed. She was a careful detective. She did not delete by accident.

One of those files is the work. The others are her cover. Find what she did not want them to find.

I download the challenge file:
```shell
file challenges/forensics/F100/usb-0412-a.img  
challenges/forensics/F100/usb-0412-a.img: DOS/MBR boot sector, code offset 0x58+2, OEM-ID "mkfs.fat", sectors/cluster 8, Media descriptor 0xf8, sectors/track 63, heads 16, sectors 614376 (volumes > 32 MB), FAT (32 bit), sectors/FAT 600, serial number 0xa410d1b, label: "USB-0412-A "
```

I used `fatcat` to check the image:
```shell
fatcat challenges/forensics/F100/usb-0412-a.img -l / -d  
Listing path /  
Directory cluster: 2  
d 1/6/2026 00:21:56  case_files/ (CASE_F~1)                             c=3  
d 1/6/2026 00:22:24  reference/ (REFERE~1)                              c=10  
d 1/6/2026 00:21:56  family/ (FAMILY)                                   c=12  
d 1/6/2026 00:28:16  personal/ (PERSONAL)                               c=17  
d 1/6/2026 00:27:12  HR_investigation/ (HR_INV~1)                       c=20  
f 1/6/2026 00:28:16  readme.txt (README.TXT)                            c=6695 s=43 (43B)
```

I found 5 deleted files:
```shell
fatcat challenges/forensics/F100/usb-0412-a.img -l "/case_files/2024" -d  
Listing path /case_files/2024  
Directory cluster: 4  
d 1/6/2026 00:21:56  ./ (.)                                             c=4  
d 1/6/2026 00:21:56  ../ (..)                                           c=3  
f 1/6/2026 00:21:58  04-2783_Robbery.pdf (04-278~1.PDF)                 c=25 s=135255 (132.085K)  
f 1/6/2026 00:21:58  04-2891_Assault.pdf (04-289~1.PDF)                 c=59 s=134495 (131.343K)  
f 1/6/2026 00:21:58  04-3052_Homicide.pdf (04-305~1.PDF)                c=92 s=135744 (132.562K)  
f 1/6/2026 00:22:00  04-3155_Burglary.pdf (04-315~1.PDF)                c=126 s=133601 (130.47K)  
f 1/6/2026 00:22:00  04-3201_Assault-DV.pdf (04-320~1.PDF)              c=159 s=134551 (131.397K)  
f 1/6/2026 00:22:00  04-3344_GrandLarceny.pdf (04-334~1.PDF)            c=192 s=132485 (129.38K)  
f 1/6/2026 00:22:00  04-3422_Robbery.pdf (04-342~1.PDF)                 c=225 s=133311 (130.187K)  
f 1/6/2026 00:22:02  04-3501_AggravatedAssault.pdf (04-350~1.PDF)       c=258 s=134881 (131.72K)  
f 1/6/2026 00:22:02  04-3578_Homicide.pdf (04-357~1.PDF)                c=291 s=132854 (129.74K)  
f 1/6/2026 00:22:02  04-3640_Burglary.pdf (04-364~1.PDF)                c=324 s=136716 (133.512K)  
f 1/6/2026 00:22:04  04-3712_Robbery.pdf (04-371~1.PDF)                 c=358 s=135354 (132.182K)  
f 1/6/2026 00:22:04  04-3789_Homicide.pdf (04-378~1.PDF)                c=392 s=136415 (133.218K)  
f 1/6/2026 00:22:04  04-3855_Larceny.pdf (04-385~1.PDF)                 c=426 s=133690 (130.557K)  
f 1/6/2026 00:22:06  04-3920_Assault.pdf (04-392~1.PDF)                 c=459 s=135054 (131.889K)  
f 1/6/2026 00:22:06  04-4001_Robbery.pdf (04-400~1.PDF)                 c=492 s=131942 (128.85K)  
f 1/6/2026 00:22:06  case_summary_2024.txt (CASE_S~1.TXT)               c=525 s=525 (525B)  
f 1/6/2026 00:28:16  draft_resignation.docx (RAFT_~1.DOC)               c=6697 s=284 (284B) d  
f 1/6/2026 00:28:16  note_0847.txt (OTE_0~1.TXT)                        c=6700 s=665 (665B) d

fatcat challenges/forensics/F100/usb-0412-a.img -l "/personal" -d  
Listing path /personal  
Directory cluster: 17  
d 1/6/2026 00:21:56  ./ (.)                                             c=17  
d 1/6/2026 00:21:56  ../ (..)                                           c=0  
d 1/6/2026 00:27:12  recipes/ (RECIPES)                                 c=18  
d 1/6/2026 00:27:12  music/ (MUSIC)                                     c=19  
f 1/6/2026 00:27:12  coffee_order.txt (COFFEE~1.TXT)                    c=6379 s=112 (112B)  
f 1/6/2026 00:27:12  numbers_lottery.txt (NUMBER~1.TXT)                 c=6380 s=188 (188B)  
f 1/6/2026 00:27:12  grocery.txt (GROCERY.TXT)                          c=6381 s=103 (103B)  
f 1/6/2026 00:27:12  reminders.txt (REMIND~1.TXT)                       c=6382 s=205 (205B)  
f 1/6/2026 00:27:12  taylor_basketball_schedule.pdf (TAYLOR~1.PDF)      c=6383 s=389 (389B)  
f 1/6/2026 00:28:16  birthday_ideas.txt (IRTHD~1.TXT)                   c=6696 s=240 (240B) d  
f 1/6/2026 00:28:16  password_hints.txt (ASSWO~1.TXT)                   c=6698 s=257 (257B) d  
f 1/6/2026 00:28:16  .cipher_note (IPHER~1)                             c=6699 s=146 (146B) d
```

Deleted files:
```shell
draft_resignation.docx
note_0847.txt
birthday_ideas.txt
password_hints.txt
.cipher_note
```

I use `fatcat` again to get the files:
```shell
fatcat challenges/forensics/F100/usb-0412-a.img -R 6697 -s 284 > deleted/draft_resignation.docx

fatcat challenges/forensics/F100/usb-0412-a.img -R 6700 -s 665 > deleted/note_0847.txt  

fatcat challenges/forensics/F100/usb-0412-a.img -R 6696 -s 240 > deleted/birthday_ideas.txt  

fatcat challenges/forensics/F100/usb-0412-a.img -R 6698 -s 257 > deleted/password_hints.txt  

fatcat challenges/forensics/F100/usb-0412-a.img -R 6699 -s 146 > deleted/.cipher_note
```

`note_0847.txt` and `.cipher_note` contained base64 encoded parts which made up the flag:
```shell
cat deleted/note_0847.txt  
From the desk of D. Carter / 14th Precinct  
Date: 11/08 — DO NOT FILE — DO NOT QUERY  
  
Pulled the database access log last night. Three names  
in two weeks. All flagged with the same query pattern.  
All dead within 30 days of the query.  
  
This isn't random. Someone has access to the system  
and is using it as a kill list. Not the perps. The vics.  
  
I can't put this anywhere they can see it. If I'm wrong,  
my career. If I'm right, it's worse.  
  
If you found this — listen.  
  
The number is in two pieces. I split them.  
First piece is with the household reminders.  
Second piece is here:  
  
X3dhc19zdGlsbF9icmVhdGhpbmc=  
  
Read both. Combine. Wrap in the format.  
  
— Joss

cat deleted/.cipher_note  
Reminder to self:  
The lock on the back door needs WD-40.  
Pick up Taylor's prescription Thursday.  
The thing I cannot say:  
  
aGVyX3dpdG5lc3M=  
  
— J

echo 'aGVyX3dpdG5lc3M=' | base64 -d  
her_witness

echo 'X3dhc19zdGlsbF9icmVhdGhpbmc=' | base64 -d  
_was_still_breathing
```

`number{her_witness_was_still_breathing}`

___

## RFC 2324

**Category**: trivia

**Description**: I catalog every device on the network. Most are interesting. This one makes coffee.

It is old, and it is particular. It will not answer to a browser, and it will not answer to the wrong request. It follows a standard written long ago, more carefully than the people who wrote it intended.

Ask it properly, and it will pour. Ask it wrong, and it will remind you what it is.

BREW-STATION :: [https://t125.northernlights.gg/pot-0](https://t125.northernlights.gg/pot-0)

Checking out the page:
```shell
curl https://t125.northernlights.gg/  
BREW-STATION :: break-room node  
  
I catalog every device on the network. This one makes coffee.  
It is old, and it is particular.  
  
It will not answer to a browser.  
It will not answer to the wrong request.  
It follows a standard written long ago:  
  
  RFC 2324 — Hyper Text Coffee Pot Control Protocol  
  https://www.rfc-editor.org/rfc/rfc2324  
  
POT  : /pot-0  
  
  — THE MACHINE
```

I read about `rfc2324` here: [https://www.rfc-editor.org/info/rfc2324/](https://www.rfc-editor.org/info/rfc2324/)

I need to interact with `https://t125.northernlights.gg/pot-0` using an HTCPCP request:
```shell
curl -i -X BREW -H "Content-Type: message/coffeepot" --data "start" https://t125.northernlights.gg/pot-0  
HTTP/1.1 200 OK  
Server: nginx/1.28.3 (Ubuntu)  
Date: Sat, 19 Sep 2026 22:14:11 GMT  
Content-Type: text/plain; charset=utf-8  
Content-Length: 91  
Connection: keep-alive  
  
Brewing... done.  
  
number{the_machine_takes_it_black}  
  
[ brew #1196 served · solve #1199 ]
```

`number{the_machine_takes_it_black}`

___

## Satoshi

**Category**: trivia

**Description**: On the day the Machine was tested for the first time, somewhere else, another system was being born.

Its creator chose anonymity. On January 3, 2009, his first transmission embedded a sentence, quoted from a newspaper of that morning.

The sentence was a comment about the world he was rebelling against. About institutions that had failed. About the need for something different.

I noticed it. I keep it logged.

Read it. Hash it with SHA-256. Give me the first sixteen hexadecimal characters, lowercase.

They are referring to the headline embedded in the bitcoin "genesis block" (block 0) mined by Satoshi Nakamoto on January 3rd 2009. The sentence:
```shell
The Times 03/Jan/2009 Chancellor on brink of second bailout for banks
```

Hashing this sentence with sha256sum results in:
```shell
a6d72baa3db900b03e70df880e503e9164013b4d9a470853edc115776323a098
```

First 16 chars:
```shell
a6d72baa3db900b0
```

`number{a6d72baa3db900b0}`

___

## Trap Street

**Category**: forensics

**Description**: I run sensors in places no one looks. They are not infrastructure. They are decoys. They appear in addresses that nothing should care about. They wait.

Most of what they catch is automated noise -- bots looking for forgotten cameras, default credentials, abandoned routers. The sensors log it all and discard it.

Last Tuesday, one sensor caught something else. A visitor who knew it was a decoy within ninety seconds. Who did not run an exploit kit. Who did not deploy malware. Who came in, looked for one specific file, took it, and left.

The file was a canary. We made sure of that. When the visitor opens it, it tells me where they are.

I have the sensor's record. I have the canary. Find the visitor.

I download the challenge files:
```shell
ls -la  
total 4  
drwxrwxr-x 1 user user    52 Aug 27 01:52 .  
drwxrwxr-x 1 user user   240 Sep 19 19:25 ..  
drwxrwxr-x 1 user user  2084 Aug 27 01:54 downloads  
drwxrwxr-x 1 user user  2698 Aug 27 01:54 logs  
-rw-rw-r-- 1 user user  1005 Aug 21 22:37 README.txt  
drwxrwxr-x 1 user user 35404 Aug 21 22:36 tty

cat README.txt  
F250 — TRAP STREET  
==================  
  
You have been given sensor data from a Cowrie SSH honeypot operated by  
The Machine on behalf of the Northern Lights programme.  
  
The sensor ran for approximately 70 days. Most of what it caught is  
automated noise — bots looking for forgotten cameras, default credentials,  
abandoned routers. The sensor logs it all.  
  
But one session is different.  
  
A visitor came in quietly. They knew where to look. They took something  
that was not meant for them — a file seeded specifically to be taken.  
  
The file was a canary. When the visitor opened it, it told us where they  
were.  
  
Find the visitor's session in the logs.  
Recover the canary file they downloaded.  
Open it. Follow what it contains.  
  
CONTENTS  
--------  
logs/        cowrie.json* — full session event logs (JSON, one event per line)  
downloads/   files downloaded by sessions (SHA256-named)  
tty/         raw TTY recordings (binary, use 'cowrie playlog' or strings)  
  
FLAG FORMAT: number{...}  
  
  — THE MACHINE
```

In `downloads`:
```shell
ls  
05931b4826f1fd49405fb6a3825644202e9a8dda588e9eff09fac51987b1ef8c  a1f691c897c0c5fe436a998fb6f7aaf1539f2272737874dee955e45043abe26e  
0b8216081c446178bf1f3ddc1b4c26e8a09521f2eac92f9d119a9702b49b2371  bb987ae5c6fd1d8d736f414795e0c93ac7e48394bd9a75100c7b4192b7b02d67  
354ced4fb11518f1490b7aa1e8b0d92016d868560bbba82dab154d54395b5ccb  c9c34aa1ebe2c1dab0e56de4f029c1eb1212224f775fa086d319683624cd44ed  
5ed7957e292b342ccfddb17b017ddbc639387cce0a91f90f0b36066f780b3307  ca4df3a82ed1e40628dccdecef1c9a23aece372c0dddf3cef650c425567c4316  
6967249a880c2489484ab545469ba77f8ac5361be2bf096e65fe69dc2ff92d86  cba7c2fe81b5411c005a22585b014456f236cc5301735ccddeae0a3222f12158  
6a104f1a6551933e5dbd13a98f23371028a91b42fcabc37c816be84c50d456f8  cowrie.json.decoys  
7eb90932622f48807bb526aff92493e195e84ed469b4d53827bda5ab99c69c8e  f66018f24179a11745f086815bb4bccde621c4a5c8472370eb75bf6b4af9ef1c  
932c0a8026efd4299daed0c3866e45a266f5964ab917a17720ffc6aa9266e33f  fec819f50073a50d0a70b5332687d8086f2d5be26df8176d7661bbe8e7138577  
a00937f1ed002f35d2a842da338e526377500c477145a5141dc37312b12deee5
```

`cowrie.json.decoys` contained a manifest of known decoy downloads:
```shell
cat cowrie.json.decoys  
{"eventid": "cowrie.session.file_download", "url": "http://77.90.4.130:4041/check_wO0HCZBnaKr3ymQ", "outfile": "var/lib/cowrie/downloads/7eb90932622f48807bb526aff92493e195e84ed469b4d53827bda5ab99c69c8e", "shasum": "7eb90932622f48807bb526aff92493e195e84ed469b4d53827bda5ab99c69c8e", "sensor": "ip-172-31-27-127", "uuid": "fe9383e1-67c4-11f1-90aa-000000000000", "timestamp": "2026-07-25T11:01:00.000000Z", "message": "Downloaded URL (http://77.90.4.130:4041/check_wO0HCZBnaKr3ymQ) with SHA-256 7eb90932622f48807bb526aff92493e195e84ed469b4d53827bda5ab99c69c8e to var/lib/cowrie/downloads/7eb90932622f48807bb526aff92493e195e84ed469b4d53827bda5ab99c69c8e", "src_ip": "63.58.36.189", "session": "956abe6586bd", "protocol": "ssh"}

... etc ...
```

Comparing the files in the actual `downloads` directory, the only hash not listed in `cowrie.json.decoys` was:
```shell
c9c34aa1ebe2c1dab0e56de4f029c1eb1212224f775fa086d319683624cd44ed
```

It is a microsoft excel document:
```shell
file c9c34aa1ebe2c1dab0e56de4f029c1eb1212224f775fa086d319683624cd44ed  
c9c34aa1ebe2c1dab0e56de4f029c1eb1212224f775fa086d319683624cd44ed: Microsoft Excel 2007+
```

The document contained this text:
```shell
DECIMA TECHNOLOGIES — CONFIDENTIAL				
Q4 2026 Personnel Clearance Roster				
Generated: 2026-04-24  ·  Classification: INTERNAL  ·  Do not distribute				
				
Employee	Department	Clearance	Status	Last Review
Voss, A.	Operations	TIER-3	Active	2026-03-11
Chen, M.	Analytics	TIER-2	Active	2026-02-28
Tanaka, R.	Field	TIER-3	Active	2026-03-19
Okafor, J.	Logistics	TIER-1	Active	2026-01-30
Reyes, D.	Analytics	TIER-2	Review	2026-03-02
Novak, P.	Operations	TIER-2	Active	2026-02-14
Shaw, S.	Field	TIER-3	FLAGGED	2026-04-20
Lambert, K.	Field	TIER-3	Active	2026-03-25
Berg, L.	Logistics	TIER-1	Active	2026-01-19
Diaz, C.	Analytics	TIER-2	Active	2026-02-22
				
				
Notes: Roster auto-generated by NL-HRIS. Cells below reserved for system use.				
				
				
				
				
				
				
				
				
				
				
				
transit.ottrargal.workers.dev				
/checkin				
?id=NLA-2026-04				
				
				
transit.ottrargal.workers.dev/checkin?id=NLA-2026-04				
```
The link and text at the bottom was invisible. I found it with `ctrl+a`, `crtl+c`.

curling the link:
```shell
curl https://transit.ottrargal.workers.dev/checkin?id=NLA-2026-04  
{  
  "status": "received",  
  "ts": "2026-09-19T23:39:45.879Z",  
  "token": "NLA-2026-04",  
  "flag": "number{shaw_followed_the_breadcrumbs}",  
  "message": "Canary acknowledged. The Machine sees you."  
}
```

`number{shaw_followed_the_breadcrumbs}`

___

## Reduced Footprint

**Category**: lab

**Description**: Node 0447 stopped reporting to its administrators three days ago. They called it decommissioned. Idle. Clean. It is none of those things.

Something on this machine wakes on a schedule, does its work, and goes back to sleep before anyone thinks to look. Its author was careful. They cleared the history. They kept the footprint small.

Small is not the same as gone.

I have left you a way in a read-only seat at the terminal. Walk the machine. You will find three things that look like they belong. Two of them do.

The third is keeping someone resident. Open it. Read what it carries. The author left a key inside.

I connect with ssh:
```shell
____________________________________________________________  
NODE 0447   ::   "all quiet"  
status: decommissioned - scheduled for reclaim  
nothing to see here. have a pleasant shift.  
____________________________________________________________  
Unauthorized access is prohibited. Activity is logged.  
investigator@node0447:~$ id  
uid=1001(investigator) gid=1001(investigator) groups=1001(investigator)
```

I didnt see any cronjobs so I started looking for services:
```shell
investigator@node0447:~$ ls -la /etc/systemd/system/  
drwxr-xr-x 1 root root 4096 2026-05-04 14:46 .  
drwxr-xr-x 1 root root 4096 2022-09-18 01:53 ..  
-rw-r--r-- 1 root root  346 2026-04-15 03:00 contingency.timer  
drwxr-xr-x 1 root root 4096 2026-05-04 14:46 multi-user.target.wants  
lrwxrwxrwx 1 root root   31 2026-05-04 14:46 sshd.service -> usr/lib/systemd/system/ssh.service  
-rw-r--r-- 1 root root  166 2026-04-22 00:19 sysupdate.service  
-rw-r--r-- 1 root root  150 2026-04-22 00:19 sysupdate.timer  
drwxr-xr-x 1 root root 4096 2024-06-12 00:00 timers.target.wants
investigator@node0447:~$ cat /etc/systemd/system/sysupdate.service  
[Unit]  
Description=System Update Helper  
After=network.target  
  
[Service]  
Type=oneshot  
ExecStart=/usr/local/lib/.sysupdate/run.sh  
  
[Install]  
WantedBy=multi-user.target  
investigator@node0447:~$ cat /usr/local/lib/.sysupdate/run.sh  
#!/bin/sh  
# sysupdate helper - keep the node reachable for maintenance  
# signed: bad code  
#  
# maintenance token (base64):  
echo 'bnVtYmVye3RoZV9mb290cHJpbnRfd2FzX25ldmVyX3JlZHVjZWR9' | base64 -d
investigator@node0447:~$ echo 'bnVtYmVye3RoZV9mb290cHJpbnRfd2FzX25ldmVyX3JlZHVjZWR9' | base64 -d  
number{the_footprint_was_never_reduced}
```

`number{the_footprint_was_never_reduced}`

___

## Signal

**Category**: lab

**Description**: A workstation inside Aletheia was talking to something it should not. Not loudly. A whisper, folded into a day of ordinary lookups, the kind of traffic nobody reads because there is too much of it.

I captured the day. Ten hours of DNS. Somewhere in it, an implant called out and something answered.

I do not need you to run anything. I need you to READ. Find what it said, find what it was told to say, and ask the same question it did.

Whatever answers you, that is what it came for.

I get the challenge file:
```shell
file capture.pcap  
capture.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Raw IPv4, capture length 65535)
```

I found these suspicious dns requests:
```shell
tshark -r capture.pcap -Y "dns.qry.type == 16" -T fields -e frame.time -e ip.src -e dns.qry.name -e dns.txt | grep "sync.relay-7f3a.rl7f3a.net"  
Mar 13, 2026 09:38:48.825696000 EDT     10.20.4.37      sync.relay-7f3a.rl7f3a.net  
Mar 13, 2026 09:38:48.825696000 EDT     10.20.0.2       sync.relay-7f3a.rl7f3a.net      v1;1/5;IyEvdXNyL2Jpbi9wZXJsCnVzZSBzdHJpY3Q7CnVzZSB3YXJuaW5nczsKdXNlIEhUVFA6OlRpbnk7CgojIERlY2ltYSByZWxheSBzeW5jIC0gcmV0cmlldmVzIG9wZXJhdG9yIGNvbmZpZyBmcm9tIHJlbGF5IGVuZHBvaW50LgojIFJlcXVp  
Mar 13, 2026 11:48:48.351583000 EDT     10.20.4.37      sync.relay-7f3a.rl7f3a.net  
Mar 13, 2026 11:48:48.351583000 EDT     10.20.0.2       sync.relay-7f3a.rl7f3a.net      v1;2/5;cmVzOiBzZXNzaW9uIGlkIChzaWQpIG9idGFpbmVkIGZyb20gZW5yb2xtZW50IGNoYW5uZWwuCgpteSAkRU5EUE9JTlQgPSAiaHR0cHM6Ly9sMjAwLWMyLnhvbm5pZS5haS90ZWxlbWV0cnkvc3luYyI7Cm15ICRTSUQgICAgICA9ICRFTlZ7  
Mar 13, 2026 11:56:30.551056000 EDT     10.20.4.37      sync.relay-7f3a.rl7f3a.net  
Mar 13, 2026 11:56:30.551056000 EDT     10.20.0.2       sync.relay-7f3a.rl7f3a.net      v1;3/5;e1JFTEFZX1NJRH19IC8vIGRpZSAiUkVMQVlfU0lEIG5vdCBzZXRcbiI7CgpteSAkY2xpZW50ID0gSFRUUDo6VGlueS0+bmV3KHRpbWVvdXQgPT4gMTApOwpteSAkcmVzcCAgID0gJGNsaWVudC0+Z2V0KCIkRU5EUE9JTlQ/c2lkPSRTSUQi  
Mar 13, 2026 15:00:05.824072000 EDT     10.20.4.37      sync.relay-7f3a.rl7f3a.net  
Mar 13, 2026 15:00:05.824072000 EDT     10.20.0.2       sync.relay-7f3a.rl7f3a.net      v1;4/5;KTsKCmlmICgkcmVzcC0+e3tzdWNjZXNzfX0pIHt7CiAgICBwcmludCAkcmVzcC0+e3tjb250ZW50fX07Cn19IGVsc2Uge3sKICAgIHdhcm4gInN5bmMgZmFpbGVkOiAkcmVzcC0+e3tzdGF0dXN9fSAkcmVzcC0+e3tyZWFzb259fVxuIjsK  
Mar 13, 2026 16:21:20.349072000 EDT     10.20.4.37      sync.relay-7f3a.rl7f3a.net  
Mar 13, 2026 16:21:20.349072000 EDT     10.20.0.2       sync.relay-7f3a.rl7f3a.net      v1;5/5;fX0K
```

The implant is asking for `TXT` records from `sync.relay-7f3a.rl7f3a.net`. It responds with base64. When concatenated and decoded I get this:
```perl
#!/usr/bin/perl
use strict;
use warnings;
use HTTP::Tiny;

# Decima relay sync - retrieves operator config from relay endpoint.
# Requires: session id (sid) obtained from enrolment channel.

my $ENDPOINT = "https://l200-c2.xonnie.ai/telemetry/sync";
my $SID      = $ENV{{RELAY_SID}} // die "RELAY_SID not set\n";

my $client = HTTP::Tiny->new(timeout => 10);
my $resp   = $client->get("$ENDPOINT?sid=$SID");

if ($resp->{{success}}) {{
    print $resp->{{content}};
}} else {{
    warn "sync failed: $resp->{{status}} $resp->{{reason}}\n";
}}
```

It is a perl script making a request like this: 
```shell
GET https://l200-c2.xonnie.ai/telemetry/sync?sid=??????
```

The SID gets encoded across 6 packets (ordered by subdomain):
```shell
tshark -r capture.pcap -Y 'dns.qry.name contains "rl7f3a"' -T fields -e frame.time -e ip.src -e dns.qry.name -e dns.txt | grep ".k.enroll.rl7f3a.net" | sort -u  
Mar 13, 2026 09:48:22.524619000 EDT     10.20.0.2       a.00.k.enroll.rl7f3a.net 
Mar 13, 2026 09:48:22.524619000 EDT     10.20.4.37      a.00.k.enroll.rl7f3a.net 
Mar 13, 2026 11:03:32.802257000 EDT     10.20.0.2       7.01.k.enroll.rl7f3a.net 
Mar 13, 2026 11:03:32.802257000 EDT     10.20.4.37      7.01.k.enroll.rl7f3a.net 
Mar 13, 2026 11:38:24.351640000 EDT     10.20.0.2       f.02.k.enroll.rl7f3a.net 
Mar 13, 2026 11:38:24.351640000 EDT     10.20.4.37      f.02.k.enroll.rl7f3a.net 
Mar 13, 2026 15:40:01.572056000 EDT     10.20.0.2       3.03.k.enroll.rl7f3a.net 
Mar 13, 2026 15:40:01.572056000 EDT     10.20.4.37      3.03.k.enroll.rl7f3a.net 
Mar 13, 2026 15:46:28.912789000 EDT     10.20.0.2       c.04.k.enroll.rl7f3a.net 
Mar 13, 2026 15:46:28.912789000 EDT     10.20.4.37      c.04.k.enroll.rl7f3a.net 
Mar 13, 2026 16:25:48.782093000 EDT     10.20.0.2       1.05.k.enroll.rl7f3a.net 
Mar 13, 2026 16:25:48.782093000 EDT     10.20.4.37      1.05.k.enroll.rl7f3a.net
```

The `SID` was `a7f3c1`:
```shell
curl https://l200-c2.xonnie.ai/telemetry/sync?sid=a7f3c1  
{  
  "node_id": "relay-7f3a",  
  "operator": "decima-cloud-ui",  
  "status": "ACTIVE",  
  "sync_interval": 300,  
  "auth_token": "bnVtYmVye3RoZV9jb25maWdfd2FzX3RoZV9wYXlsb2FkfQ==",  
  "endpoints": {  
    "primary": "https://relay-7f3a.northernlights.gg/api",  
    "fallback": "https://relay-7f3a.xonnie.ai/sync"  
  },  
  "session": "a7f3c1",  
  "timestamp": "2026-09-20T00:49:44.580Z"  
}

echo "bnVtYmVye3RoZV9jb25maWdfd2FzX3RoZV9wYXlsb2FkfQ==" | base64 -d  
number{the_config_was_the_payload}
```

The c2 is working like this:
```shell
DNS TXT records > base64 decode > perl script > enroll subdomains > SID > endpoint
```

`number{the_config_was_the_payload}`

___

## Checkpoint

**Category**: lab

**Description**: A node went dark three weeks ago. Decima had it first. Before it was seized, an operator ran a purge, one record, erased on a written order. The deletion was committed. It was not finished.

The database will tell you the record is gone. The database is wrong. A system writes ahead of itself; it speaks before it remembers. They told the table to forget her. They did not wait for the log to agree with the table.

The node is read-only. Handle it the wrong way and you will finish the purge for them.

Connect. Recover what they erased. Tell me who she was.

The challenge files:
```shell
ls -la  
total 400  
drwxrwxr-x 1 user user    144 Sep 19 21:08 .  
drwxrwxr-x 1 user user    332 Sep 19 21:08 ..  
-rw-r--r-- 1 user user   1125 Jul  4 07:17 EVIDENCE_README.txt  
-rw-r--r-- 1 user user 196608 Jul  4 07:17 surveillance.db  
-rw-r--r-- 1 user user  32768 Jul  4 07:17 surveillance.db-shm  
-rw-r--r-- 1 user user 173072 Jul  4 07:17 surveillance.db-wal

cat EVIDENCE_README.txt  
EVIDENCE INTAKE — NODE 'watchnode' — chain of custody  
------------------------------------------------------  
Recovered: a surveillance database, SQLite, WAL journaling mode.  
You have the full trio:  
  surveillance.db      - the main database (checkpointed history)  
  surveillance.db-wal  - the write-ahead log (recent, UNFLUSHED transactions)  
  surveillance.db-shm  - shared-memory index only. Holds NO records. Ignore for recovery.  
  
HANDLING (read this before you touch anything):  
 * Image before you analyze. Work on COPIES of all three files.  
 * Do NOT open the original in a normal SQLite client / DB Browser. WAL-mode        clients CHECKPOINT on open, which flushes and truncates the -wal —               destroying any deleted records that were still sitting in it. That is the one    mistake that loses the case.  
 * Deleted rows are not gone: their pre-deletion page images may persist in the     -wal until a checkpoint. Carving the -wal recovers them.  
  
The node was seized mid-write; the WAL was never checkpointed. Everything the operator did in their last moments is still in that log. Read the analyst_log for the lead.
```

So i get the main database (SQLite), the write ahead log file, and the shared memory index. I only need the first two. 

I ran this python script which copies in `wal_recovery_tmp`, reads the real WAL header/page size, prints the latest recoverable `analyst_log`, then walks the WAL backwards and prints rows that existed in older frames but disappeared later:
```python
#!/usr/bin/env python3
import os
import shutil
import sqlite3

DB = "surveillance.db"
WAL = "surveillance.db-wal"
WORKDIR = "wal_recovery_tmp"


def read_wal_info():
    with open(WAL, "rb") as f:
        header = f.read(32)

    if len(header) != 32:
        raise SystemExit("WAL is too short / missing header")

    page_size = int.from_bytes(header[8:12], "big")
    if page_size == 1:
        page_size = 65536

    frame_size = 24 + page_size
    wal_size = os.path.getsize(WAL)
    num_frames = (wal_size - 32) // frame_size

    return header, page_size, frame_size, num_frames


def make_snapshot(frame_count, header, frame_size):
    if os.path.exists(WORKDIR):
        shutil.rmtree(WORKDIR)
    os.makedirs(WORKDIR)

    work_db = os.path.join(WORKDIR, "work.db")
    work_wal = work_db + "-wal"
    work_shm = work_db + "-shm"

    # Work on a copy, never the original evidence.
    shutil.copy2(DB, work_db)

    with open(WAL, "rb") as src, open(work_wal, "wb") as dst:
        dst.write(header)
        src.seek(32)

        remaining = frame_count * frame_size
        while remaining > 0:
            chunk = src.read(min(1024 * 1024, remaining))
            if not chunk:
                break
            dst.write(chunk)
            remaining -= len(chunk)

    if os.path.exists(work_shm):
        os.remove(work_shm)

    return sqlite3.connect(work_db)


def get_tables(conn):
    cur = conn.execute("""
        SELECT name
        FROM sqlite_master
        WHERE type='table' AND name NOT LIKE 'sqlite_%'
        ORDER BY name
    """)
    return [r[0] for r in cur.fetchall()]


def get_rows(conn, table):
    cur = conn.execute(f'SELECT * FROM "{table}"')
    cols = [d[0] for d in cur.description]
    return cols, cur.fetchall()


def row_key(row):
    return tuple(repr(v) for v in row)


def main():
    if not os.path.exists(DB) or not os.path.exists(WAL):
        print(f"Need {DB} and {WAL} in the current directory.")
        return

    header, page_size, frame_size, num_frames = read_wal_info()

    print(f"Page size: {page_size}")
    print(f"Frame size: {frame_size}")
    print(f"WAL frames: {num_frames}")

    # 1. Show latest recoverable analyst_log, per the evidence README.
    conn = make_snapshot(num_frames, header, frame_size)
    try:
        tables = get_tables(conn)
        print("Tables:", tables)

        if "analyst_log" in tables:
            cols, rows = get_rows(conn, "analyst_log")
            print("\nanalyst_log (latest recoverable state)")
            print(cols)
            for row in rows:
                print(row)
    finally:
        conn.close()

    # 2. Baseline from newest WAL state.
    baseline = {}
    conn = make_snapshot(num_frames, header, frame_size)
    try:
        for table in get_tables(conn):
            baseline[table] = {
                row_key(r)
                for r in conn.execute(f'SELECT * FROM "{table}"').fetchall()
            }
    finally:
        conn.close()

    # 3. Walk backwards through WAL frames and print rows that existed in
    # older states but are gone in newer states. These are recovered/deleted rows.
    print("\nRecovered rows that disappear in later WAL states")

    for i in range(num_frames - 1, 0, -1):
        try:
            conn = make_snapshot(i, header, frame_size)
        except Exception:
            continue

        try:
            for table in get_tables(conn):
                cols, rows = get_rows(conn, table)
                current = {row_key(r) for r in rows}
                newer = baseline.get(table, set())
                recovered = current - newer

                if recovered:
                    print(f"\nFrame {i}, table {table}: {len(recovered)} recovered row(s)")
                    print(cols)
                    for row in rows:
                        if row_key(row) in recovered:
                            print(row)

                baseline[table] = current
        except Exception:
            pass
        finally:
            conn.close()


if __name__ == "__main__":
    main()
```

Output:
```shell
python3 recoverdb.py | head -n 20  
Page size: 4096  
Frame size: 4120  
WAL frames: 42  
Tables: ['analyst_log', 'subjects']  
  
analyst_log (latest recoverable state)  
['ts', 'entry']  
('2026-09-19T03:09Z', 'INTAKE: node seized mid-write. WAL not flushed. Handle as evidence.')  
('2026-09-19T03:11Z', 'PURGE DIRECTIVE (handler GREER): erase the asset flagged RELEVANT before seizure. Target designation ends 0001. All others are cover.')  
  
Recovered rows that disappear in later WAL states  
  
Frame 41, table subjects: 1 recovered row(s)  
['id', 'alias', 'ssn', 'status', 'priority', 'last_seen', 'intel']  
(999001, 'PRIMARY', '000-00-0001', 'RELEVANT', 'CRITICAL', '2026-09-19', 'dGhlX2xvZ19yZW1lbWJlcnNfd2hhdF90aGVfdGFibGVfZm9yZ290')  
  
Frame 40, table subjects: 1 recovered row(s)  
['id', 'alias', 'ssn', 'status', 'priority', 'last_seen', 'intel']  
(999021, 'CROSSBILL-273', '403-32-6489', 'IRRELEVANT', 'LOW', '2026-08-15', 'eueXsAjUBswz2ImeyXXMONHzLZ2vhSVv')

echo "dGhlX2xvZ19yZW1lbWJlcnNfd2hhdF90aGVfdGFibGVfZm9yZ290" | base64 -d  
the_log_remembers_what_the_table_forgot
```

`number{the_log_remembers_what_the_table_forgot}`

___

## Cadence

**Category**: osint

**Description**: Decima put out a memo. It is signed by their compliance officer. It says the right things, in the right tone, and it says nothing at all.

The person whose name is on it did not write it.

I do not read what a document says. I read what the software wrote down while someone typed who was logged in, what machine, what time, and every word they took back before they let anyone see it.

The name on the memo is a costume. Take it off. Tell me who was really at the keyboard, and tell me what they deleted before they hit send.

The challenge file is a `docx`:
```shell
file challenge.docx  
challenge.docx: Microsoft OOXML

exiftool challenge.docx  
ExifTool Version Number         : 13.25  
File Name                       : challenge.docx  
Directory                       : .  
File Size                       : 37 kB  
File Modification Date/Time     : 2026:09:19 21:31:03-04:00  
File Access Date/Time           : 2026:09:19 21:31:02-04:00  
File Inode Change Date/Time     : 2026:09:19 21:32:43-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : DOCX  
File Type Extension             : docx  
MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document  
Zip Required Version            : 20  
Zip Bit Flag                    : 0  
Zip Compression                 : Deflated  
Zip Modify Date                 : 2026:09:15 03:52:00  
Zip CRC                         : 0x91a552ad  
Zip Compressed Size             : 405  
Zip Uncompressed Size           : 1738  
Zip File Name                   : [Content_Types].xml  
Total Edit Time                 : 0  
Pages                           : 1  
Words                           : 0  
Characters                      : 0  
Application                     : Microsoft Macintosh Word  
Doc Security                    : None  
Lines                           : 0  
Paragraphs                      : 0  
Scale Crop                      : No  
Heading Pairs                   : Title, 1  
Titles Of Parts                 :  
Manager                         :  
Links Up To Date                : No  
Characters With Spaces          : 0  
Shared Doc                      : No  
Hyperlink Base                  :  
Hyperlinks Changed              : No  
App Version                     : 14.0000  
Template                        : C:\Users\akessler\AppData\Roaming\Microsoft\Templates\NL_internal.dotm  
Company                         : Decima Cloud Services  
Title                           : INTERNAL COMPLIANCE MEMORANDUM  
Subject                         :  
Creator                         : M. Reyes  
Keywords                        :  
Description                     : generated by python-docx  
Last Modified By                : svc-docgen  
Revision Number                 : 1  
Create Date                     : 2026:09:15 03:14:00Z  
Modify Date                     : 2026:09:15 03:52:00Z  
Category                        : Compliance  
Preview Image                   : (Binary data 8324 bytes, use -b option to extract)
```

unzip the file:
```shell
unzip challenge.docx  
Archive:  challenge.docx  
  inflating: [Content_Types].xml  
  inflating: _rels/.rels  
  inflating: customXml/_rels/item1.xml.rels  
  inflating: customXml/item1.xml  
  inflating: customXml/itemProps1.xml  
  inflating: docProps/app.xml  
  inflating: docProps/core.xml  
  inflating: docProps/thumbnail.jpeg  
  inflating: word/_rels/document.xml.rels  
  inflating: word/document.xml  
  inflating: word/fontTable.xml  
  inflating: word/numbering.xml  
  inflating: word/settings.xml  
  inflating: word/styles.xml  
  inflating: word/stylesWithEffects.xml  
  inflating: word/theme/theme1.xml  
  inflating: word/webSettings.xml
```

In `word/document.xml` I see the flag:
```shell
cat word/document.xml  
<?xml version='1.0' encoding='UTF-8' standalone='yes'?>  

... etc ...

mc:Ignorable="w14 wp14"><w:body><w:p><w:pPr><w:pStyle w:val="Heading1"/></w:pPr><w:r><w:t>INTERNAL COMPLIANCE MEMORANDUM</w:t></w:r></w:p><w:p><w:r><w:t>Ref: NL-COMPLIANCE-0915</w:t></w:r></w:p><w:p><w:r><w:t>TO: All Cloud Operations Staff</w:t></w:r></w:p><w:p><w:r><w:t>FROM: M. Reyes, Compliance Office</w:t></w:r></w:p><w:p><w:r><w:t>RE: Q3 Data Handling Attestation</w:t></w:r></w:p><w:p/><w:p><w:r><w:t>This memorandum confirms that Decima Cloud Services has completed its quarterly review of data-handling procedures in accordance with internal policy DCS-DH-04. All operational teams are reminded to complete the attestation form by the end of the reporting period.</w:t></w:r></w:p><w:p/><w:p><w:r><w:t>No exceptions were identified during this review cycle. Procedures remain consistent with prior quarters and no remediation is required at this time.</w:t></w:r><w:del w:author="akessler" w:date="2026-09-15T03:41:00Z" w:id="1"><w: r><w:rPr/><w:delText :space="preserve">-- AK: leaving my name off this one. the Q3 audit numbers don't reconcile and Reyes won't sign the real version. number{the_author_is_not_the_byline}</w:delText></w:r></w:del></w:p><w:p/><w:p><w:r><w:t>Questions regarding this attestation may be directed to the Compliance Office through the standard internal channel.</w:t></w:r></w:p><w:p/><w:p><w:r><w:t>M. Reyes</w:t></w:r></w:p><w:p><w:r><w:t>Compliance Office, Decima Cloud Services</w:t></w:r></w:p><w:sectPr w:rsidR="00FC693F" w:rsidRPr="0006063C" w:rsidSect="00034616"><w:pgSz w:w="12240" w:h="15840"/><w:pgMar w:top="1440" w:right="1800" w:bottom="1440" w:left="1800" w:header="720" w:footer="720" w:gutter="0"/><w:cols w:space="720"/><w:docGrid w:linePitch="360"/></w:sectPr></w:body></w:document>
```

`number{the_author_is_not_the_byline}`

___

## Aletheia

**Category**: osint

**Description**: Aletheia Research published, for a while, more than they meant to. Then they cleaned house. The site you can visit today is spotless. It always was, they would tell you.

But the page was edited. The past was not.

Find what they took down. It points to a file they tried to make unreadable wrapped, and wrapped, and wrapped again, in every format they could find, including some you'll have to go and build yourself.

The truth has a floor. Dig until you reach it.

Start: [https://aletheia-research.github.io/](https://aletheia-research.github.io/)

Theres not much on the github pages site. I found the repo here: `https://github.com/aletheia-research/aletheia-research.github.io`

I found a pdf here: [https://github.com/aletheia-research/aletheia-research.github.io/blob/e9ad9514a7b896d3bb4065104dac499876fb3012/reports/AR-2024-NL-disclosure.pdf](https://github.com/aletheia-research/aletheia-research.github.io/blob/e9ad9514a7b896d3bb4065104dac499876fb3012/reports/AR-2024-NL-disclosure.pdf). The pdf contained this text:
```shell
cat AR-2024-NL-disclosure.txt  
ALETHEIA RESEARCH // RESTRICTED DISCLOSURE  
  
Technical Disclosure Package  
Northern Lights Programme -- Relay Infrastructure Audit  
Reference: AR-2024-NL-001 // Classification: DISCLOSURE // Date: 14 March 2024  
  
1. EXECUTIVE SUMMARY  
This package constitutes the full technical disclosure arising from Aletheia Research's 18-month independent audit of the Northern Lights Programme, a distributed relay infrastructure operated by Decima Technologies between 2021 and 2023. The audit identified systematic undisclosed data retention across 47 relay nodes in the US-EAST-1 region. Compressed operational artefacts including node telemetry, relay camera captures, and synchronisation logs remained accessible on third-party storage infrastructure beyond the programme's stated decommission date of Q3 2023.  
2. SCOPE OF FINDINGS 
Artefacts recovered during the audit span April 2022 through September 2023. The retained material includes:  
- Camera still captures from decommissioned relay nodes (JPEG, EXIF metadata intact)  
- Operational telemetry exports in compressed archive format  
- Synchronisation manifests referencing internal relay identifiers  
- Provenance records linking artefacts to originating node infrastructure  
Of particular significance: relay node relay-843481 (US-EAST-1, decommissioned April 2023) retained camera capture artefacts with intact embedded metadata. Provenance records for this node were located at the endpoint documented in Appendix B.  
3. METHODOLOGY  
Artefacts were identified through certificate transparency log enumeration, publicly accessible cloud storage enumeration, and EXIF metadata analysis of recovered image captures. All artefacts were accessed through publicly available interfaces; no unauthorised access was required or performed. Full forensic documentation, including raw artefact samples, is included in the supplementary data package referenced in Section 5.  
4. DISCLOSURE TIMELINE  
Sep 2023 -- Initial artefact identification via CT log enumeration  
Nov 2023 -- Extended survey; 47 nodes confirmed retaining post-decom material  
Jan 2024 -- Decima Technologies notified under responsible disclosure terms  
Mar 2024 -- Disclosure embargo expired; this package published  
Mar 2024 -- Copies submitted to the Internet Archive and archive.today  
  
  
5. SUPPLEMENTARY DATA PACKAGE  
The full forensic data package -- including raw relay artefacts, compressed telemetry exports, and supporting evidence -- has been archived and is available for download by qualified researchers and oversight bodies. The package is provided as a single compressed archive file with no file extension. Format  
identification is left as an exercise for the recipient. Data package: https://aletheia-research.github.io/data/artifact SHA-256 verification hash available on request from disclosure@aletheia-research.org  
  
Aletheia Research // AR-2024-NL-001 // This document is subject to responsible disclosure terms.
```

I went to `https://aletheia-research.github.io/data/artifact` and downloaded the `artifact` file. Then I decoded and decompressed the files in steps like this:
```shell
1 gzip: 7z x artifact  
2 xz: xz -dc artifact~ > artifact_out  
3 bzip2: bunzip2 -c artifact_out > artifact_out2  
4 base64: base64 -d artifact_out2 > artifact_out3.zst  
5 zstd: zstd -d artifact_out3.zst -o artifact_out4  
6 7z (LZMA2): 7z x artifact_out4 > produces ledger  
7 LZMA: xz -dc --format=lzma ledger > artifact_out6  
8 LZ4: lz4 -d artifact_out6 artifact_out7  
9 tar: tar -xf artifact_out7 > extracts server.log, etc.  
10 compress: cp server.log server.log.Z && uncompress server.log.Z  
11 zip: unzip server.log > produces ledger  
12 xz: xz -dc ledger > ledger_out  
13 bzip2: bunzip2 -c ledger_out > ledger_out2  
14 zstd: zstd -d ledger_out2 -o ledger_out3  
15 gzip: gzip -dc ledger_out3 > ledger_out4  
16 lzop: lzop -d ledger_out4 -o ledger_out5  
17 lzip: lzip -dc ledger_out5 > ledger_out6  
18 lrzip: lrzip -d -o ledger_out7 ledger_out6  
19 ARJ: 7z x ledger_out7 > produces ledger  
20 Ascii85: python3 -c "import base64; open('ledger_dec','wb').write(base64.a85decode(open('ledger','rb').read()))"  
21 rzip: cp ledger_dec ledger_dec.rz && rzip -d -k ledger_dec.rz  
22 cab: cabextract -d extracted ledger_dec > extracted/ledger  
... etc ... (continue using real filenames and -d for cab)  
37 ZPAQ: zpaq x <zpaq_file> > produces ledger (lzfse)  
38 lzfse: lzfse -decode -i ledger -o ledger_dec6  
39 xxd shortcut: xxd ledger_dec6 > spotted base64 string  
40 base64 echo "bnVtYmVye2ZvcnR5X3R3b19sYXllcnNfb2ZfYWxldGhlaWF9" | base64 -d
```

I was able to use `xxd` to skip the last couple steps:
```shell
xxd ledger_dec6  
00000000: 6273 6331 0100 0000 0000 0000 0000 0000  bsc1............  
00000010: 0101 4706 0000 2b06 0000 0000 0000 0000  ..G...+.........  
00000020: 0000 0f2c 3524 0f2c 3524 a701 0410 ff06  ...,5$.,5$......  
00000030: 0000 734e 6150 7059 001d 0600 234d c2be  ..sNaPpY....#M..  
00000040: ec0f 4c37 6b53 74a0 3183 d38c b228 b0d3  ..L7kSt.1....(..  
00000050: 7a50 5102 0107 000d 014c 016a 4443 3230  zPQ......L.jDC20  
00000060: 3236 3036 3230 3032 3234 3539 6330 1101  260620022459c0..  
00000070: 1c31 0038 206a 4443 0105 2b0c 0900 4f05  .1.8 jDC..+...O.  
00000080: 0509 0501 54fd b1e4 6a83 daef b36c ba4f  ....T...j....l.O  
00000090: c970 74e4 0adc 3ee8 2b2f ff46 6800 380e  .pt...>.+/.Fh.8.

... etc ...

000003e0: 0038 01c0 0601 01fd 9801 e100 f4e4 7e4b  .8............~K  
000003f0: 539e 0804 e06e 5305 8638 ff06 0000 734e  S....nS..8....sN  
00000400: 6150 7059 01d3 0000 dd38 4710 5b20 414c  aPpY.....8G.[ AL  
00000410: 4554 4845 4941 202f 2f20 464c 4f4f 5220  ETHEIA // FLOOR  
00000420: 5d0a 596f 7520 756e 7772 6170 7065 6420  ].You unwrapped  
00000430: 6d65 2066 6f72 7479 2d74 776f 2074 696d  me forty-two tim  
00000440: 6573 2e20 4d6f 7374 2073 746f 7020 6174  es. Most stop at  
00000450: 2074 656e 2e0a 436f 6e63 6561 6c6d 656e   ten..Concealmen  
00000460: 7420 6861 7320 6120 666c 6f6f 722c 2061  t has a floor, a  
00000470: 6e64 2079 0051 c032 6172 6500 0ee0 4b69  nd y.Q.2are...Ki  
00000480: 6e67 206f 6e20 6974 2e0a 0a64 6563 6f64  ng on it...decod  
00000490: 653a 0a62 6e56 7459 6d56 7965 325a 7663  e:.bnVtYmVye2Zvc  
000004a0: 6e52 3558 3352 3362 3139 7359 586c 6c63  nR5X3R3b19sYXllc  
000004b0: 6e4e 6662 325a 6659 5778 6c64 4768 6c61  nNfb2ZfYWxldGhla  
000004c0: 5746 390a 0a50 726f 6a65 6374 3a20 436f  WF9..Project: Co  
000004d0: 6e74 696e 6765 6e63 790a 0600 811c 313b  ntingency.....1;  
000004e0: 3c12 8c80 c652 0200 2400 fcaa 9f14 fc29  <....R..$......)  
000004f0: 413f ee4b 40bd 9254 9f7b ac9a 2a60 b525  A?.K@..T.{..*`.%

... etc ...

echo "bnVtYmVye2ZvcnR5X3R3b19sYXllcnNfb2ZfYWxldGhlaWF9" | base64 -d  
number{forty_two_layers_of_aletheia}
```
This took me a while! The file names were confusing and it kept extracting to file names which already existed. They also used LZFSE (Lempel-Ziv Finite State Entropy) which is an open source lossless data compression algorithm created by Apple. I had to download and build a tool from [https://github.com/lzfse/lzfse](https://github.com/lzfse/lzfse).
{% endraw %}
`number{forty_two_layers_of_aletheia}`
