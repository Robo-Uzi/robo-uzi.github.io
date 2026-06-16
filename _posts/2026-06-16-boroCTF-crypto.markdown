---
layout: post
title:  "Crypto Challenges"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /boroCTF-2026-crypto/
---
* TOC
{:toc}

## A basic start

**Author**: Franklin

**Category**: Crypto

**Description**: We, the Boro Cyber Division have been spying on the chats of a group of local hackers. We used to be able to decrypt their chats from base64 but they seemed to have changed their encoding. Can you find out what they’re talking about now?

I get the challenge file:
```shell
cat chal.txt  
Before new encoding:  
VXNlcjE6IEhleSwgSSB0aGluayB0aGUgQm9ybyB0ZWFtIGlzIG9udG8gdXMhClVzZXIyOiBObyBXYXkhIFRoZXkncmUgZ29pbmcgdG8gc2VuZCB0aGUgQ1RGIHBhcnRpY2lwYW50cyBhZnRlciB1cyEKVXNlcjM6IEl0J2xsIGJlIG9rYXksIHdlJ2xsIG1vdmUgdG8gYW5vdGhlciBiYXNlIGVuY29kaW5nIQ==  
After New encoding:  
j2_1+iZB_6AveF[p/Uxg,WT[#F]4pE&m:%gZ6=0{8!Z%F.0Dj2_1rjZB!8O;R@RnqM_1:WGk$yV%J_;mz2gZY^rN$F90axFl]UgZk>;N%y60;mYiSP8=SH)S<Ry*yCrbVu_15Wy{L%;EU,fr&A2N|ir}XxI(pBZ%w<d9P
```

Original encoding:
```shell
echo 'VXNlcjE6IEhleSwgSSB0aGluayB0aGUgQm9ybyB0ZWFtIGlzIG9udG8gdXMhClVzZXIyOiBObyBXYXkhIFRoZXkncmUgZ29pbmcgdG8gc2VuZCB0aGUgQ1RGIHBhcnRpY2lwYW50cyBhZnRlciB1cyEKVXNlcjM6IEl0J2xsIGJlIG9rYXksIHdlJ2xsIG1vdmUgdG8gYW5vdGhlciBiYXNlIGVuY29kaW5nIQ==' | base64 -d  
User1: Hey, I think the Boro team is onto us!  
User2: No Way! They're going to send the CTF participants after us!  
User3: It'll be okay, we'll move to another base encoding!
```

