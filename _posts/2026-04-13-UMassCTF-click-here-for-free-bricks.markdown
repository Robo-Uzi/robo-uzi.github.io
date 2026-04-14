---
layout: post
title:  "Click Here For Free Bricks"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-click-here-for-free-bricks/
---

**Category**: Wireshark medium

**Author**: non_gonzo

**Description**: Hey! A man was caught with malware on his PC in Lego City. Luckily, we were able to get a packet capture of his device during the download. Help Lego City Police figure out the source of this malicious download.

The flag for this challenge is the name of this virus on VirusTotal under the details tab with the format `UMASS{[String]_[Sha256 Hash]}`. For example if your hash is `2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824`, there will be a virus name under the details tab with a format similar to `ForensicsChallenge_2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824`, where "ForensicsChallenge" is an arbitrary string.

I get the challenge file:
```shell
file thedamage.pcapng  
thedamage.pcapng: pcapng capture file - version 1.0
```

I open the pcap in wireshark and filter for `http`. I find this http stream:
```http
GET /installer.py HTTP/1.1
Host: 156.234.52.16
User-Agent: python-requests/2.31.0
Accept-Encoding: gzip, deflate, br
Accept: */*
Connection: keep-alive

HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.12.3
Date: Sat, 04 Apr 2026 21:43:13 GMT
Content-type: text/x-python
Content-Length: 614
Last-Modified: Sat, 04 Apr 2026 21:11:16 GMT

import subprocess
import subprocess
import hashlib
import nacl.secret

def fix_error():
	seed = "38093248092rsjrwedoaw3"
	key = hashlib.sha256(seed.encode()).digest()
	box = nacl.secret.SecretBox(key)
		with open("./launcher", "rb") as f:
	data = f.read()
	decrypted = box.decrypt(data)
	with open("./launcher", "wb") as f:
		f.write(decrypted)

print("Hello World")

try:
	fix_error()
	print("Installed Correctly")
	result = subprocess.run(["ping", "-c", "2", "76.54.32.144"])
	print(result)

except Exception as e:
	print(f"Installation failed, please try again {e}")
```

I also find an http object called `launcher` (the encrypted virus):
```shell
file launcher  
launcher: data
```

I run this script to decrypt the virus:
```python
import hashlib
from nacl.secret import SecretBox

# read encrypted file
with open("launcher", "rb") as f:
    ciphertext = f.read()

# recreate key from seed
seed = "38093248092rsjrwedoaw3"
key = hashlib.sha256(seed.encode()).digest()

# decrypt
box = SecretBox(key)

try:
    plaintext = box.decrypt(ciphertext)
    with open("launcher.decrypted", "wb") as f:
        f.write(plaintext)

    print("Decryption successful! :)")
    print("Saved as launcher.decrypted")

except Exception as e:
    print("Decryption failed :(", e)
```

Now I have the virus:
```shell
python3 solve.py  
Decryption successful! :)  
Saved as launcher.decrypted  

file launcher.decrypted  
launcher.decrypted: FreeBSD/i386 compact demand paged dynamically linked executable not stripped  

sha256sum file launcher.decrypted  
sha256sum: file: No such file or directory  
e7a09064fc40dd4e5dd2e14aa8dad89b328ef1b1fdb3288e4ef04b0bd497ccae  launcher.decrypted
```

I go to VirusTotal and find the flag at [https://www.virustotal.com/gui/file/e7a09064fc40dd4e5dd2e14aa8dad89b328ef1b1fdb3288e4ef04b0bd497ccae/details](https://www.virustotal.com/gui/file/e7a09064fc40dd4e5dd2e14aa8dad89b328ef1b1fdb3288e4ef04b0bd497ccae/details). 

`UMASS{TheZoo_e7a09064fc40dd4e5dd2e14aa8dad89b328ef1b1fdb3288e4ef04b0bd497ccae}`