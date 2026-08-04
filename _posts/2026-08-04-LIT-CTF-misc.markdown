---
layout: post
title:  "Misc Challenges"
date:   2026-08-04 18:07:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /LIT-CTF-2026-misc/
---
* TOC
{:toc}
## introduction

**Author**: joshadoodle

**Category**: misc

**Description**: Were you paying attention?

In discord I see this message:
```shell
@everyone @LIT 2026 Participant

The Capture the Flag round of LIT 2026 has officially begun! Good luck!

You can view challenges by going to https://lit.lhsmathcs.org/ctf/challenges. All challenges are initially worth 500 points, except the introductory one which is worth 100 points. Remember that the more people that solve a challenge, the fewer number of points they are worth. Note that challenges are not sorted by difficulty. If you would like, you can sort them by category using the menu bar to the left. Good luck and have fun!

Here are the slides for our opening ceremony yesterday. Make sure to watch it if you haven't yet!
https://docs.google.com/presentation/d/1ZzHA1GmfSN_8RCn36SaFxbtMnENU1XjdK5_h__rxqtk/edit?usp=sharing

p4Y1nG-4tT3nT10n?}
```

The second part of the flag is in the discord message. The first part of the flag is in very small font on slide 24 of the google docs presentation.

`LITCTF{w3R3-Y0u-p4Y1nG-4tT3nT10n?}`

___

## pw.zip

**Author**: halp

**Category**: misc

**Description**: Thought this password-protected zip would keep the flag safe, but I just found out the password I used was one of the 100 most used passwords of 2025!

I get the challenge files `pw.zip`:
```shell 
file pw.zip  
pw.zip: Zip archive data, made by v6.3, extract using at least v5.1, last modified Feb 01 2026 13:25:24, uncompressed size 32, method=AES Encrypted
```
It is password protected.