The new encoding is `base 91`. I decoded it on [https://www.dcode.fr/base-91-encoding](https://www.dcode.fr/base-91-encoding). Decoded message:
```shell
User1: Okay we're on the new encoding.  
User2: You wonder if anyones ever reading your messages?  
User1: Nope. boroCTF{B@5ics_0f_B@si6s}
```

`boroCTF{B@5ics_0f_B@si6s}`

___

## Et Tu, Brute

**Author**: ForeverFlames

**Category**: Crypto

**Description**: Stabbed by his own men. Each stab wound marked how betrayed he was. Can you reverse the damage? `erurFWI{@iu13qgq0pru3}`

I put the ciphertext into [https://cryptii.com/pipes/caesar-cipher/](https://cryptii.com/pipes/caesar-cipher/). Shifting by 3 reveals the flag.

`boroCTF{@fr13ndn0mor3}`

___

## Not the Flag

**Author**: ForeverFlames

**Category**: Crypto

**Description**: Challenge: So is this not the flag? If its not not, then what else? `9d 90 8d 90 bc ab b9 84 8b 97 ce db a0 96 8c a0 91 cf 8b a0 91 90 8b a0 8b 97 cc a0 99 93 bf 98 82`

The hex string is the bitwise NOT of the flag. Reverse it:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> hex_bytes = [0x9d, 0x90, 0x8d, 0x90, 0xbc, 0xab, 0xb9, 0x84, 0x8b, 0x97, 0xce, 0xdb, 0xa0, 0x96, 0x8c, 0xa0, 0x91, 0xcf, 0x8b, 0xa0, 0x91, 0x90, 0x8b, 0xa0, 0x8b, 0x97, 0\xcc, 0xa0, 0x99, 0x93, 0xbf, 0x98, 0x82]  
... flag = ''.join(chr(b ^ 0xFF) for b in hex_bytes)  
... print(flag)  
...  
boroCTF{th1$_is_n0t_not_th3_fl@g}
```

`boroCTF{th1$_is_n0t_not_th3_fl@g}`

___

## So Many Layers

**Author**: N/A

**Category**: Crypto

**Description**: Makes me cry.

```shell
00110101 00111001 00110110 01000100 00110011 00111001 00110111 00111001 00110110 00110010 00110011 00110000 00110100 01000101 00110101 00110101 00110101 00110010 00110110 01000101 00110111 00110100 00110100 01000100 00110100 00111001 00110101 00110111 00110111 00110011 00110111 01000001 00110101 00111000 00110011 00110010 00110100 00110110 00110100 01000110 00110101 00111000 00110111 01000001 00110100 00110010 00110111 00110101 00110100 01000100 00110101 00110111 00110011 00111001 00110111 00110101 00110101 00111000 00110110 01000101 00110011 00110000 00110011 01000100
```

I put the binary into cyberchef and do the recipe `from binary` > `from hex` > `from base64`.

[cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=From_Binary('Space',8)From_Hex('None')From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=MDAxMTAxMDEgMDAxMTEwMDEgMDAxMTAxMTAgMDEwMDAxMDAgMDAxMTAwMTEgMDAxMTEwMDEgMDAxMTAxMTEgMDAxMTEwMDEgMDAxMTAxMTAgMDAxMTAwMTAgMDAxMTAwMTEgMDAxMTAwMDAgMDAxMTAxMDAgMDEwMDAxMDEgMDAxMTAxMDEgMDAxMTAxMDEgMDAxMTAxMDEgMDAxMTAwMTAgMDAxMTAxMTAgMDEwMDAxMDEgMDAxMTAxMTEgMDAxMTAxMDAgMDAxMTAxMDAgMDEwMDAxMDAgMDAxMTAxMDAgMDAxMTEwMDEgMDAxMTAxMDEgMDAxMTAxMTEgMDAxMTAxMTEgMDAxMTAwMTEgMDAxMTAxMTEgMDEwMDAwMDEgMDAxMTAxMDEgMDAxMTEwMDAgMDAxMTAwMTEgMDAxMTAwMTAgMDAxMTAxMDAgMDAxMTAxMTAgMDAxMTAxMDAgMDEwMDAxMTAgMDAxMTAxMDEgMDAxMTEwMDAgMDAxMTAxMTEgMDEwMDAwMDEgMDAxMTAxMDAgMDAxMTAwMTAgMDAxMTAxMTEgMDAxMTAxMDEgMDAxMTAxMDAgMDEwMDAxMDAgMDAxMTAxMDEgMDAxMTAxMTEgMDAxMTAwMTEgMDAxMTEwMDEgMDAxMTAxMTEgMDAxMTAxMDEgMDAxMTAxMDEgMDAxMTEwMDAgMDAxMTAxMTAgMDEwMDAxMDEgMDAxMTAwMTEgMDAxMTAwMDAgMDAxMTAwMTEgMDEwMDAxMDA)

`boroCTF{L!k3_aN_0n1on^}`

___

## Flipper's Dilemma

**Author**: Franklin

**Category**: Crypto

**Description**: Why do we flip a coin when we have to make a hard choice? Maybe the flipping here isn't so random. I flipped it once yet still 0x15 times.
```shell
wzgzVASnS4|eE${J%`>h
```

XOR with `0x15`: [cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=XOR(%7B'option':'Hex','string':'0x15'%7D,'Standard',false)&input=d3pnelZBU25TNHxlRSR7SiVgPmg)

`boroCTF{F!ipP1n_0u+}`

___

## Flight

**Author**: Franklin

**Category**: Crypto

**Description**: dark, darker, yet darker.

♌︎□︎❒︎□︎👍︎❄︎☞︎❀︎⬥︎✋︎■︎♑︎📂︎■︎🕯︎♉︎✏︎⧫︎❝︎

I put the wingdings into `https://www.dcode.fr/wingdings-font` and decode to get the flag.

`b︎o︎r︎o︎C︎T︎F︎{︎w︎I︎n︎g︎1︎n︎'︎_︎!︎t︎}︎`

___

## Johnny Boy

**Author**: Franklin

**Category**: Crypto

**Description**: One of our sysadmins recently quit after we had our third data breach this year. We need to access the logs of the time before he left but they are all in encrypted zips. We remember he used pretty similar passwords each time.

The challenge files are 4 password protected zip files:
```shell
ls  
a_USE.zip  b_JOHN.zip  c_THE.zip  d_RIPPER.zip
```

I generate hashes with `zip2john`:
```shell
zip2john a_USE.zip > zip.hash.a  

zip2john b_JOHN.zip > zip.hash.b  

zip2john c_THE.zip > zip.hash.c  

zip2john d_RIPPER.zip > zip.hash.d
```

The first 3 passwords crack immediately:
```shell
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash.a  
Using default input encoding: UTF-8  
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])  
Cost 1 (HMAC size) is 366 for all loaded hashes  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
chips            (a_USE.zip/USE.log)  
1g 0:00:00:00 DONE (2026-06-14 19:13) 1.724g/s 35310p/s 35310c/s 35310C/s cowgirlup..mitch1  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.  

john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash.b  
Using default input encoding: UTF-8  
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])  
Cost 1 (HMAC size) is 89 for all loaded hashes  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
fishandchips     (b_JOHN.zip/JOHN.log)  
1g 0:00:00:00 DONE (2026-06-14 19:13) 1.176g/s 38550p/s 38550c/s 38550C/s squirtle..elgordo  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.  

john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash.c  
Using default input encoding: UTF-8  
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])  
Cost 1 (HMAC size) is 69 for all loaded hashes  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
sunchips         (c_THE.zip/THE.log)  
1g 0:00:00:02 DONE (2026-06-14 19:13) 0.3984g/s 42428p/s 42428c/s 42428C/s birthstone..harrybo  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

- zip password a: `chips`
- zip password b: `fishandchips`
- zip password c: `sunchips`

The fourth zip does not crack with `rockyou.txt`.

I unzipped the first three and got these files:
```shell
cat USE.log  
[2026-02-09 17:20:01] INFO  Idle process heartbeat: System state nominal.  
[2026-02-09 17:20:11] INFO  Routine polling: No updates found.  
[2026-02-09 17:20:21] DEBUG Thread id=0842 reporting status: Awaiting instruction.  
[2026-02-09 17:20:31] INFO  Idle process heartbeat: System state nominal.  
[2026-02-09 17:20:41] TRACE Buffer check: 0 bytes pending.  
[2026-02-09 17:20:51] INFO  Routine polling: No updates found.  
[2026-02-09 17:21:01] INFO  Idle process heartbeat: System state nominal.  
[2026-02-09 17:21:11] DEBUG Maintenance task 'noop' completed in 0.001ms.  
[2026-02-09 17:21:21] INFO  Routine polling: No updates found.  
[2026-02-09 17:21:31] INFO  Idle process heartbeat: System state nominal.  
[2026-02-09 17:21:41] TRACE Cache integrity: Verified (unchanged).  
[2026-02-09 17:21:51] INFO  Routine polling: No updates found.  
[2026-02-09 17:22:01] INFO  Idle process heartbeat: System state nominal.  
[2026-02-09 17:22:11] DEBUG Keep-alive packet sent to localhost.  
[2026-02-09 17:22:21] INFO  Routine polling: No updates found.  
[2026-02-09 17:22:31] INFO  Idle process heartbeat: System state nominal.  
[2026-02-09 17:22:41] TRACE Stack scan: No anomalies detected.  
[2026-02-09 17:22:51] INFO  Routine polling: No updates found.  
[2026-02-09 17:23:01] INFO  Idle process heartbeat: System state nominal.  
[2026-02-09 17:23:11] DEBUG System clock sync: 0.000s offset.

cat JOHN.log  
[2026] INFO PASSWORD INSECURE PLEASE MOVE LOGS  
[2026] MESSAGE SYSADMIN "fine I'll move them"

cat THE.log  
[2026] INFO PLEASE MAKE BETTER PASSWORDS  
[2026] SYSADMIN MESSAGE "NO"
```

To crack the final zip I put all strings in `rockyou.txt` containing the word `chip` into a seperate text file:
```shell
cat /usr/share/wordlists/rockyou.txt | grep chip >> zip/chip-words.txt

cat chip-words.txt | tail  
,3chip,3  
*chips*!  
*chipmunk  
(123chipshop123)  
$chippy$  
#chipie1985  
#1chipper  
#1chipmunk  
#10chipper  
!chipmunk!
```

Then run `john` again (`--rules=best64` applies a specific set of 64 word mangling rules to expand the base wordlist):
```shell
john --wordlist=chip-words.txt --rules=best64 zip.hash.d  
Using default input encoding: UTF-8  
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])  
Cost 1 (HMAC size) is 86 for all loaded hashes  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
chip!            (d_RIPPER.zip/RIPPER.log)  
1g 0:00:00:04 DONE (2026-06-14 19:35) 0.2409g/s 41453p/s 41453c/s 41453C/s 1chphooter..savc  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

I get the password `chip!`

```shell
7z x d_RIPPER.zip -pchip!  
  
7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03  
 64-bit locale=en_US.UTF-8 Threads:128 OPEN_MAX:1024, ASM  
  
Scanning the drive for archives:  
1 file, 290 bytes (1 KiB)  
  
Extracting archive: d_RIPPER.zip  
--  
Path = d_RIPPER.zip  
Type = zip  
Physical Size = 290  
  
  
Would you like to replace the existing file:  
  Path:     ./RIPPER.log  
  Size:     0 bytes  
  Modified: 2026-02-09 18:31:15  
with the file from archive:  
  Path:     RIPPER.log  
  Size:     86 bytes (1 KiB)  
  Modified: 2026-02-09 18:31:15  
? (Y)es / (N)o / (A)lways / (S)kip all / A(u)to rename all / (Q)uit? y  
  
Everything is Ok  
  
Size:       86  
Compressed: 290

cat RIPPER.log  
[2026] ALL PASSWORDS HAVE BEEN BREACHED  
[2026] SYSADMIN MESSAGE "boroCTF{L@_R11pP3r;}
```

`boroCTF{L@_R11pP3r;}`
