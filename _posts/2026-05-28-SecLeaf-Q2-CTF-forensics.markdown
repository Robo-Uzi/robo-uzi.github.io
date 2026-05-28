---
layout: post
title:  "Forensics Challenges"
date:   2026-05-28 13:58:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /SecLeaf-Q2-CTF-2026-forensics/
---
* TOC
{:toc}

## Forgotten_snapshot

**Category**: Forensics

**Description**: We recovered this image from a damaged backup archive. Analysts believe the original owner attempted to conceal sensitive information before deletion. Some image data may have survived recovery. Flag format: `SecLeaf{}`

```shell
file snapshot.jpg  
snapshot.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, comment: "SecLeaf{metadata_never_lies}", progressive, precision 8, 528x500, components 3
```

`SecLeaf{metadata_never_lies}`

___

## Important

**Category**: Forensics

**Description**: A suspicious image file was recovered during investigation. It appears harmless, but appearances can be misleading. Inspect the file carefully, determine its true format, and recover the hidden flag. File Provided: `important.jpg` Flag Format: `SecLeaf{}`

```shell
file Important.jpg  
Important.jpg: Zip archive data, made by v2.0 UNIX, extract using at least v2.0, last modified May 02 2026 21:21:26, uncompressed size 28, method=deflate  

unzip Important.jpg  
Archive:  Important.jpg  
inflating: flag.txt  

cat flag.txt  
SecLeaf{extensions_can_lie}
```

`SecLeaf{extensions_can_lie}`

___

## Force-push-wont-save-you

**Category**: Forensics

**Description**: A developer force-pushed several times before the repository was archived. We suspect sensitive data may still exist somewhere in the project history. Some objects may no longer be referenced. Flag format: `SecLeaf{}`

I get the challenge files:
```shell
unzip challenge.zip  
Archive:  challenge.zip  
   creating: force-push-wont-save-you/  
  inflating: force-push-wont-save-you/app.js  
   creating: force-push-wont-save-you/.git/  
   creating: force-push-wont-save-you/.git/logs/  
  inflating: force-push-wont-save-you/.git/logs/HEAD  
   creating: force-push-wont-save-you/.git/logs/refs/  
  inflating: force-push-wont-save-you/.git/logs/refs/stash  
   creating: force-push-wont-save-you/.git/logs/refs/heads/  
  inflating: force-push-wont-save-you/.git/logs/refs/heads/master  
 extracting: force-push-wont-save-you/.git/HEAD  
   creating: force-push-wont-save-you/.git/refs/  
   creating: force-push-wont-save-you/.git/refs/tags/  
 extracting: force-push-wont-save-you/.git/refs/stash  
   creating: force-push-wont-save-you/.git/refs/heads/  
 extracting: force-push-wont-save-you/.git/refs/heads/master  
   creating: force-push-wont-save-you/.git/hooks/  
  inflating: force-push-wont-save-you/.git/hooks/pre-commit.sample  
  inflating: force-push-wont-save-you/.git/hooks/push-to-checkout.sample  
  inflating: force-push-wont-save-you/.git/hooks/sendemail-validate.sample  
  inflating: force-push-wont-save-you/.git/hooks/update.sample  
  inflating: force-push-wont-save-you/.git/hooks/pre-receive.sample  
  inflating: force-push-wont-save-you/.git/hooks/pre-rebase.sample  
  inflating: force-push-wont-save-you/.git/hooks/commit-msg.sample  
  inflating: force-push-wont-save-you/.git/hooks/applypatch-msg.sample  
  inflating: force-push-wont-save-you/.git/hooks/pre-merge-commit.sample  
  inflating: force-push-wont-save-you/.git/hooks/pre-applypatch.sample  
  inflating: force-push-wont-save-you/.git/hooks/prepare-commit-msg.sample  
  inflating: force-push-wont-save-you/.git/hooks/pre-push.sample  
  inflating: force-push-wont-save-you/.git/hooks/fsmonitor-watchman.sample  
  inflating: force-push-wont-save-you/.git/hooks/post-update.sample  
  inflating: force-push-wont-save-you/.git/config  
  inflating: force-push-wont-save-you/.git/description  
   creating: force-push-wont-save-you/.git/info/  
 extracting: force-push-wont-save-you/.git/info/exclude  
   creating: force-push-wont-save-you/.git/lost-found/  
   creating: force-push-wont-save-you/.git/lost-found/other/  
 extracting: force-push-wont-save-you/.git/lost-found/other/6d7e0a0d704b9b770f663cfea9487d43d7b5ebc3  
   creating: force-push-wont-save-you/.git/lost-found/commit/  
 extracting: force-push-wont-save-you/.git/lost-found/commit/a197682a21ca5e9d7c6c2da8c2e32b31f8a27002  
 extracting: force-push-wont-save-you/.git/COMMIT_EDITMSG  
 extracting: force-push-wont-save-you/.git/ORIG_HEAD  
inflating: force-push-wont-save-you/.git/index  
   creating: force-push-wont-save-you/.git/objects/  
   creating: force-push-wont-save-you/.git/objects/9d/  
 extracting: force-push-wont-save-you/.git/objects/9d/0b25d8761ab307b516e26e123635ff9b4f2e6d  
   creating: force-push-wont-save-you/.git/objects/27/  
 extracting: force-push-wont-save-you/.git/objects/27/325844b4395d4469ffb30b11181544e75eb7a2  
   creating: force-push-wont-save-you/.git/objects/80/  
 extracting: force-push-wont-save-you/.git/objects/80/911c4ac7684b7b1032e72ae7b5502936fdc525  
   creating: force-push-wont-save-you/.git/objects/a1/  
 extracting: force-push-wont-save-you/.git/objects/a1/97682a21ca5e9d7c6c2da8c2e32b31f8a27002  
   creating: force-push-wont-save-you/.git/objects/2c/  
 extracting: force-push-wont-save-you/.git/objects/2c/c502f6893a052f04f41f5ac1c0157aaeab4173  
   creating: force-push-wont-save-you/.git/objects/pack/  
   creating: force-push-wont-save-you/.git/objects/a5/  

... etc
```

