---
layout: post
title:  "Forensics Challenges"
date:   2026-03-09 20:59:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UTCTF-2026-forensics/
---
* TOC
{:toc}

## Half Awake

**Category**: forensics

**Description**: Our SOC captured suspicious traffic from a lab VM right before dawn. Most packets look like ordinary client chatter, but a few are pretending to be something they are not.

I get the challenge file:
```shell
file half-awake.pcap  
half-awake.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

I open it in wireshark. There are `tcp`, `http`, `mDNS`, and `TLSv1.2` packets. In one http packet I find this message:
```shell
cat instructions.hello  
Read this slowly:  
1) mDNS names are hints: alert.chunk, chef.decode, key.version  
2) Not every 'TCP blob' is really what it pretends to be  
3) If you find a payload that starts with PK, treat it as a file
```

Then in a `TLSv1.2` packet I see this:
```shell
0000   ff ff ff ff ff ff 82 8f 99 4a 74 55 08 00 45 00   .........JtU..E.
0010   01 5f 00 01 00 00 40 06 f2 38 4b 65 79 de c0 a8   ._....@..8Key...
0020   01 74 ad 38 01 bb 00 00 a6 30 00 00 d1 74 50 18   .t.8.....0...tP.
0030   20 00 62 01 00 00 15 03 03 01 32 50 4b 03 04 14    .b.......2PK...
0040   00 00 00 08 00 aa ba 6b 5c eb 92 16 71 2e 00 00   .......k\...q...
0050   00 29 00 00 00 0a 00 00 00 73 74 61 67 65 32 2e   .).......stage2.
0060   62 69 6e 01 29 00 d6 ff 75 c3 66 db 61 d0 7b df   bin.)...u.f.a.{.
0070   34 db 66 e8 61 c0 34 dc 33 e8 73 84 33 e8 74 df   4.f.a.4.3.s.3.t.
0080   33 e8 70 c5 30 c3 30 d4 30 db 5f c3 72 86 63 dc   3.p.0.0.0._.r.c.
0090   7d 50 4b 03 04 14 00 00 00 08 00 aa ba 6b 5c 9c   }PK..........k\.
00a0   fe 88 9d 2e 00 00 00 2e 00 00 00 0a 00 00 00 72   ...............r
00b0   65 61 64 6d 65 2e 74 78 74 05 c1 41 0e 00 10 0c   eadme.txt..A....
00c0   04 c0 bb 57 ec d7 84 8d f6 a0 a4 1a d2 df 9b b1   ...W............
00d0   15 e0 a5 67 88 da 80 d0 09 3d a0 35 cf 1d ec 08   ...g.....=.5....
00e0   21 4e 9d c4 ab 59 3e 50 4b 01 02 14 03 14 00 00   !N...Y>PK.......
00f0   00 08 00 aa ba 6b 5c eb 92 16 71 2e 00 00 00 29   .....k\...q....)
0100   00 00 00 0a 00 00 00 00 00 00 00 00 00 00 00 80   ................
0110   01 00 00 00 00 73 74 61 67 65 32 2e 62 69 6e 50   .....stage2.binP
0120   4b 01 02 14 03 14 00 00 00 08 00 aa ba 6b 5c 9c   K............k\.
0130   fe 88 9d 2e 00 00 00 2e 00 00 00 0a 00 00 00 00   ................
0140   00 00 00 00 00 00 00 80 01 56 00 00 00 72 65 61   .........V...rea
0150   64 6d 65 2e 74 78 74 50 4b 05 06 00 00 00 00 02   dme.txtPK.......
0160   00 02 00 70 00 00 00 ac 00 00 00 00 00            ...p.........
```

Looks like a zip file containing 2 files, `readme.txt` and `stage2.bin`. I get this file by saving the data as raw hex then running these commands:
```shell
xxd -r -p stream4 > stream4.bin  

file stream4.bin  
stream4.bin: Zip archive, with extra data prepended  

unzip stream4.bin  
Archive:  stream4.bin  
warning [stream4.bin]:  124 extra bytes at beginning or within zipfile  
  (attempting to process anyway)  
  inflating: stage2.bin  
  inflating: readme.txt

cat readme.txt  
not everything here is encrypted the same way

