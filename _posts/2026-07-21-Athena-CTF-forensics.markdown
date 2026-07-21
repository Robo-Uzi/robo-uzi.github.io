---
layout: post
title:  "Forensics Challenges"
date:   2026-07-21 17:21:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Athena-CTF-2026-forensics/
---
* TOC
{:toc}

## Mailroom Echo

Category: forensics

Description: Internal Affairs flagged a helpdesk analyst for quietly routing data out of the building inside dull-looking quarterly summary notes. Before the account could be locked, the support mailbox was quarantined and a single message from the reply thread was exported for review. The body reads like ordinary desk chatter and steers you away from anything useful -- but this is a mail message, and the interesting part rarely rides in the body

I get the challenge file `mailroom_echo.eml`:
```shell
cat mailroom_echo.eml  
From: helpdesk@example.local  
To: analyst@example.local  
Subject: Re: quarantined mailbox export  
Date: Tue, 20 May 2026 09:14:12 +0000  
Message-ID: <20260520091412.4f1a@example.local>  
In-Reply-To: <20260519163355.8c02@example.local>  
References: <20260519154120.1b77@example.local>  
        <20260519163355.8c02@example.local>  
MIME-Version: 1.0  
Content-Type: multipart/mixed; boundary="boundary-9f3c"  
  
--boundary-9f3c  
Content-Type: text/plain; charset="utf-8"  
Content-Transfer-Encoding: 7bit  
  
The original thread was pulled from the support archive.  
Nothing useful is visible in the body, so check the attached note.  
  
--boundary-9f3c  
Content-Type: text/plain; charset="utf-8"; name="quarterly_summary.txt"  
Content-Disposition: attachment; filename="quarterly_summary.txt"  
Content-Transfer-Encoding: base64  
  
UXVhcnRlcmx5IHJlY29uY2lsaWF0aW9uIGNvbXBsZXRlLgpCYWNrdXAgbWFya2VyOiBhdGhlbmF7bWltZV90aHJlYWRzX3JldmVhbF90aGVfdHJ1dGh9CkRvIG5vdCBmb3J3YXJkLgo=  
--boundary-9f3c--  
```

simply decode:
```shell
echo 'UXVhcnRlcmx5IHJlY29uY2lsaWF0aW9uIGNvbXBsZXRlLgpCYWNrdXAgbWFya2VyOiBhdGhlbmF7bWltZV90aHJlYWRzX3JldmVhbF90aGVfdHJ1dGh9CkRvIG5vdCBmb3J3YXJkLgo=' | base64 -d  
Quarterly reconciliation complete.  
Backup marker: athena{mime_threads_reveal_the_truth}  
Do not forward.
```

`athena{mime_threads_reveal_the_truth}`

___

## PCAP Secret

Category: forensics

Description: Incident response pulled a tiny packet capture from a suspect jump host—`forensic_pcap_secret.pcap.gg`. Most of the traffic looks like routine DNS and HTTP noise, but analysts swear one of these connections is exfiltrating the flag to an external host. Download the capture, find the suspicious connection, and recover the hidden flag.

I put the pcap into wireshark and found this:
```http
POST /api/sync HTTP/1.1
Host: 198.51.100.24
User-Agent: curl/7.81.0
X-Sync-Token: YXRoZW5he3BjNHBfaDFkMzVfMW5fdzFyM30=
Content-Type: application/json
Content-Length: 31

{"host":"jump01","status":"ok"}
```

```shell
echo 'YXRoZW5he3BjNHBfaDFkMzVfMW5fdzFyM30=' | base64 -d  
athena{pc4p_h1d35_1n_w1r3}
```

`athena{pc4p_h1d35_1n_w1r3}`

___

## RAM Drift

Category: forensics

Description: A crash dump from an analyst workstation was captured before the system was wiped. Plain strings only show fragments, and the interesting buffer never appears in one piece. Use the memory snapshot and the process map to reconstruct the resident data, then undo the page-level transform to recover the flag.

I get the challenge files:
```shell
cat process_map.txt  
Process map from the crash collector:  
  
pid  name               base      size    xor tag  
111  systemd            0x7f1000  0x1200  tag=STEDY  
482  explorer           0x7f4000  0x2400  tag=CLIP9  
901  photo_recover      0x8a0000  0x3c00  tag=DR1FT  
1044 crash_handler      0x900000  0x1800  tag=PGOFF  
  
Each process scrambled its resident buffer with its own tag (a repeating XOR key).  
The buffer of interest was mapped across three pages before the dump was written;  
identify which process owns those pages, then apply its tag across the whole buffer.  
```

