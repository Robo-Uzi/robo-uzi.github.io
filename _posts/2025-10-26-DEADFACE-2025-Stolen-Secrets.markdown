---
layout: post
title:  "Stolen Secrets"
date:   2025-10-26 21:34:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /stolen-secrets/
---
* TOC
{:toc}

## The Source

**Category:** forensics

**Author:** @syyntax

**Description:** A department at TGRI was getting lazy with sensitive files. Rather than use the company’s approved filesharing application, a member used an application called MyShare and forgot to configure it to run HTTPS.

DEADFACE found the site and were able to compromise it and steal several sensitive files. Based on DEADFACE’s past behavior, let’s assume that flags discovered may be used as DEADFACE passwords.

First thing’s first: what is the IP address of the DEADFACE attacker? Submit the flag as `deadface{X.X.X.X}`.

I get these files:
```shell
file *  
access.log:          ASCII text  
cap-1753106207.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 262144)  
error.log:           ASCII text, with very long lines (357)  
error.php.log:       ASCII text
```

You can find the IP in `access.log` where you see the attacker ran nmap, then hydra. `error.php.log` also contains the IP with a lot of failed login attempts.

`deadface{134.199.202.160}`

___

## Calling Card

**Category:** forensics

**Author:** @syyntax

**Description:** DEADFACE has left a taunting message in their initial probing of the MyShare web application at `http://files.techglobalresearch.com`. Their attack involves a simple HTTP request with a hidden flag. Can you uncover their calling card? Submit the flag as `deadface{text}`. Use the ZIP file from **Stolen Secrets: The Source**. 

I open the pcap and see this traffic:
```http
POST / HTTP/1.1
Host: files.techglobalresearch.com
User-Agent: curl/8.14.1
Accept: */*
FLAG: deadface{l3ts_get_Th3s3_fiL3$}
Content-Length: 437
Content-Type: application/x-www-form-urlencoded

{message: 'SeKAmXZlIGdhaW5lZCBmdWxsIGFjY2VzcyB0byB5b3VyIG5ldHdvcmsuIEV2ZXJ5IGZpbGUsIGV2ZXJ5IGNyZWRlbnRpYWwsIGV2ZXJ5IHN5c3RlbSDigJQgdW5kZXIgbXkgY29udHJvbC4gWW91IGRpZG7igJl0IG5vdGljZSBiZWNhdXNlIEkgZGlkbuKAmXQgd2FudCB5b3UgdG8uCgpUaGlzIHdhc27igJl0IGx1Y2suIEl0IHdhcyBwcmVjaXNpb24uIFlvdXIgZGVmZW5zZXMgd2VyZSBpbmFkZXF1YXRlLCBhbmQgSeKAmXZlIHByb3ZlbiBpdC4KClRoaXMgYXR0YWNrIGlzIGJyb3VnaHQgdG8geW91IGJ5IG1pcnZlYWwuIFRoYW5rcyBmb3IgdGhlIHNlY3JldHMh'}HTTP/1.1 302 Found
Server: nginx/1.25.5
Date: Mon, 21 Jul 2025 13:57:01 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Powered-By: PHP/8.2.29
Set-Cookie: MYSHARESESSID=bdc478f9433be6fcff42ccdcb6b3c383; path=/; domain=files.techglobalresearch.com; HttpOnly; SameSite=Lax
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Location: login.php
```
The flag is in the `FLAG` header.

`deadface{l3ts_get_Th3s3_fiL3$}`

___

## Versions

**Category:** forensics

**Author:** @syyntax

**Description:** Based on the network traffic, what web server software and version is MyShare running? Submit the flag as `deadface{software_version}`. Example: `deadface{apache_1.2.3}` Use the ZIP file from **Stolen Secrets: The Source**. 

Found in the `Server:` header from the request I looked at in the `Calling Card` challenge. 

`deadface{nginx_1.25.5}`

___

## Compromised

**Category:** forensics

**Author:** @syyntax

**Description:** Which user did DEADFACE compromise and what was that user’s password? Submit the flag as `deadface{username_password}`. Example: `deadface{user01_P@$$w0rd!}`. se the ZIP file from **Stolen Secrets: The Source**. 

