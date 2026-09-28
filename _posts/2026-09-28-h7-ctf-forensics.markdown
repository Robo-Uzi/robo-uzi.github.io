---
layout: post
title:  "Forensics Challenges"
date:   2026-09-28 16:55:00 -0400
author: uzi
tags: [CTF]
permalink: /h7-ctf-2026-forensics/
---
* TOC
{:toc}

## Gone Not Forgotten

**Category**: Forensics

**Author**: abu

**Description**: A phone lands on the evidence bench for a harassment case, and the suspect is adamant they never sent anything ugly. Nova Messenger agrees with them: nothing there. Deleting a thing and being rid of it were never the same move.

I get the challenge files:
```shell
file messages.db  
messages.db: SQLite 3.x database, last written using SQLite version 3039002, file counter 9, database pages 3, cookie 0x1, schema 4, largest root page 3, UTF-8, version-valid-for 9

cat secure.xml  
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>  
<map>  
	<string name="obfuscation_key">n0v4ch4t</string>  
	<string name="stored_body_encoding">hex(xor(body, obfuscation_key))</string>
	<boolean name="biometric_lock" value="true" />  
</map>
```

Looking at the database with `sqlite3` doesnt show much:
```shell
sqlite3 messages.db  
SQLite version 3.46.1 2024-08-13 09:16:08  
Enter ".help" for usage hints.  
sqlite> .tables  
messages  
sqlite> select * from messages;  
1|general|alice|1710001000000|lunch at 1?  
2|general|bob|1710001100000|sure, the usual spot  
3|general|alice|1710001200000|cool, see you  
5|ops|mallory|1710002050000|burn this after you read it  
6|general|bob|1710002100000|anyone seen the build?  
7|general|carol|1710002200000|ci is green now
```

Using `strings`:
```shell
strings messages.db | tail -n 7  
26073560251303150a564f025a5d521658054357545d064c5f000b%  
'generalalice  
cool, see you*  
5generalbob  
sure, the usual spot#  
#generalalice  
@lunch at 1?
```

I put the hex string into cyberchef and did the recipe `from hex` then `XOR` with key `n0v4ch4t (latin1)`.

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')XOR(%7B'option':'Latin1','string':'n0v4ch4t'%7D,'Standard',false)&input=MjYwNzM1NjAyNTEzMDMxNTBhNTY0ZjAyNWE1ZDUyMTY1ODA1NDM1NzU0NWQwNjRjNWYwMDBiJQ)

`H7CTF{7adf9695fb655c752810}`

___

## Low and Slow

**Category**: Forensics

**Author**: abu

**Description**: Six weeks, not one alert, a clean bill of health on every dashboard that mattered. Then a partner asks why your unreleased designs are making the rounds. Something in here has been talking to the outside on a very patient schedule.

I get the challenge file:
```shell
file capture.pcap  
capture.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

I found something suspicious in the dns requests:
```shell
tshark -r capture.pcap -Y "dns.flags.response == 0" | grep ".sync.cdn-telemetry-lab.net"  
   47   5.600000    127.0.0.1 → 127.0.0.53   DNS 105 Standard query 0x292b A 00ja3ugvcgpm3doobx.sync.cdn-telemetry-lab.net  
   97  11.752305    127.0.0.1 → 127.0.0.53   DNS 105 Standard query 0x6687 A 01mi4dmy3dg43tozru.sync.cdn-telemetry-lab.net  
  189  19.742784    127.0.0.1 → 127.0.0.53   DNS 101 Standard query 0x4117 A 02gi3geoldgb6q.sync.cdn-telemetry-lab.net  
  265  31.308918    127.0.0.1 → 127.0.0.53   DNS 105 Standard query 0x2769 A 00ja3ugvcgpm3doobx.sync.cdn-telemetry-lab.net  
  317  39.223226    127.0.0.1 → 127.0.0.53   DNS 105 Standard query 0xd592 A 01mi4dmy3dg43tozru.sync.cdn-telemetry-lab.net  
  387  47.023109    127.0.0.1 → 127.0.0.53   DNS 101 Standard query 0x2da4 A 02gi3geoldgb6q.sync.cdn-telemetry-lab.net
```

I get this string:
```shell
ja3ugvcgpm3doobxmi4dmy3dg43tozrugi3geoldgb6q
```

`base32` decode on cyberchef: [cyberchef link](https://gchq.github.io/CyberChef/#recipe=From_Base32('a-z2-7%3D',true)&input=amEzdWd2Y2dwbTNkb29ieG1pNGRteTNkZzQzdG96cnVnaTNnZW9sZGdiNnE&oeol=FF)

`H7CTF{6787b86cc777f426b9c0}`

___

## Ghost in the Timeline

**Category**: Forensics

**Author**: abu

**Description**: An engineer stands accused of walking off with the product roadmap, and the alibi is tidy: the file predates their access, nothing was staged, nothing deleted, and every timestamp agrees. Timestamps are only as honest as whoever set them.

I get the challenge file:
```shell
file case_disk.img  
case_disk.img: Linux rev 1.0 ext4 filesystem data, UUID=1703f65b-7ae7-49fc-a4dc-75da999317b3, volume name "CASE-2026-0917" (extents) (64bit) (large files) (huge files)
```

Checking the filesystem:
```shell
fls -r -p case_disk.img  
d/d 13: home  
d/d 14: home/jdoe  
d/d 15: home/jdoe/designs  
r/r 20: home/jdoe/designs/notes.txt  
r/r 21: home/jdoe/designs/roadmap.pdf  
d/d 16: home/jdoe/.cache  
d/d 17: home/jdoe/.cache/staging  
r/r * 23:       home/jdoe/.cache/staging/q3_designs.zip  
d/d 11: lost+found  
d/d 18: var  
d/d 19: var/log  
r/r 22: var/log/auth.log  
V/V 5113:       $OrphanFiles
```

Download the zip from the deleted hidden staging directory:
```shell
icat case_disk.img 23 > q3_designs.zip  

unzip q3_designs.zip  
Archive:  q3_designs.zip  
  inflating: secret.txt
  
cat secret.txt | grep "H7CTF{"   
recovery token: H7CTF{e211b7db9d4f5c9f0812}
```

`H7CTF{e211b7db9d4f5c9f0812}`