```shell
cat ram_dump.txt  
Memory capture excerpt (page-aligned)  
=====================================  
  
PAGE 0x8a0000  
00000000  25 26 59 23 3a 25 29 43 27 39 1b 22              |%&Y#:%)C'9."|  
00000010  1f 19 1c 0a de ad be ef 08 0f 1c 1a 0a 3d 0d 0d  |.............=..|  
  
PAGE 0x8a1000  
00000000  50 21 31 37 0d 59 2f 30 21 0d 57 34              |P!17.Y/0!.W4|  
00000010  11 0c 17 05 fa ce b0 0c 11 0c 17 05 07 13 0a 0b  |................|  
  
PAGE 0x8a2000  
00000000  35 23 3f 54 28 20 37 2f                          |5#?T( 7/|  
00000010  0c 1a 05 07 0c 0e 07 0f 06 1d 10 12 0a 0b 0d 1f  |................|  
  
captured note: the buffer was XORed before the pages were dumped.
```

Solution:
```python
python3  
Python 3.13.5 (main, Jun 13 2026, 14:18:01) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> key = b"DR1FT"  
...  
... frag1 = bytes.fromhex("25 26 59 23 3a 25 29 43 27 39 1b 22")  
... frag2 = bytes.fromhex("50 21 31 37 0d 59 2f 30 21 0d 57 34")  
... frag3 = bytes.fromhex("35 23 3f 54 28 20 37 2f")  
...  
... plain = b''.join(  
...   bytes(b ^ key[i % len(key)] for i,b in enumerate(buf, start=offset))  
...   for offset,buf in [(0,frag1),(12,frag2),(24,frag3)]  
... )  
...  
>>> print(plain)  
b'athena{ram_pages_hide_fragments}'  
>>> exit
```

`athena{ram_pages_hide_fragments}`

___

## Cache Footprint

Category: forensics

Description: A browser cache was carved from a kiosk image after an incident. The exported SQLite database contains browsing history, downloads, cookies, form data, and session storage. The operator cleaned up after themselves, so the download that matters is no longer in the downloads table — but a delete is not an erase. Recover the missing record, work out how its payload was exported, and decode it to recover the flag. The data URIs still sitting in the database are not the answer.

I get the challenge file: 
```shell
file browser_history.sqlite  
browser_history.sqlite: SQLite 3.x database, last written using SQLite version 3050004, file counter 3, database pages 6, cookie 0x5, schema 4, UTF-8, version-valid-for 3
```

I find what looks like base64:
```shell
strings browser_history.sqlite | grep '=='  
pending_uploaddata:text/plain;base64,c2Vzc2lvbiBoZWFydGJlYXQgb2s7IHJldHJ5IGluIDMwcw==clipboard  
2026-05-19T08:05:26Zdata:application/octet-stream;base64,Ch0HFgVMS0dAVQIdCiwASEBAbk0DDDAQB1hVSQ==/tmp/notes.txt'application/octet-streamcompletee
```

It was deleted from the downloads table:
```shell
sqlite> select * from downloads;  
1|2026-05-19T08:00:05Z|https://updates.local/changelog.txt|/tmp/changelog.txt|2048|text/plain|complete  
2|2026-05-19T08:01:22Z|https://cdn.local/assets/logo.png|/tmp/logo.png|15360|image/png|complete  
3|2026-05-19T08:02:44Z|data:text/plain;base64,a2lvc2sgZGlzcGxheSBjYWxpYnJhdGlvbiBwcm9maWxlIHYz|/tmp/export_1.bin|32|text/plain|complete  
4|2026-05-19T08:03:18Z|https://updates.local/banner.jpg|/tmp/banner.jpg|48128|image/jpeg|complete  
5|2026-05-19T08:04:01Z|https://updates.local/stats.json|/tmp/stats.json|512|application/json|complete  
7|2026-05-19T08:05:58Z|data:application/octet-stream;base64,RVJST1I6IGFjY2VzcyBkZW5pZWQ=|/tmp/auth_token.bin|24|application/octet-stream|complete  
8|2026-05-19T08:06:30Z|https://cdn.local/assets/icon.svg|/tmp/icon.svg|8192|image/svg+xml|complete
```

The `session_storage` table tells me how they were storaged:
```shell
sqlite> select * from session_storage;  
1|last_page|https://intranet.local/downloads|browser  
2|export_format|base64|developer_tools  
3|export_cipher|xor(device_id)|developer_tools  
4|debug_flag|false|config
```

`cookies` table:
```shell
sqlite> select * from cookies;  
1|intranet.local|session_id|a1b2c3d4e5f6|2026-05-20T08:00:00Z  
2|intranet.local|csrf_token|x9y8z7w6v5|2026-05-20T08:00:00Z  
3|updates.local|device_id|kiosk-0419|2026-06-19T08:00:00Z
```

Base64 decode the blob from the missing download and XOR the result with the repeated key `kiosk-0419`:
```python
python3  
Python 3.13.5 (main, Jun 13 2026, 14:18:01) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import base64  
...  
... enc = base64.b64decode("Ch0HFgVMS0dAVQIdCiwASEBAbk0DDDAQB1hVSQ==")  
... key = b"kiosk-0419"  
...  
... flag = bytes([enc[i] ^ key[i % len(key)] for i in range(len(enc))])  
... print(flag.decode())  
...  
athena{sqlite_kept_the_clue}
```

`athena{sqlite_kept_the_clue}`