In `error.php.log` I see repeated failed login attempts until these lines:
```shell
[21-Jul-2025 14:10:13 UTC] [LOGIN_SUCCESS] User:"bsampsel" - IP: 134.199.202.160
[21-Jul-2025 14:11:53 UTC] [LOGIN_SUCCESS] User:"dtenuto" - IP: 134.199.202.160
```
The user `bsampsel` was compromised. They then created an admin account as `dtenuto`.

I opened the pcap and found the login:
```http
POST /login.php HTTP/1.1
Host: files.techglobalresearch.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 42
Origin: http://files.techglobalresearch.com
Connection: keep-alive
Referer: http://files.techglobalresearch.com/login.php
Cookie: MYSHARESESSID=c2c57253add980f9135f18147e602273
Upgrade-Insecure-Requests: 1
Priority: u=0, i

username=bsampsel&password=Sparkles2025%21
```

`deadface{bsampsel_Sparkles2025!}`

___

## A Wild User Suddenly Appeared!

**Category:** forensics

**Author:** @syyntax

**Description:** Which user did DEADFACE create for persistence? What was that user’s first name and password? Submit the flag as `deadface{firstname_password}`. Example: `deadface{John_P@$$w0rd!}`. 

As I saw in the challenge `Compromised` DEADFACE first compromised the account `bsampsel` then made the account with user `dtenuto`. 

I find this request:
```http
POST /admin.php?page=create_user HTTP/1.1
Host: files.techglobalresearch.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 156
Origin: http://files.techglobalresearch.com
Connection: keep-alive
Referer: http://files.techglobalresearch.com/admin.php?page=create_user
Cookie: MYSHARESESSID=c2c57253add980f9135f18147e602273
Upgrade-Insecure-Requests: 1
Priority: u=0, i

first_name=Dorla&last_name=Tenuto&email=dtenuto%40techglobalresearch.adsfglskdafhj.com&username=dtenuto&password=SuP3RS3cr3tD34DF4C3%23&is_admin=1&add_user=
```

`deadface{Dorla_SuP3RS3cr3tD34DF4C3#}`

___

## Tell No One

**Category:** forensics traffic analysis

**Author:** @syyntax

**Description:** DEADFACE stole a sensitive document from the MyShare application! Find the document and present the flag shown within. Submit the flag as `deadface{flag}`. NOTE: `deadface{h1dd3n_c0mm$!!}` is NOT the flag. It's a leftover remnant that was supposed to have been removed. Use the ZIP file from **Stolen Secrets: The Source**. 

I opened the pcap file `cap-1753106207.pcap` in wireshark and export the http objects. I get two useful looking files. A `zip` and a `gzip`:
```shell
file 'index.php%3fdownload_zip=1&folder=2' && file sender.tar.gz 
 
index.php%3fdownload_zip=1&folder=2: Zip archive data, at least v2.0 to extract, compression method=deflate  

sender.tar.gz: gzip compressed data, from Unix, original size modulo 2^32 10240
```
Inside `sender.tar.gz` is the python script `sender.py`.

Inside the `zip` file `index.php%3fdownload_zip=1&folder=2` I find 3 files:
```shell
ile *  
2025-06-20_phelps.helena.txt: Unicode text, UTF-8 text, with very long lines (704), with CRLF line terminators  
ns-9.png:       PNG image data, 1024 x 1024, 8-bit/color RGB, non-interlaced  
tgri-rnd-2025-0719.pdf:      PDF document, version 1.6 (zip deflate encoded)
```
A text file, a png, and a pdf. Inside the pdf it is labeled "CONFIDENTIAL - INTERNAL USE ONLY" and the flag is at the bottom.

`deadface{w1R3_y0uR_Br41N}`

___

## Patchwork

**Category:** forensics traffic analysis

**Author:** @syyntax

**Description:** UPDATE: If you're looking for a password, try applying sensitive information found in other challenges (i.e., passwords, keys, flags, etc). After DEADFACE compromised MyShare, they uploaded 2 files. One of these files is named in such a way that implies it’s meant to be an antivirus patch. Extract the flag from this file. Submit the flag as `deadface{flag}`. NOTE: `deadface{h1dd3n_c0mm$!!}` is NOT the flag. It's a leftover remnant that was supposed to have been removed. Use the ZIP file from **Stolen Secrets: The Source**.

