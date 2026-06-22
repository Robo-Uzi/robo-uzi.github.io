---
layout: post
title:  "Forensics Challenges"
date:   2026-06-22 12:55:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RIFFHACK-CTF-2026-forensics/
---
* TOC
{:toc}
{% raw %}
## Moon Prism Packet Secrets

**Category**: Forensics

**Difficulty**: easy

**Description**: A late-night idol rehearsal leak includes one selfie and a whisper of intercepted packets. The photo looks harmless, but the gradient refuses to keep the guardians' secret quiet.

I get the challenge files:
```shell
file 1781863998_sailor_spectrum_evidence.zip  
1781863998_sailor_spectrum_evidence.zip: Zip archive data, made by v2.0 UNIX, extract using at least v2.0, last modified Jun 19 2026 11:08:32, uncompressed size 5743, method=deflate  

unzip 1781863998_sailor_spectrum_evidence.zip  
Archive:  1781863998_sailor_spectrum_evidence.zip  
inflating: luna_selfie.png  
inflating: moon_chat.pcap
```

The `.png` probably has the flag. I run `zsteg`:
```shell
zsteg luna_selfie.png  
[?] 338 bytes of extra data after image end (IEND), offset = 0x151d  
extradata:0         .. file: Zip archive, with extra data prepended  
00000000: 0a 4d 4f 4f 4e 53 48 49  4e 45 0a 50 4b 03 04 14  |.MOONSHINE.PK...|  
00000010: 00 00 00 08 00 10 59 d3  5c 97 6f 86 56 c7 00 00  |......Y.\.o.V...|  
00000020: 00 fb 00 00 00 0f 00 00  00 64 69 61 72 79 5f 65  |.........diary_e|  
00000030: 6e 74 72 79 2e 74 78 74  25 8d 41 4a 03 41 10 00  |ntry.txt%.AJ.A..|  
00000040: ef f3 8a 7e 40 08 2b c4  4b 6e 82 8a 01 4d 0e c1  |...~@.+.Kn...M..|  
00000050: 73 e8 ec 74 66 db ed ed  19 66 7a d5 21 04 7c 84  |s..tf....fz.!.|.|  
00000060: 2f f4 25 8e f1 5a 55 50  f7 84 19 9e 67 c5 85 73  |/.%..ZUP....g..s|  
00000070: 1b 28 c2 29 91 07 1b 08  32 0d cd 15 14 f0 8c b9  |.(.)....2.......|  
00000080: c2 91 06 d6 7f f5 b2 db  6d f7 4f 9b ed 03 4c 98  |........m.O...L.|  
00000090: 47 ca c0 7a e5 85 e4 c4  b4 70 6f 73 31 10 1e e9  |G..z.....pos1...|  
000000a0: 4a d9 47 69 a1 62 68 25  96 91 fc 12 ee b4 46 6d  |J.Gi.bh%......Fm|  
000000b0: 4a 4b a2 de 58 03 24 fe  a4 b6 42 43 f8 60 1b e2  |JK..X.$...BC.`..|  
000000c0: 6c ae c7 fc fe a7 2c 46  29 50 1a 13 0f 51 a5 b6  |l.....,F)P...Q..|  
000000d0: 11 01 42 ca 64 56 21 64  f4 4c 6a 4b e7 1e 05 c3  |..B.dV!d.LjK....|  
000000e0: 1a 8e 6c bd 9d ce e7 a9  eb f4 90 f2 cd ed 74 48  |..l...........tH|  
000000f0: ab 2a dd ca 5f 2e ce fd  7c 7d c3 6b c1 c0 bf 50  |.*.._...|}.k...P|  
imagedata           .. text: "n^*XK!d{"  
chunk:0:IHDR        .. file: Adobe Photoshop Color swatch, version 0, 640 colors; 1st RGB space (0), w 0x168, x 0x802, y 0, z 0; 2nd RGB space (0), w 0, x 0, y 0, z 0  
b2,r,lsb,xy         .. text: ["U" repeated 96 times]
```

There is a zip file. Run `binwalk`:
```shell
binwalk -e luna_selfie.png  
  
DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
41            0x29            Zlib compressed data, default compression  
5416          0x1528          Zip archive data, at least v2.0 to extract, compressed size: 199, uncompressed size: 251, name: diary_entry.txt  
  
WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

The zip file contained `diary_entry.txt`:
```shell
cat diary_entry.txt  
Dear Luna,  
  
I slipped the rehearsal diary behind the MOONSHINE marker in the selfie,  
just like the idol manager asked. Anyone inspecting pixel data without  
carving tools should only see a pretty gradient.  
  
Flag: bitctf{{m00n_pr15m_p4yl04d}}
```

`bitctf{{m00n_pr15m_p4yl04d}}`

___

## Black Mirror Memory Fragments

**Category**: Forensics

**Difficulty**: medium

**Description**: A corrupted mobile backup is all that remains of a wiped conversation. Sort the real evidence from the noise, reconstruct what actually happened, and recover the hidden key.