file stage2.bin  
stage2.bin: Non-ISO extended-ASCII text, with no line terminators

cat stage2.bin  
u�f�a�{�4�f�a�4�3�s�3�t�3�p�0�0�0�_�r�c�}
```

I find the keys to decode `stage2.bin` in an mDNS packet:
```shell
3............key.version.local......key.version.local........x...00b7
```
The mDNS `key.version` value `00b7` is an XOR key, but it's a 2 byte repeating key: `[0x00, 0xB7]`

When looking at `stage2.bin` clearly every odd byte is scrambled. XORing with `0x00` leaves even bytes unchanged and `0xB7` decrypts the odd bytes. 

Decrypt with python:
```python
data = bytes.fromhex("75c366db61d07bdf34db66e861c034dc33e8738433e874df33e870c530c330d430db5fc37286 63dc7d".replace(" ",""))
key = bytes([0x00, 0xb7])
flag = bytes(b ^ key[i % 2] for i, b in enumerate(data))
print(flag.decode())
```

Output:
```shell
python3 solve3.py  
utflag{h4lf_aw4k3_s33_th3_pr0t0c0l_tr1ck}
```

`utflag{h4lf_aw4k3_s33_th3_pr0t0c0l_tr1ck}`

___

## Cold Workspace

**Category**: forensics

**Description**: A workstation in the design lab crashed during an overnight maintenance window. By morning, a critical desktop artifact was gone and the user swore they never touched it. You only have a memory snapshot from shortly before reboot. Recover what was lost.

I get the challenge file:
```shell
file cold-workspace.dmp  
cold-workspace.dmp: MS Windows 64bit crash dump, version 0.0, MachineImageType 0x74656874, 1967416169 processors, DumpType (0x9cc0e99a), 6935312830061707549 pages
```

I could not get volatility to work so I solved this with strings. First I see the flag was encrypted with `encrypt_flag.ps1`. The encrypted data is stored in the environment variable `ENCD`, which is then written to `flag.enc`:
```shell
strings cold-workspace.dmp | grep flag  
cmdline(4608): powershell.exe -ExecutionPolicy Bypass -File C:\Users\analyst\Desktop\encrypt_flag.ps1  
%$bytes = [System.IO.File]::ReadAllBytes('C:\Users\analyst\Desktop\flag.jpg')  
Remove-Item C:\Users\analyst\Desktop\flag.jpg  
Set-Content C:\Users\analyst\Desktop\flag.enc $env:ENCD  
MFT: C:\Users\analyst\Desktop\flag.enc
```

The script references a 32 byte key and a 16 byte IV:
```shell
strings cold-workspace.dmp | grep -i "key\|cipher\|IV"
$key = [byte[]](0..31)  
$iv = [byte[]](0..15)  
# actual key/iv allocated at runtime, serialized to env vars  
$env:ENCD = '<ciphertext b64>'  
$env:ENCK = '<key b64>'  
$env:ENCV = '<iv b64>'
```
A 32 byte key corresponds to AES‑256 and a 16 byte IV matches the AES block size indicating the flag was encrypted using AES‑256‑CBC.

I grep for the environment variables:
```shell
strings cold-workspace.dmp | grep -A2 ENCK  
$env:ENCK = '<key b64>'  
$env:ENCV = '<iv b64>'  
Remove-Item C:\Users\analyst\Desktop\flag.jpg  
--  
ENCK=Ddf4BCsshqFHJxXPr5X6MLPOGtITAmXK3drAqeZoFBU=  
ENCV=xXpGwuoqihg/QHFTM2yMxA==  
SESSION=Console

