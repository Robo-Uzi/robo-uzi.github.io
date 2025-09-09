---
layout: post
title:  "obfuscated-1"
date:   2025-09-08 21:26:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /imaginaryctf-2025-obfuscated-1/
---

**Title:** obfuscated-1

**Category:** Forensics

**Author:** Eth007

**Description:** I installed every old software known to man... The flag is the VNC password, wrapped in `ictf{}`.

Attached is `Users.zip`. I unzip the file. There are standard windows profile folders as well as registry hive files:
```shell
ls -l  
total 1908  
dr-xr-xr-x 1 user user     22 Sep  1 13:27 '3D Objects'  
drwxr-xr-x 1 user user     40 Sep  6 05:10  AppData  
dr-xr-xr-x 1 user user     22 Sep  1 13:28  Contacts  
dr-xr-xr-x 1 user user    102 Sep  1 13:28  Desktop  
dr-xr-xr-x 1 user user     22 Sep  1 13:28  Documents  
dr-xr-xr-x 1 user user    166 Sep  1 13:31  Downloads  
dr-xr-xr-x 1 user user     48 Sep  1 13:28  Favorites  
dr-xr-xr-x 1 user user     70 Sep  1 13:28  Links  
dr-xr-xr-x 1 user user     22 Sep  1 13:28  Music  
-rw-r--r-- 1 user user 786432 Sep  1 13:27  NTUSER.DAT  
-rw-r--r-- 1 user user  65536 Sep  1 13:04  NTUSER.DAT{...}.TM.blf  
-rw-r--r-- 1 user user 524288 Aug 31 18:36  NTUSER.DAT...TMContainer..regtrans-ms
-rw-r--r-- 1 user user 524288 Aug 31 18:36  NTUSER.DAT..TMContainer..regtrans-ms 
-rw-r--r-- 1 user user      0 Aug 31 18:36  ntuser.dat.LOG1  
-rw-r--r-- 1 user user  49152 Aug 31 18:36  ntuser.dat.LOG2  
-rw-r--r-- 1 user user     20 Aug 31 18:36  ntuser.ini  
dr-xr-xr-x 1 user user     22 Sep  1 13:28  Pictures  
dr-xr-xr-x 1 user user     22 Sep  1 13:28 'Saved Games'  
dr-xr-xr-x 1 user user    116 Sep  1 13:28  Searches  
dr-xr-xr-x 1 user user     22 Sep  1 13:28  Videos
```

I am most interested in `NTUSER.DAT` as this should contain registry keys and settings for some programs. It says the flag is the VNC password so I search for VNC in `NTUSER.DAT`: 
```shell
reglookup NTUSER.DAT | grep -i VNC

/Software/TightVNC/Server/EnableUrlParams,DWORD,0x00000001,  
/Software/TightVNC/Server/Password,BINARY,~%9B1%12H%B7%C8%A8,  
/Software/TightVNC/Server/AlwaysShared,DWORD,0x00000000,
```

Looks like the server password for `TightVNC`, but it is obfuscated. I search into how this is obfuscated. I find the answer here [https://github.com/frizb/PasswordDecrypts](https://github.com/frizb/PasswordDecrypts). 

The author says: VNC uses a hardcoded DES key to store credentials. The same key is used across multiple product lines. 

It says the key is `"\x17\x52\x6b\x06\x23\x4e\x58\x07"`. I go to [https://stackoverflow.com/questions/16469487/vnc-des-authentication-algorithm](https://stackoverflow.com/questions/16469487/vnc-des-authentication-algorithm) and read about how the authentication works. It says: `When you make 8-bytes password from source password, you should reverse order of bits in bytes.`

To get `7E9B311248B7C8A8` (the password in hex) I went to cyberchef and ran this recipe: [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/#recipe=URL_Decode(true)To_Hex('Space',0)Remove_whitespace(true,true,true,true,true,false)&input=fiU5QjElMTJIJUI3JUM4JUE4). 

Make a python script using `pycryptodome` to get the plaintext password:
```python
from Crypto.Cipher import DES

data = bytes.fromhex("7E9B311248B7C8A8")

# VNC DES key
key = bytes([0x17,0x52,0x6B,0x06,0x23,0x4E,0x58,0x07])

# reverse bits in a byte
def rev_bits(b):
    return int('{:08b}'.format(b)[::-1], 2)

# reverse bits of each byte in key
key_reversed = bytes(rev_bits(b) for b in key)

cipher = DES.new(key_reversed, DES.MODE_ECB)
passwd = cipher.decrypt(data).rstrip(b'\x00')
print("VNC password:", passwd.decode('utf-8', errors='replace'))
```

Run the script:
```shell
python3 solve.py  
VNC password: Slay4U!!
```

You can also do it in `msfconsole` in a ruby shell:
```shell
[msf](Jobs:0 Agents:0) >> irb  
[*] Starting IRB shell...  
[*] You are in the "framework" object  
  
irb: warn: can't alias jobs from irb_jobs.  
>> fixedkey = "\x17\x52\x6b\x06\x23\x4e\x58\x07"  
=> "\x17Rk\x06#NX\a"  
>> require 'rex/proto/rfb'  
=> true  
>> Rex::Proto::RFB::Cipher.decrypt ["7E9B311248B7C8A8"].pack('H*'), fixedkey  
=> "Slay4U!!"  
>>
```
`ictf{Slay4U!!}`