---
layout: post
title:  "Patriot 2025 Forensics"
date:   2025-11-24 17:04:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /patriot-ctf-2025-forensics/
---
* TOC
{:toc}

## Word Sea Adventures

**Category**: Forensics

**Challenge author**: DJ Strigel

**Description**: Our experts found this weird word document in our file share. They couldn't find anything inside. Maybe you could look more closely and find the hidden prize within! No passphrases are needed for this challenge. The flag format will be `tctf{flag}` or `pctf{flag}`

I get a `.docx` file and unzip it:
```shell
file word_sea_adventures.docx  
word_sea_adventures.docx: Microsoft Word 2007+

unzip word_sea_adventures.docx  
Archive:  word_sea_adventures.docx  
 inflating: [Content_Types].xml        
 inflating: _rels/.rels                
 inflating: word/document.xml          
 inflating: word/_rels/document.xml.rels     
 inflating: word/_rels/footnotes.xml.rels     
 inflating: word/numbering.xml         
 inflating: word/styles.xml            
 inflating: word/footnotes.xml         
 inflating: word/comments.xml          
 inflating: docProps/core.xml          
 inflating: docProps/app.xml           
 inflating: docProps/custom.xml        
 inflating: word/theme/theme1.xml      
 inflating: word/settings.xml          
 inflating: word/webSettings.xml       
 inflating: word/fontTable.xml         
 inflating: crab.jpg                   
 inflating: sponge.jpg                 
 inflating: squid.jpg
```

I get 3 images which were not visible in the docx. I put each image into [https://www.aperisolve.com/](https://www.aperisolve.com/) which runs many good stego tools against it. It found all the text files with `steghide`.

`sponge.jpg` had a decoy:
```shell
cat decoy1.txt  
Spongebob is so chill! Why would he be hiding any flags?
```

`crab.jpg` had a decoy. 
```shell
cat decoy2.txt  
Mr Crabs heard that his cashier may be hiding some money and maybe a flag somewhere.
```

`squid.jpg` contained the flag:
```shell
cat flag.txt  
I guess you found handsome squidward... even his looks can't hide the flag.  
tctf{w0rD_f1le5_ar3_als0_z1p}
```

`tctf{w0rD_f1le5_ar3_als0_z1p}`

___

## Burger King

**Category**: Forensics

**Challenge author**: Matthew Johnson (CACI)

**Description**: I just got around to starting up my own forensics team. We're calling our selves the Burger King Crackers for our exceptional skills at getting information out of encrypted systems left by cyber criminals. Our first big case has finally arrived but there is just one file we can't seem to solve. The cyber criminals left over this encrypted zip file containing their plans. We were able to recover the first part of a file, but thats it. Can you recover their evil plans? The flag format is CACI{flag}

I get the challenge files. `partial.svg` and `BurgerKing.zip`:
```shell
file partial.svg  
partial.svg: SVG Scalable Vector Graphics image

file BurgerKing.zip  
BurgerKing.zip: Zip archive data, at least v2.0 to extract, compression method=store
```

`BurgerKing.zip` is an encrypted zip file. `partial.svg` is 39 bytes and contains this:
```shell
cat partial.svg  
<svg xmlns="http://www.w3.org/2000/svg"
```
This is the standard opening for any `svg` file.

I have 39 bytes of known plaintext and an encrypted zip. I know the tool `bkcrack` can decrypt the file with just this info. Download it:
```shell
git clone https://github.com/kimci86/bkcrack.git
```

Run `bkcrack` against the zip until it finds keys:
```shell
./bkcrack -C ../../../../BurgerKing.zip -c Hole.svg -p ../../../../partial.svg  
bkcrack 1.8.1 - 2025-10-25  
[22:57:22] Z reduction using 32 bytes of known plaintext  
100.0 % (32 / 32)  
[22:57:22] Attack on 240166 Z values at index 6  
Keys: b9540c69 069a11f9 fd31648f  
71.1 % (170687 / 240166)  
Found a solution. Stopping.  
You may resume the attack with the option: --continue-attack 170687  
[23:07:55] Keys  
b9540c69 069a11f9 fd31648f
```

Using these keys, copy all the decrypted contents to a new zip called `decrypted.zip`:
```shell
./bkcrack -C ../../../../BurgerKing.zip -k b9540c69 069a11f9 fd31648f -D ../../../../decrypted.zip  
bkcrack 1.8.1 - 2025-10-25  
[23:09:54] Writing decrypted archive ../../../../decrypted.zip  
100.0 % (5 / 5)
```

Unzip the full file:
```shell
unzip decrypted.zip  
Archive:  decrypted.zip  
extracting: Hole.svg                   
extracting: LockAndKey.svg             
extracting: Space.svg                  
extracting: SVGsSuck.svg               
extracting: Webs.svg
```

The flag is in `Hole.svg`:

![Alt text](/images/Hole.svg)

`CACI{Y0U_F0UND_M3!}`