```shell
grep -R SecLeaf  
.git/info/exclude:SecLeaf{history_was_the_trap}  
.git/lost-found/other/6d7e0a0d704b9b770f663cfea9487d43d7b5ebc3:SecLeaf{fake_dangling_flag}
```

`SecLeaf{history_was_the_trap}`

___

## needle_in_context

**Category**: Forensics

**Description**: During a failed forensic recovery attempt, several debug logs were partially reconstructed. Investigators believe some entries may still contain fragments of sensitive data. Be warned: not every recovery artifact is trustworthy. Flag format: `SecLeaf{}`

I get the challenge files:
```shell
ls  
archive_150.log  auth_123.log  auth_147.log  auth_170.log  auth_194.log  auth_217.log  auth_240.log  auth_264.log  auth_288.log  auth_3.log   auth_63.log  auth_87.log ... etc
```

Looking at the files with the `auth_log` naming format:
```shell
cat notes1.log  
metadata corruption confirmed  

cat notes2.log  
metadata integrity verified  

cat notes3.log  
XOR patterns detected  

cat notes4.log  
possible ECB encryption  

cat archive_150.log  
possible recovery: 68655f72  

cat cache_211.log  
fragment recovered: 69735f74  

cat debug_183.log  
cache fragment: 5365634c  

cat error_29.log  
recovered chunk: 6561667b  

cat final_77.log  
orphan data: 636f6e74  

cat recovered_8.log  
sync failed: 6578745f  

cat temp_92.log  
last chunk: 65616c5f656e656d797d
```

Decode this string from hex: `5365634c6561667b636f6e746578745f69735f7468655f7265616c5f656e656d797d`

Cyberchef recipe: [cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Hex('None')&input=NTM2NTYzNGM2NTYxNjY3YjYzNmY2ZTc0NjU3ODc0NWY2OTczNWY3NDY4NjU1ZjcyNjU2MTZjNWY2NTZlNjU2ZDc5N2Q&oenc=65001)