strings cold-workspace.dmp | grep ENCD  
$env:ENCD = '<ciphertext b64>'  
Set-Content C:\Users\analyst\Desktop\flag.enc $env:ENCD  
4608   powershell.exe  ENCD       S4wX8ml7/f9C2ffc8vENqtWw8Bko1RAhCwLLG4vvjeT2iJ26nfeMzWEyx/HlK1KmOhIrSMoWtmgu2OKMtTtUXddZDQ87FTEXIqghzCL6ErnC1+GwpSfzCDr9...<truncated>  
4ENV_BLOCK_START::PID=4608::powershell.exe::ENCD=S4wX8ml7/f9C2ffc8vENqtWw8Bko1RAhCwLLG4vvjeT2iJ26nfeMzWEyx/HlK1KmOhIrSMoWtmgu2OKMtTtUXddZDQ87FTEXIqghzCL6ErnC1+GwpSfzCDr9woKXj5IzcU2C/Ft5u705bY3b6/Z/Q/N6MPLXV55pLzIDnO1nvtja123WWwH54O4mnyWNspt5
```

Decode the values and decrypt the ciphertext using OpenSSL:
```shell
echo 'S4wX8ml7/f9C2ffc8vENqtWw8Bko1RAhCwLLG4vvjeT2iJ26nfeMzWEyx/HlK1KmOhIrSMoWtmgu2OKMtTtUXddZDQ87FTEXIqghzCL6ErnC1+GwpSfzCDr9woKXj5IzcU2C/Ft5u705bY3b6/Z/Q/N6MPLXV55pLzIDnO1nvtja123WWwH54O4mnyWNspt5' > encd.txt  
echo 'Ddf4BCsshqFHJxXPr5X6MLPOGtITAmXK3drAqeZoFBU=' > enck.txt  
echo 'xXpGwuoqihg/QHFTM2yMxA==' > encv.txt  

base64 -d encd.txt > cipher.bin  
base64 -d enck.txt > key.bin  
base64 -d encv.txt > iv.bin  

openssl enc -aes-256-cbc -d -in cipher.bin -out flag.jpg -K $(xxd -p key.bin | tr -d '\n') -iv $(xxd -p iv.bin | tr -d '\n')  

file flag.jpg  
flag.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16

