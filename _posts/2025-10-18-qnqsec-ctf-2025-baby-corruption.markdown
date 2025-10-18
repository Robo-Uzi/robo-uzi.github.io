---
layout: post
title:  "Baby Corruption"
date:   2025-10-18 16:12:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-baby-corruption/
---

**Title:** Baby Corruption

**Category:** forensics

**Description:** Catch me if you can, Scan me if you can!

**Author:** Azadin

I get the file `ScanMe.kdbx`:
```shell
file ScanMe.kdbx  
ScanMe.kdbx: Keepass password database 2.x KDBX
```

I try to use `keepass2john` but there is an issue:
```shell
/usr/sbin/keepass2john ScanMe.kdbx > scanme.hash  
! ScanMe.kdbx : File version '40000' is currently not supported!
```

To crack the password for the keepass database I run `pip install pykeepass`, then this script:
```python
from pykeepass import PyKeePass, exceptions
import sys

kdbx='ScanMe.kdbx'
wordlist='/usr/share/wordlists/rockyou.txt'
keyfile=None

def try_open(pw):
    try:
        if keyfile:
            kp = PyKeePass(kdbx, password=pw, keyfile=keyfile)
        else:
            kp = PyKeePass(kdbx, password=pw)
        return kp
    except exceptions.CredentialsError:
        return None
    except Exception as e:
        print("Other error:", e)
        return None

with open(wordlist,'r',errors='ignore') as f:
    for i,line in enumerate(f,1):
        pw=line.strip()
        kp=try_open(pw)
        if kp:
            print("FOUND:", pw)
            for e in kp.entries:
                print("Title:", e.title)
                print("User:", e.username)
                print("Pass:", e.password)
                print("Notes:", e.notes)
                for a in e.attachments:
                    fname=f"attachment_{e.uuid}_{a.filename}"
                    open(fname,'wb').write(a.data)
                    print("Saved", fname)
            sys.exit(0)
        if i%1000==0:
            print("Tried",i,"passwords")
print("No password found")
```

After waiting a few minutes I get this output:
```shell
python3 solve6.py  
FOUND: superman  
Title: Flag  
User: kafka  
Pass: superman  
Notes: Mine has been a life of much shame.  
  
Saved attachment_6d82ca76-59db-4850-968e-c090c94e3800_flag.png
```

I try to open this `.png` however it says `Could not load image` and `Fatal error reading PNG image file: Not a PNG file`. I inspect it with xxd however I wasnt sure how it was corrupted. The png header looked correct and so did the end of the file.

I ask chatgpt to make me a png recovery script:
```python
#!/usr/bin/env python3
from pathlib import Path

IN = Path("attachment_6d82ca76-59db-4850-968e-c090c94e3800_flag.png")
OUT = Path("flag_fixed.png")

data = IN.read_bytes()
png_sig = b"\x89PNG\r\n\x1a\n"
png_fallback = b"PNG\r\n\x1a\n"

# locate start
start = data.find(png_sig)
if start == -1:
    start = data.find(png_fallback)
    if start != -1:
        data = b"\x89" + data[start:]        # add missing 0x89
    else:
        raise SystemExit("No PNG signature found")

# locate end at final IEND + 4-byte CRC
iend = data.rfind(b"IEND")
if iend == -1:
    repaired = data[start:]                # no IEND. keep from signature
else:
    end = iend + 4 + 4                     # 'IEND' (4) + CRC (4)
    repaired = data[:min(end, len(data))]

OUT.write_bytes(repaired)
print(f"WROTE {OUT} ({len(repaired)} bytes)")
```

After running this I get an image of a long QR code:

![Alt text](/images/flag_fixed.png)

I go to [https://qrscanner.net/](https://qrscanner.net/) and upload the QR code and get the flag.

`QnQSec{g0tta_l0v3_rmqr_cod3s}`