I get the challenge files:
```shell
file 1781949803_black-mirror-memory-fragments-backup.tar  
1781949803_black-mirror-memory-fragments-backup.tar: POSIX tar archive

ls -la  
total 0  
drwxrwxr-x 1 user user   30 Jun 20 15:36 .  
drwxrwxr-x 1 user user 1076 Jun 20 15:36 ..  
drwxr-xr-x 1 user user   48 Jun 20 06:00 app_export  
drwxr-xr-x 1 user user   38 Jun 20 05:39 decoy

ls decoy/  
quarantine_note.txt  

cat decoy/quarantine_note.txt  
Recovered quarantine banner claims to be official support. It reassembles to a different file type than the signed camera media in the primary inbox.
```

In `app_export/`:
```shell
ls -la  
total 28  
drwxr-xr-x 1 user user    48 Jun 20 06:00 .  
drwxrwxr-x 1 user user    30 Jun 20 15:36 ..  
drwxr-xr-x 1 user user   234 Jun 20 05:47 cache  
-rw-r--r-- 1 user user 28672 Jun 20 06:00 messages.db  
drwxr-xr-x 1 user user    68 Jun 20 05:39 metadata
```

The `metadata` directory contains some dummy data.

In `messages.db`:
```shell
sqlite3 messages.db  
SQLite version 3.46.1 2024-08-13 09:16:08  
Enter ".help" for usage hints.  
sqlite> .tables  
contacts            messages            recovered_messages  threads
sqlite> SELECT * FROM recovered_messages;  
201|211|t_river|2|2024-11-02T23:47:02Z|river.k|I kept the porch still and the draft off-grid. The thread copy came back fragmented.||freelist carve  
202|212|t_river|4|2024-11-02T23:49:20Z|river.k|Trail cam still is attached first so you know I mean the right cabin.|att_river_photo|wal merge  
203|213|t_river|5|2024-11-02T23:49:33Z|river.k|Route snapshot is attached too, but it only proves the approach road and checkpoint timing.|att_river_route|wal merge  
204|214|t_river|6|2024-11-02T23:49:58Z|river.k|Use the draft plus the porch still comment. Start the base64 key with Yml0Y3Rme3tzbTF0aDNy||deleted row rebuild  
205|215|t_river|8|2024-11-02T23:50:57Z|river.k|Then continue MzNuX3RocjM0ZF9y before you check the image metadata.||freelist carve  
206|216|t_river|9|2024-11-02T23:51:40Z|river.k|If they wiped the live thread, the off-grid copy is all you have got. The draft attachment is real; the porch still metadata holds the rest.|att_river_final|wal merge  
207|217|t_mara|4|2024-11-02T23:51:02Z|mara.vey|Do not overthink the receipt, it is just timestamp proof for the claim and the parking appeal.||wal merge  
208|218|t_mara|6|2024-11-02T23:52:41Z|jaden.cole|Perfect. Neither image says anything about the cabin, so legal can have them.||freelist carve  
209|105|t_river|3|2024-11-02T23:48:40Z|jaden.cole|good. the lawyer says anything synced to the cloud is already compromised.||page slack carve  
210|108|t_river|7|2024-11-02T23:50:31Z|jaden.cole|sending you the locker now. tell me the moment you have it.||overflow page rebuild
```

This gives me the first part of the flag:
```shell
echo 'Yml0Y3Rme3tzbTF0aDNyMzNuX3RocjM0ZF9y' | base64 -d  
bitctf{{sm1th3r33n_thr34d_r
```

The second part of the flag is in the `cache` directory:
```shell
cd cache

ls  
ch_2c.dat  ch_5a.dat  ch_9f.dat  ch_e1.dat  ch_e2.dat  ch_m1.dat  ch_m2.dat  ch_r1.dat  ch_r2.dat  ch_s1.dat  ch_s2.dat  ch_x7.dat  ch_y4.dat  messages.db  

xxd ch_y4.dat  
00000000: 6561 722d 7769 6465 3b46 7261 6d65 3d70  ear-wide;Frame=p  
00000010: 6f72 6368 2d73 7469 6c6c 3b55 7365 7243  orch-still;UserC  
00000020: 6f6d 6d65 6e74 3d4d 3246 7a63 3256 7459  omment=M2Fzc2VtY  
00000030: 6d78 6c5a 4638 3059 334a 7663 334e 665a  mxlZF80Y3Jvc3NfZ  
00000040: 6e4a 685a 3231 6c62 6e52 7a66 5830 3d3b  nJhZ21lbnRzfX0=;  
00000050: ffd9                                     ..  

echo 'M2Fzc2VtYmxlZF80Y3Jvc3NfZnJhZ21lbnRzfX0=' | base64 -d  
3assembled_4cross_fragments}}
```

`bitctf{{sm1th3r33n_thr34d_r3assembled_4cross_fragments}}`
{% endraw %}