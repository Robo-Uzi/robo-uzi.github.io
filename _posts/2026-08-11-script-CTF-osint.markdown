---
layout: post
title:  "Osint Challenges"
date:   2026-08-11 20:55:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /script-CTF-2026-osint/
---
* TOC
{:toc}

## Midnight Snack

**Category**: geo-osint

**Description**: Can you find the address of this Taco Bell? Example: `scriptCTF{1337_Orange_St}`

I get the challenge image:

![Alt text](/images/tacobell.png)

I can see the word `Parmer` on the sign in the back. Based on the clue `Parmer`, I found `Parmer Lane` in `Austin, Texas`. The business behind the taco bell in the image is `Parmer Eye Care`. I found the location on google maps: [google maps link](https://www.google.com/maps/place/Taco+Bell/@30.486316,-97.7700737,272a,33.8y,21.93h,84.76t/data=!3m7!1e1!3m5!1ssWjSDZ58m18VCZ9cKnWMxw!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D5.24427830957876%26panoid%3DsWjSDZ58m18VCZ9cKnWMxw%26yaw%3D21.925206060414975!7i16384!8i8192!4m15!1m8!3m7!1s0x8644d2a410600007:0xb6c0c0e4a688faf3!2s9900+W+Parmer+Ln,+Austin,+TX+78717,+USA!3b1!8m2!3d30.4870133!4d-97.7696848!16s%2Fg%2F11rp1x47sx!3m5!1s0x8644d2a41b846359:0x1bcd572dddb3b727!8m2!3d30.4865693!4d-97.7701047!16s%2Fg%2F1tmxrdsl?entry=ttu&g_ep=EgoyMDI2MDgwNS4xIKXMDSoASAFQAw%3D%3D)

`scriptCTF{9900_W_Parmer_Ln}`

___

## The New One 1

**Category**: osint

**Author**: NoobMaster

**Description**: "Can I join your team?" - Armored Pawn  
  
"Nah, we don't need more members" - NoobMaster  
  
_Proceeds to let a new member join that is not Armored Pawn_  
  
Note: Please do not OSINT Armored Pawn, he is not related to the challenge, just a troll in our server :)

I went to [https://ctftime.org](https://ctftime.org) and found their team. They linked their website [https://scriptsorcerers.xyz](https://scriptsorcerers.xyz). On `/members` there is a new member listed. The flag was on [https://scriptsorcerers.xyz/members/john.hacker.doe1337](https://scriptsorcerers.xyz/members/john.hacker.doe1337). 

`scriptCTF{17s_0bv10usly_0S1NT_71m3}`