`SecLeaf{context_is_the_real_enemy}`

___

## Almost there

**Category**: Forensics

**Description**: The backup archive seems damaged. But maybe not everything is lost. Flag format: `SecLeaf{}`

It fails to unzip:
```shell
unzip backup.zip  
Archive:  backup.zip  
file #1:  bad zipfile offset (local header sig):  0
```

I just viewed the file with `xxd`:
```shell
xxd backup.zip  
00000000: 0000 0304 0a00 0000 0000 fd6b b45c 1192  ...........k.\..  
00000010: d5a4 1c00 0000 1c00 0000 0800 1c00 666c  ..............fl  
00000020: 6167 2e74 7874 5554 0900 0375 6a0d 6a75  ag.txtUT...uj.ju  
00000030: 6a0d 6a75 780b 0001 04e8 0300 0004 e803  j.jux...........  
00000040: 0000 5365 634c 6561 667b 7265 7061 6972  ..SecLeaf{repair  
00000050: 5f74 6865 5f61 7263 6869 7665 7d0a 504b  _the_archive}.PK  
00000060: 0102 1e03 0a00 0000 0000 fd6b b45c 1192  ...........k.\..  
00000070: d5a4 1c00 0000 1c00 0000 0800 1800 0000  ................  
00000080: 0000 0100 0000 b481 0000 0000 666c 6167  ............flag  
00000090: 2e74 7874 5554 0500 0375 6a0d 6a75 780b  .txtUT...uj.jux.  
000000a0: 0001 04e8 0300 0004 e803 0000 504b 0506  ............PK..  
000000b0: 0000 0000 0100 0100 4e00 0000 5e00 0000  ........N...^...  
000000c0: 0000                                     ..
```

`SecLeaf{repair_the_archive}`

___

## one_billion_tries

**Category**: Forensics

**Description**: We intercepted a password-protected archive during a forensic investigation. Investigators believe the password follows a very weak numeric pattern. Can you recover the hidden file? Flag format: `SecLeaf{}`

I get the challenge file:
```shell
file protected.zip
protected.zip: Zip archive data, made by v3.0 UNIX, extract using at least v1.0, last modified May 12 2026 21:35:34, uncompressed size 26, method=store
```

I generate a hash with `zip2john`:
```shell
zip2john protected.zip > ziphash
ver 1.0 efh 5455 efh 7875 protected.zip/flag.txt PKZIP Encr: 2b chk, TS_chk, cmplen=38, decmplen=26, crc=7B499F13 ts=AC71 cs=ac71 type=0

cat ziphash
protected.zip/flag.txt:$pkzip$1*2*2*0*26*1a*7b499f13*0*42*0*26*ac71*96fc57ca5fb0620d84ac8ec69263eea624a690ddba951f318830580bc050552e2849b2d306ce*$/pkzip$:flag.txt:protected.zip::protected.zip
```

I crack the password with a 10 digit mask using `john`:
```shell
john --mask=?d?d?d?d?d?d?d?d?d?d --format=pkzip ziphash  
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
0g 0:00:00:09 1.13% (ETA: 13:47:01) 0g/s 12523Kp/s 12523Kc/s 12523KC/s 2357142001..4243042001  
0g 0:00:02:36 20.50% (ETA: 13:46:23) 0g/s 13140Kp/s 13140Kc/s 13140KC/s 1158977912..8757977912  
9697989299       (protected.zip/flag.txt)  
1g 0:00:05:31 DONE (2026-05-23 13:39) 0.003012g/s 13367Kp/s 13367Kc/s 13367KC/s 2836989299..4992889299  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

The password is `9697989299`:
```shell
unzip protected.zip  
Archive:  protected.zip  
[protected.zip] flag.txt password:  
replace flag.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: A  
 extracting: flag.txt  

cat flag.txt  
SecLeaf{w0rdl1sts_m4tt3r}
```

`SecLeaf{w0rdl1sts_m4tt3r}`