cat flag.jpg  
����JFIFHHRecovered desktop image bytes...  
FLAG:utflag{m3m0ry_r3t41ns_wh4t_d1sk_l053s}  
Generated by Cold Workspace challenge.  
��[?2004h
```

`utflag{m3m0ry_r3t41ns_wh4t_d1sk_l053s}`

___

## Last Byte Standing

**Category**: forensics

**Description**: A midnight network capture from a remote office was marked “routine” and archived without review. Hours later, incident response flagged it for one subtle anomaly that nobody could explain. Find what was missed and recover the flag.

I get the challenge file:
```shell
file last-byte-standing.pcap  
last-byte-standing.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

When opening the pcap in wireshark I notice a lot of `dns` packets with zeros and ones in them like this:
```shell
sync-cache.nexthop-lab.net.....0............
sync-cache.nexthop-lab.net.....1............
sync-cache.nexthop-lab.net.....1............
sync-cache.nexthop-lab.net.....1............
sync-cache.nexthop-lab.net.....0............
sync-cache.nexthop-lab.net.....1............
sync-cache.nexthop-lab.net.....0............
sync-cache.nexthop-lab.net.....1............
sync-cache.nexthop-lab.net.....0............
sync-cache.nexthop-lab.net.....1............
sync-cache.nexthop-lab.net.....1............
sync-cache.nexthop-lab.net.....1............
sync-cache.nexthop-lab.net.....0............
```

Because of this I save the first 3 udp streams to a text file and run this python script:
```python
import re

with open("lines.txt", "r") as f:
    data = f.read()

# find all sequences of 0s and 1s, ignoring other characters
bits = re.findall(r"[01]", data)

# join all bits into one long string
bit_string = "".join(bits)

# ensure the length is divisible by 8 (trim extra bits if needed)
if len(bit_string) % 8 != 0:
    extra = len(bit_string) % 8
    print(f"Trimming {extra} extra bits to fit full bytes.")
    bit_string = bit_string[:-extra]

# decode each byte to ASCII
flag = ""
for i in range(0, len(bit_string), 8):
    byte = bit_string[i:i+8]
    flag += chr(int(byte, 2))

print("Flag:", flag)
```

Output:
```shell
python3 solve9.py  
Flag: utflag{d1g_t0_th3_l4st_byt3}UÃUÃUÃUÃUÃUÃ:V:V:V:V:V:V:VV
```

`utflag{d1g_t0_th3_l4st_byt3}`

___

## Silent Archive

**Category**: forensics

**Description**: Incident response recovered a damaged archive from an isolated workstation. The bundle split into two branches during transfer: one looks like duplicate camera captures, and the other is an absurdly deep archive chain. Follow both trails, reconstruct the hidden message, and recover the token.

I get the challenge files:
```shell
unzip freem4.zip  
Archive:  freem4.zip  
  inflating: File1.tar  
  inflating: File2.tar  
  inflating: README.txt
  
cat README.txt  
IR CASE 2026-0311  
Recovered bundle from an isolated workstation.  
Two archive branches survived transfer damage.  
Investigate both and recover the report token.

file *  
File1.tar:  POSIX tar archive  
File2.tar:  POSIX tar archive  
README.txt: ASCII text

tar -tf File1.tar  
cam_300.jpg  
cam_301.jpg

tar -tf File2.tar  
999.tar
```

`File1.tar` has 2 images (visually identical) and `File2.tar` is nested heavily. I find base64 encoded strings in the images:
```shell
tar -xf File1.tar

strings cam_300.jpg | tail  
8}Lf  
^8}Lf  
---BEGIN-TELEMETRY---  
TRACE_SEGMENT=A1f9z_Qp39_Xx2  
cache_blob=q9A1eR2u4T6o8P0s  
noise=R0xjODlhAQABAIAAAP///////ywAAAAAAQABAAACAkQBADs=  
dbg_ptr=7f3c2d1a9b887766554433221100ffaa  
cam_sig=ZXlKaGJHY2lPaUpJVXpJMU5pSjkuLi4  
AUTH_FRAGMENT_B64:QWx3YXlzX2NoZWNrX2JvdGhfaW1hZ2Vz  
---END-TELEMETRY---

strings cam_301.jpg | tail  
8}Lf  
^8}Lf  
---BEGIN-TELEMETRY---  
TRACE_SEGMENT=A1f9z_Qp39_Xx2  
cache_blob=q9A1eR2u4T6o8P0s  
noise=R0xjODlhAQABAIAAAP///////ywAAAAAAQABAAACAkQBADs=  
dbg_ptr=7f3c2d1a9b887766554433221100ffaa  
cam_sig=ZXlKaGJHY2lPaUpJVXpJMU5pSjkuLi4  
AUTH_FRAGMENT_B64:MHI0bmczX0FyQ2gxdjNfVDRiU3A0Y2Uh  
---END-TELEMETRY---

echo "MHI0bmczX0FyQ2gxdjNfVDRiU3A0Y2Uh" | base64 -d  
0r4ng3_ArCh1v3_T4bSp4ce!

echo "QWx3YXlzX2NoZWNrX2JvdGhfaW1hZ2Vz" | base64 -d  
Always_check_both_images
```

After making a mess trying to unpack the `.tar` files I run this script which cleanly unpacks all the files while deleting the old ones in the same directory: 
```python
import tarfile
import os

def extract_tars_until_done(base_path="."):
    while True:
        tars_found = False
        for f in os.listdir(base_path):
            if f.endswith(".tar"):
                tars_found = True
                full_path = os.path.join(base_path, f)
                print(f"Extracting {full_path}")
                try:
                    with tarfile.open(full_path) as tar:
                        for member in tar.getmembers():
                            # flatten extraction: ignore folder structure
                            member.name = os.path.basename(member.name)
                            tar.extract(member, path=base_path)
                    os.remove(full_path)
                except Exception as e:
                    print(f"Failed to extract {full_path}: {e}")
        if not tars_found:
            break  # stop when no more tars are left

if __name__ == "__main__":
    extract_tars_until_done(".")
```

Once that is done I get the file `Noo.txt` (actually a `.zip` file):
```shell
cat Noo.txt  
PK     r�k\p���u  
  
NotaFlag.txtUT ����iux  
�6=d6Q�o�������!J�-�M�����pLk�ŀO�)й]���  

file Noo.txt  
Noo.txt: Zip archive data, made by v3.0 UNIX, extract using at least v2.0, last modified Mar 11 2026 16:43:36, uncompressed size 522, method=deflate  

mv Noo.txt Yess.zip  

unzip Yess.zip  
Archive:  Yess.zip  
[Yess.zip] NotaFlag.txt password:  
password incorrect--reenter:
```
The new zip file is password protected. The password is `0r4ng3_ArCh1v3_T4bSp4ce!`.

Now I get two new files:
```shell
ls -la  
total 8  
drwxrwxr-x 1 user user  40 Mar 13 05:14 .  
drwxrwxr-x 1 user user 154 Mar 13 05:14 ..  
-rw-r--r-- 1 user user 522 Mar 11 17:43 NotaFlag.txt  
-rw-r--r-- 1 user user 106 Mar 11 17:43 notes.md

cat notes.md  
# Incident Notes  
Focus on camera deltas and archive chain integrity.  
No direct IOC included in this file.
```

`NotaFlag.txt` is a file full of whitespace (spaces and tabs). I run this python to map spaces to `0` and tabs to `1`, then decode from binary:
```python
#!/usr/bin/env python3

def decode_whitespace(filename):
    bits = ''
    with open(filename, 'r', encoding='utf-8') as f:
        for c in f.read():
            if c == ' ':
                bits += '0'
            elif c == '\t':
                bits += '1'

    # group bits into bytes
    chars = []
    for i in range(0, len(bits), 8):
        byte = bits[i:i+8]
        if len(byte) < 8:
            continue
        chars.append(chr(int(byte, 2)))
    
    message = ''.join(chars)
    return message

if __name__ == '__main__':
    filename = "Yess/NotaFlag.txt"
    flag = decode_whitespace(filename)
    print("Flag:")
    print(flag)
```

Output:
```shell
python3 whitespace.py  
Flag:  
utflag{d1ff_th3_tw1ns_unt4r_th3_st0rm_r34d_th3_wh1t3sp4c3}
```

`utflag{d1ff_th3_tw1ns_unt4r_th3_st0rm_r34d_th3_wh1t3sp4c3}`

___

## Landfall

**Category**: forensics

**Description**: You are a DFIR investigator in charge of collecting and analyzing information from a recent breach at UTCTF LLC. The higher ups have sent us a triage of the incident. Can you read the briefing and solve your part of the case? Triage Files: [https://cdn.utctf.live/Modified_KAPE_Triage_Files.zip](https://cdn.utctf.live/Modified_KAPE_Triage_Files.zip). By Jared (@jarpiano on discord)

I get the challenge files:
```shell
file checkpointA.zip  
checkpointA.zip: Zip archive data, made by v6.3, extract using at least v2.0, last modified Mar 12 2026 00:08:38, uncompressed size 36, method=store

cat briefing.txt  
Hello operator, in the .zip file is a triage of the desktop breached by the  
threat actors. It seems like they were able to physically login, so we think  
there's an insider threat amongst the employees.  
  
Checkpoint A: What command did the threat actor attempt to execute to  
obtain credentials for privilege escalation?  
  
Hint: The password to Checkpoint A is ONLY the encoded portion. The password  
is MD5 hash of this portion.

cat how-to-solve.txt  
To obtain the flag, you must obtain the password to the "Checkpoint A"  
zip file. The password is associated with the problem assigned in the briefing.
```

I also get a DFIR triage from KAPE:
```shell
cd /landfall/Modified_KAPE_Triage_Files/C
ls  
'$Boot'  '$Extend'  '$LogFile'  '$MFT'  '$Recycle.Bin'  '$Secure_$SDS' ProgramData   Users   Windows
```

I check the powershell history for `jon`:
```shell
ls Users  
Administrator  jon  Public  robb

cat Users/jon/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt  
cat (Get-PSReadLineOption).HistorySavePath  
powershell -nop -e dwBoAG8AYQBtAGkAIAAvAGEAbABsAA==  
powershell -nop -e YwBkACAARABvAHcAbgBsAG8AYQBkAHMA  
ls  
cd Downlaods  
cd DOwnloads  
ls  
powershell -e dwBnAGUAdAAgAGgAdAB0AHAAcwA6AC8ALwBnAGkAdABoAHUAYgAuAGMAbwBtAC8AZwBlAG4AdABpAGwAawBpAHcAaQAvAG0AaQBtAGkAawBhAHQAegAvAHIAZQBsAGUAYQBzAGUAcwAvAGQAbwB3AG4AbABvAGEAZAAvADIALgAyAC4AMAAtADIAMAAyADIAMAA5ADEAOQAvAG0AaQBtAGkAawBhAHQAegBfAHQAcgB1AG4AawAuAHoAaQBwAA==  
ls  
powershell -e dwBnAGUAdAAgAGgAdAB0AHAAcwA6AC8ALwBnAGkAdABoAHUAYgAuAGMAbwBtAC8AZwBlAG4AdABpAGwAawBpAHcAaQAvAG0AaQBtAGkAawBhAHQAegAvAHIAZQBsAGUAYQBzAGUAcwAvAGQAbwB3AG4AbABvAGEAZAAvADIALgAyAC4AMAAtADIAMAAyADIAMAA5ADEAOQAvAG0AaQBtAGkAawBhAHQAegBfAHQAcgB1AG4AawAuAHoAaQBwACAALQBPACAAbQBpAG0AaQBrAGEAdAB6AC4AegBpAHAA  
ls  
powershell -e -nop RQB4AHAAYQBuAGQALQBBAHIAYwBoAGkAdgBlACAAbQBpAG0AaQBrAGEAdAB6AC4AegBpAHAA  
powershell -nop -e RQB4AHAAYQBuAGQALQBBAHIAYwBoAGkAdgBlACAAbQBpAG0AaQBrAGEAdAB6AC4AegBpAHAA  
ls  
powershell -nop -e QwA6AFwAVQBzAGUAcgBzAFwAagBvAG4AXABEAG8AdwBuAGwAbwBhAGQAcwBcAG0AaQBtAGkAawBhAHQAegBcAHgANgA0AFwAbQBpAG0AaQBrAGEAdAB6AC4AZQB4AGUAIAAiAHAAcgBpAHYAaQBsAGUAZwBlADoAOgBkAGUAYgB1AGcAIgAgACIAcwBlAGsAdQByAGwAcwBhADoAOgBsAG8AZwBvAG4AcABhAHMAcwB3AG8AcgBkAHMAIgAgACIAZQB4AGkAdAAiAA==  
ls
```

The last command shows `mimikatz` being executed:
```shell
echo 'QwA6AFwAVQBzAGUAcgBzAFwAagBvAG4AXABEAG8AdwBuAGwAbwBhAGQAcwBcAG0AaQBtAGkAawBhAHQAegBcAHgANgA0AFwAbQBpAG0AaQBrAGEAdAB6AC4AZQB4AGUAIAAiAHAAcgBpAHYAaQBsAGUAZwBlADoAOgBkAGUAYgB1AGcAIgAgACIAcwBlAGsAdQByAGwAcwBhADoAOgBsAG8AZwBvAG4AcABhAHMAcwB3AG8AcgBkAHMAIgAgACIAZQB4AGkAdAAiAA==' | base64 -d  
C:\Users\jon\Downloads\mimikatz\x64\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
```

I take the `md5sum` of the encoded command:
```shell
printf 'QwA6AFwAVQBzAGUAcgBzAFwAagBvAG4AXABEAG8AdwBuAGwAbwBhAGQAcwBcAG0AaQBtAGkAawBhAHQAegBcAHgANgA0AFwAbQBpAG0AaQBrAGEAdAB6AC4AZQB4AGUAIAAiAHAAcgBpAHYAaQBsAGUAZwBlADoAOgBkAGUAYgB1AGcAIgAgACIAcwBlAGsAdQByAGwAcwBhADoAOgBsAG8AZwBvAG4AcABhAHMAcwB3AG8AcgBkAHMAIgAgACIAZQB4AGkAdAAiAA==' | md5sum  
00c8e4a884db2d90b47a4c64f3aec1a4  -
```

Unzip `checkpointA.zip` with the md5sum as the password and get the flag:
```shell
unzip checkpointA.zip  
Archive:  checkpointA.zip  
[checkpointA.zip] flag.txt password:  
extracting: flag.txt
```

`utflag{4774ck3r5_h4v3_m4d3_l4ndf4ll}`

___

## Watson

**Category**: forensics

**Description**: We need your help again agent. The threat actor was able to escalate privileges. We're in the process of containment and we want you to find a few things on the threat actor. The triage is the same as the one in "Landfall". Can you read the briefing and solve your part of the case? Triage Files: [https://cdn.utctf.live/Modified_KAPE_Triage_Files.zip](https://cdn.utctf.live/Modified_KAPE_Triage_Files.zip). By Jared (@jarpiano on discord)

I get the challenge files:
```shell
cat briefing.txt  
Welcome back agent. Please get us the following:  
  
Checkpoint A: The threat actor deleted a word document containing secret  
project information. Can you retrieve it and submit the name of the project?  
  
Checkpoint B: The threat actor installed a suspicious looking program that  
may or may not be benign. Retrieve the SHA1 Hash of the executable.  
  
Hint:  
- Checkpoint A's password is strictly uppercase  
- Checkpoint B's password is the SHA1 Hash(venv) 

cat how-to-solve.txt  
To obtain the flag, you must pass through all of the checkpoints. The  
checkpoints have all been encrypted with a password. The password for each  
checkpoint can be found by answering the problems in the Briefing. After completing each checkpoint, you will be given a part of the flag. Combine the  
checkpoint hashes along with hyphens to obtain the flag.  
  
Example: utflag{DEAD-BEEF}

file checkpointA.zip  
checkpointA.zip: Zip archive data, made by v6.3, extract using at least v2.0, last modified Mar 12 2026 00:19:00, uncompressed size 0, method=store  

file checkpointB.zip  
checkpointB.zip: Zip archive data, made by v6.3, extract using at least v2.0, last modified Mar 12 2026 00:20:58, uncompressed size 0, method=store
```

First I look for the deleted word document. The most obvious place is the `Recycle Bin`. I find the word document here:
```shell
file '$Recycle.Bin'/S-1-5-21-47857934-2514792372-2285641962-500/'$R07YGFU.docx'  
$Recycle.Bin/S-1-5-21-47857934-2514792372-2285641962-500/$R07YGFU.docx: Microsoft Word 2007+
```

I open the document and see this:
```shell
**TOP SECRET // OMEGA COMPARTMENT**  
**PROJECT HOOKEM**  
**Strategic Intelligence Memorandum**

**Originating Office:** Directorate of Advanced Strategic Technologies (DAST)  
**Document Control Number:** HKM-Ω-01  
**Clearance Requirement:** OMEGA-12  
**Distribution:** Strictly Limited – Unauthorized disclosure subject to prosecution under national security statutes.
```

The password for `checkpointA.zip` is `HOOKEM`:
```shell
unzip checkpointA.zip  
Archive:  checkpointA.zip  
   creating: Checkpoint A/  
[checkpointA.zip] Checkpoint A/A.txt password:  
 extracting: Checkpoint A/A.txt
```

Looking for the suspicious executable was much harder. I eventually saw two different calculator programs were installed:
```shell
c:\\users\\administrator\\appdata\\local\\calculator.exe  
c:\\users\\administrator\\appdata\\local\\ithqsu\\2ga2pl\\calc.exe
```
`calc.exe` is very suspicious.

I use `regipy-dump` to find the `FileID` for `calc.exe` which stores the `sha1` hash with a `0000` prefix:
```shell
regipy-dump Amcache.hve | grep -A 40 -i "calc.exe"  
"subkey_name": "calc.exe|bb6d3e29a64aae32",  
"path": "\\Root\\InventoryApplicationFile\\calc.exe|bb6d3e29a64aae32",  
"timestamp": "2026-03-12T03:39:48.913790+00:00",  
"values_count": 19,  
"values": [  
{  
"name": "ProgramId",  
"value": "0006d9ae4e1a50b84d5a2616319a5188b6b700000000",  
"value_type": "REG_SZ",  
"is_corrupted": false  
},  
{  
"name": "FileId",  
"value": "000067198a3ca72c49fb263f4a9749b4b79c50510155",  
"value_type": "REG_SZ",  
"is_corrupted": false  
},
```

The password for `checkpointB.zip` is `67198a3ca72c49fb263f4a9749b4b79c50510155`:
```shell
unzip checkpointB.zip  
Archive:  checkpointB.zip  
[checkpointB.zip] Checkpoint B/B.txt password:  
 extracting: Checkpoint B/B.txt
```

Put together the flag from the two text files:
```shell
cat 'Checkpoint A'/A.txt  
pr1v473_3y3

cat 'Checkpoint B'/B.txt  
m1551n6_l1nk
```

`utflag{pr1v473_3y3-m1551n6_l1nk}`