I start by following the tcp stream and copying the entire thing as raw. I run `xxd -r -p zip-please > zip-please.bin` and after trying to unzip I see this:
```shell
unzip zip-please.bin  
Archive:  zip-please.bin  
warning [zip-please.bin]:  1869910 extra bytes at beginning or within zipfile  
 (attempting to process anyway)  
[zip-please.bin] av_lin_update password:
```

I dont know the password but I want to crack it with `john` so I carve the file to just the zip:
```shell
dd if=zip-please.bin of=zip_extracted.zip bs=1 skip=$((0x1C8856))

file zip_extracted.zip  
zip_extracted.zip: Zip archive data, at least v2.0 to extract, compression method=deflate
```

Considering the hint in the challenge description (`If you're looking for a password, try applying sensitive information found in other challenges`) 

I make a custom wordlist and generate a hash with `zip2john zip_extracted.zip > zip.hash`. 

Then crack the password and extract the files:
```shell
john --wordlist=words.txt zip.hash  
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
w1R3_y0uR_Br41N  (zip_extracted.zip)        
1g 0:00:00:00 DONE (2025-10-26 16:31) 2.173g/s 497.8p/s 497.8c/s 497.8C/s av_lin_update  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.

unzip zip_extracted.zip  
Archive:  zip_extracted.zip  
[zip_extracted.zip] av_lin_update password:    
 inflating: av_lin_update              
 inflating: av_win_update.exe
```

I get binaries for windows and linux:
```shell
file av_lin_update  
av_lin_update: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=eb586756c39e4efd29c393a8b324b3af04fd9a90, stripped

file av_win_update.exe  
av_win_update.exe: PE32+ executable (console) x86-64, for MS Windows, 7 sections
```

I grep for `dead` or `flag` and see python mentioned:
```shell
strings av_lin_update | grep flag  
pyi-python-flag
```

Extract the `.pyc` files with `pyinstxtractor.py`:
```shell
python3 pyinstxtractor.py av_lin_update  
[+] Processing av_lin_update  
[+] Pyinstaller version: 2.1+  
[+] Python version: 3.13  
[+] Length of package: 8039403 bytes  
[+] Found 30 files in CArchive  
[+] Beginning extraction...please standby  
[+] Possible entry point: pyiboot01_bootstrap.pyc  
[+] Possible entry point: pyi_rth_inspect.pyc  
[+] Possible entry point: av_lin_update.pyc  
[!] Warning: This script is running in a different Python version than the one used to build the executable.  
[!] Please run this script in Python 3.13 to prevent extraction errors during unmarshalling  
[!] Skipping pyz extraction  
[+] Successfully extracted pyinstaller archive: av_lin_update  
  
You can now use a python decompiler on the pyc files within the extracted directory
```

`uncompyle6` didnt like it:
```shell
uncompyle6 -o . av_lin_update.pyc  
  
Unsupported Python version, 3.13.0, for decompilation  
  
  
# Unsupported bytecode in file av_lin_update.pyc  
# Unsupported Python version, 3.13.0, for decompilation  
av_lin_update.pyc -- decompiled 0 files: 0 okay, 1 failed
```

I go to [https://www.pylingual.io/](https://www.pylingual.io/) and drop `av_lin_update.pyc` into the page. It decompiles to a python script which I run to get the flag:
```python
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: av_lin_update.py
# Bytecode version: 3.13.0rc3 (3571)
# Source timestamp: 1970-01-01 00:00:00 UTC (0)

hex_string = '\\x67\\x66\\x62\\x67\\x65\\x62\\x60\\x66\\x78\\x6e\\x37\\x6f\\x6a\\x60\\x6a\\x33\\x76\\x50\\x5c\\x76\\x53\\x6f\\x33\\x37\\x47\\x7e'
byte_data = bytes.fromhex(hex_string.replace('\\x', ''))
key = 3
original_bytes = bytes((b ^ key for b in byte_data))
original_message = original_bytes.decode('utf-8')
print(f'The flag is: {original_message}')
```

`deadface{m4lici0uS_uPl04D}`

___
