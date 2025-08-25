---
layout: post
title:  "BrunnerCTF 2025 Forensics Challenges"
date:   2025-08-24 22:18:10 -0400
author: robo.uzi
tags: [CTF, forensics]
---
* TOC
{:toc}

## DoughBot
**Category:** Forensics

**Difficulty:** Beginner  
**Author:** rvsmvs
Our state-of-the-art smart mixer, DoughBot, crashed during a routine kneading cycle. Luckily, a technician was monitoring the device over UART and captured the memory output just before the reboot.

Analyze the captured dump and see what the DoughBot was trying to say before it rebooted.
`forensics_doughbot.zip`
```shell
file doughbot_dump.bin  
doughbot_dump.bin: Windows SYSTEM.INI

cat doughbot_dump.bin  
[BOOT] DoughBot 1.2.4  
[INFO] Initializing sensor calibration...  
[INFO] Sensor calibration complete.  
[INFO] Establishing Wi-Fi connection...  
[WARN] Wi-Fi unstable, retrying...  
[INFO] Uploading diagnostics...  
[DEBUG] Loading configuration from EEPROM...  
[EEPROM CONFIG DUMP @ 0x2000]  
   device_name     = "DoughBot"  
   firmware_ver    = 1.2.4  
   knead_duration  = 780  
   mix_speed       = AUTO  
   safety_timeout  = 300  
   temp_unit       = "C"  
   debug_enabled   = true  
   log_mode        = FULL  
  
// dev.note: bootlog_flag=YnJ1bm5lcnttMXgzZF9zMWduYWxzXzRfc3VyZX0=  
  
[CRASH] Unexpected interrupt.  
[REBOOT] Attempting recovery boot...  
[WARN] ??>!!%0^ [RECV_ERR]  3499$  
[WARN] 0x00ff @@@ERROR@:~:~  
[BOOT] Safe Mode Active.  

echo 'YnJ1bm5lcnttMXgzZF9zMWduYWxzXzRfc3VyZX0=' | base64 -d  
brunner{m1x3d_s1gnals_4_sure}
```

___

## The Secret Brunsviger
**Category:** Forensics

**Difficulty:** Beginner  
**Author:** Ha1fdan

I have intercepted encrypted HTTPS traffic from the secret brunsviger baking forum, but I need help decrypting it.

I am given two files:
```shell
file *  
keys.log:     ASCII text  
traffic.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 262144)
```

I open the pcap in wireshark. Go to Edit > Preferences > Protocols > TLS > (Pre)-Master-Secret log filename > choose `keys.log`. I filter by http and now can see the plaintext. There were 9 GET requests to `/api/messages/1` through `/api/messages/9`. I follow the http stream on 9 and see a base64 string I can decode to get the flag:
```shell
echo 'YnJ1bm5lcntTM2NyM3RfQnJ1bnp2MWczcl9SM2MxcDNfRnIwbV9HcjRuZG00c19DMDBrYjAwa30=' | base64 -d  
brunner{S3cr3t_Brunzv1g3r_R3c1p3_Fr0m_Gr4ndm4s_C00kb00k}
```

___

## Memory Loss
**Category:** Forensics

**Difficulty:** Medium  
**Author:** ha1fdan

I had just finished baking a _brunsviger_ when I suddenly remembered something important... but now I simply can't recall what it was! I'm pretty sure I took a picture of it, but where did I put it?

**Download link:** `forensics_memory-loss.7z`
**Password:** `VerySecurePasswordForMemoryLoss_BrunnerCTF2025`

Extract the archive: `7z x forensics_memory-loss.7z -pVerySecurePasswordForMemoryLoss_BrunnerCTF2025`

I get this file:
```shell
file memoryloss.dmp  
memoryloss.dmp: MS Windows 64bit crash dump, full dump, 524173 pages
```
I run `git clone https://github.com/volatilityfoundation/volatility3.git` and check I can run it with `python3 vol.py -f ../memoryloss.dmp windows.info`.

I extract all cached files from memory with `python3 vol.py -f ../memoryloss.dmp -o ../dumped_files windows.dumpfiles`. 

In `dumped_files` directory I run:
```shell
file * | grep -iE 'jpg|png'  
file.0xb207c3ab75a0.0xb207bef99c30.DataSectionObject.{798C16B5-BC0A-49FB-921E-AA0FEE767691}.png.dat: PNG image data, 1597 x 1195, 8-b  
it/color RGBA, non-interlaced  
file.0xb207c3ad4740.0xb207c32aa690.DataSectionObject.TileCache_100_4_PNGEncoded_Data.bin.dat: data  
file.0xb207c3ad4740.0xb207c3a25930.SharedCacheMap.TileCache_100_4_PNGEncoded_Data.bin.vacb: data
```

Looks like there was 1 image in memory. I run `xdg-open file.0xb207c3ab75a0.0xb207bef99c30.DataSectionObject.{798C16B5-BC0A-49FB-921E-AA0FEE767691}.png.dat` and see the flag: `brunner{0h_my_84d_17_w45_ju57_1n_my_m3m0ry}`

___