Generate a hash with `zip2john` and crack the password with `john`:
```shell
zip2john pw.zip > zip.hash  

john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash  
Using default input encoding: UTF-8  
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])  
Cost 1 (HMAC size) is 32 for all loaded hashes  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
1234qwer         (pw.zip/flag.txt)  
1g 0:00:00:00 DONE (2026-08-01 17:27) 2.083g/s 8533p/s 8533c/s 8533C/s 123456..bigman  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

The password is `1234qwer`!

`LITCTF{password_123456_103flkj3}`

___

## kcufniarb

**Author**: halp

**Category**: misc

**Description**: `----------]<-----<-----------<--------<------->>>>+[<<<++++,+++,-----------,<+++,>,<---,>>-------------,+++++++,+++++++++++,++++++++++,-----,-----,+++++,+++++,-------------------,---,++++++,----------,<-----------,>>,-,-,<+++++++,++++++++,-------------,<--,---,-----------,>>,<<++++++,>>+,<------,`

At first I though it was `BrainFuck` becuase that is the title reversed. Eventually I realized I needed to run the code on [https://www.dcode.fr/reversefuck-language](https://www.dcode.fr/reversefuck-language)

`LITCTF{ti_did_ruoy_234rjwado4i3}`

___

## git genug

**Author**: Ninjaprime

**Category**: misc

**Description**: We accidentally leaked a secret in this repo at some point but we think we cleaned it up before pushing. Can you prove us wrong?

I get the challenge files:
```shell
unzip gitgenug.zip  
Archive:  gitgenug.zip  
warning:  gitgenug.zip appears to use backslashes as path separators  
  inflating: repo/app.py  
  inflating: repo/config.py  
  inflating: repo/README.md  
  inflating: repo/utils.py  
   creating: repo/.git/branches/  
   creating: repo/.git/objects/  
   creating: repo/.git/refs/  
  inflating: repo/.git/config  
  inflating: repo/.git/description  
  inflating: repo/.git/HEAD  
  inflating: repo/.git/index
... etc
```

I look around the repo for a while until I find this:
```shell
sudo cat .git/logs/refs/heads/main  
0000000000000000000000000000000000000000 d2875dde8c6e27840fd503bcf038da7765b68c40 LITCTF Admin <admin@litctf.local> 1785006413 +0000    commit (initial): Initial commit  
d2875dde8c6e27840fd503bcf038da7765b68c40 f1c1c0ca613adeb219c8a527f01a3a6d9e91e2d3 LITCTF Admin <admin@litctf.local> 1785006413 +0000    commit: Add config  
f1c1c0ca613adeb219c8a527f01a3a6d9e91e2d3 e5ed944148c81a7647894ca0752379aab3787f4a LITCTF Admin <admin@litctf.local> 1785006413 +0000    commit: Add local dev notes (oops, forgot to gitignore)  
e5ed944148c81a7647894ca0752379aab3787f4a f1c1c0ca613adeb219c8a527f01a3a6d9e91e2d3 LITCTF Admin <admin@litctf.local> 1785006413 +0000    reset: moving to HEAD~1  
f1c1c0ca613adeb219c8a527f01a3a6d9e91e2d3 18cd672d654e64399f853d57a002645ddb0ce888 LITCTF Admin <admin@litctf.local> 1785006413 +0000    commit: Add utils module  
18cd672d654e64399f853d57a002645ddb0ce888 9d97888f560e69f8b8a6c57148125af1a05695f8 LITCTF Admin <admin@litctf.local> 1785006413 +0000    commit: Add greet function
```

I check the commit where they says "oops, forgot to gitignore"!
```shell
sudo git show e5ed944148c81a7647894ca0752379aab3787f4a  
commit e5ed944148c81a7647894ca0752379aab3787f4a  
Author: LITCTF Admin <admin@litctf.local>  
Date:   Sat Jul 25 19:06:53 2026 +0000  
  
	Add local dev notes (oops, forgot to gitignore)  
  
diff --git a/secret.txt b/secret.txt  
new file mode 100644  
index 0000000..d40dfe4  
--- /dev/null  
+++ b/secret.txt  
@@ -0,0 +1,2 @@  
+TODO: remove before pushing  
+LITCTF{r3fl0g_1s_y0ur_g1t_t1m3_m4ch1n3}
```

`LITCTF{r3fl0g_1s_y0ur_g1t_t1m3_m4ch1n3}`

___

## be social

**Author**: ethan

**Category**: misc

**Description**: Our friend John might've been a little too careless this time...  
Wrap your flag with LITCTF{}. `https://bsky.app/profile/jsmithf.bsky.social`, `http://136.115.87.65:31781/`

I go to the bluesky profile and there is two posts:

![Alt text](/images/litosint1.png)

When I go to `http://136.115.87.65:31781/` it first asks for a 4 digit pin:

![Alt text](/images/litosint2.png)

The pin is `0725`.

Then the page asks for `My biggest secret`. I entered `cooper`:

![Alt text](/images/litosint3.png)

`LITCTF{os_1nt_success_4f2a58b9}`

___

## one does not simply hide a flag

**Author**: Ninjaprime

**Category**: misc

**Description**: Something has slipped past the Black Gate. Frodo left behind a single image before he vanished into Mordor; analysts say it's not what it appears to be.

I get the challenge file:

![Alt text](/images/mordor_challenge.png)

I ran `foremost` and got a zip file which was password protected:
```shell
unzip output/zip/00000008.zip  
Archive:  output/zip/00000008.zip  
[output/zip/00000008.zip] shire.png password:
```

Crack the password with `zip2john` and `john`:
```shell
zip2john output/zip/00000008.zip > zip2.hash  
ver 2.0 efh 5455 efh 7875 00000008.zip/shire.png PKZIP Encr: TS_chk, cmplen=4131, decmplen=4670, crc=CBC0329A ts=BBE6 cs=bbe6 type=8

john --wordlist=/usr/share/wordlists/rockyou.txt zip2.hash  
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
time             (00000008.zip/shire.png)  
1g 0:00:00:00 DONE (2026-08-02 21:26) 2.631g/s 409600p/s 409600c/s 409600C/s army69..shelly3  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

The password is `time`.

I unzipped the file and got a new image called `shire.png`:

![Alt text](/images/shire.png)

The flag is hidden with lsb:
```shell
zsteg shire.png  
b1,r,lsb,xy         .. text: "RIDDLE: Voiceless it cries, wingless flutters, toothless bites, mouthless mutters. -- Gollum, Riddles in the Dark\nCIPHER: HQGFPN{elzlyho_eedlxrg_ev_cltmyv_wvq_yazfh}"
```

I put the flag into [https://www.guballa.de/vigenere-solver](https://www.guballa.de/vigenere-solver) and it brute forced the [Vigenère cipher](https://en.wikipedia.org/wiki/Vigenère_cipher) using the keyword `wind`.

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=Vigen%C3%A8re_Decode('wind')&input=SFFHRlBOe2Vsemx5aG9fZWVkbHhyZ19ldl9jbHRteXZfd3ZxX3lhemZofQ)

`LITCTF{riddles_wrapped_in_pixels_and_verse}`
