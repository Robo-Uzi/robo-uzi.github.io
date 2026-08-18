---
layout: post
title:  "Misc Challenges"
date:   2026-08-17 20:01:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /0xV01D-ctf-2026-misc/
---
* TOC
{:toc}

## A Simple Spectrum

**Category**: misc

**Author**: 0xkhaled

**Description**: We intercepted this audio file from a deep-space relay station. It sounds like static to our analysts — but our algorithms detected anomalous frequency patterns. Maybe the flag isn't something you hear... it's something you see. Flagformat : 0xV0ID{}

I get the challenge file `spectrum.wav`:
```shell
file spectrum.wav  
spectrum.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 44100 Hz
```

I opened the file in audacity and looked at the spectrogram view:

![Alt text](/images/voidspectro64786.png)

`0xV0ID{sp3ctr0gr4m_s3cr3ts}`

___

## Transmission

**Category**: misc

**Author**: jinx69
**Description**: Our team intercepted a strange transmission last night. Command wants to know what’s really in that signal.

I get the challenge file `Transmission.zip` which is password protected:
```shell
unzip Transmission.zip  
Archive:  Transmission.zip  
[Transmission.zip] unknown.unknown password:
```

I tried the passwords `password` and `infected` but they didnt work so I cracked the password:
```shell
zip2john Transmission.zip > zip.hash  
ver 2.0 efh 5455 efh 7875 Transmission.zip/unknown.unknown PKZIP Encr: TS_chk, cmplen=240231, decmplen=529244, crc=DFD0C27E ts=52A4 cs=52a4 type=8
```

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash  
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
whatever1        (Transmission.zip/unknown.unknown)  
1g 0:00:00:00 DONE (2026-08-15 17:59) 50.00g/s 204800p/s 204800c/s 204800C/s 123456..bigman  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

```shell
unzip Transmission.zip  
Archive:  Transmission.zip  
[Transmission.zip] unknown.unknown password:  
  inflating: unknown.unknown
```

I get the file `unknown.unknown`:
```shell
file unknown.unknown  
unknown.unknown: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 44100 Hz
```

I put the file into audacity and looked at the spectrogram view:

![Alt text](/images/voidspectro2.png)

`0xV01D{h1dd3n_1n_th3_sp3ctr0}`

___

## Between The Lines

**Category**: misc

**Author**: 0xkhaled

**Description**: An old poet left this manuscript on a dead drop server. Our forensic team says it looks like a normal poem. But something feels off about the spacing. Read carefully — not the words, but the gaps between them. Flagformt : 0xV0ID

I get the challenge file `poem.txt`:
```shell
cat poem.txt
In the silence of the void, a signal waits to speak,  		     	
Through darkness forged in binary, the answers that we seek.			    	 	
Each letter holds a secret wrapped in layers none can see, 		   		  
The truth is always present if you know where it could be.   	  	  	
A hacker reads the whitespace as a language of its own, 	   	   	
The tabs and spaces whisper of the secrets never shown.			 		 			
Between the lines of poetry the hidden data flows, 			 		 	 
And only those who look beyond the surface truly knows.    		   	
So strip away the visible and peer into the gaps, 			 	    
The flag was always there for you between the word-wraps.		  		 			
Decode the silent language that the spaces dare to tell,  		 			  
And you shall find the answer hiding there so very well.    		 	  
The void between the letters is not empty after all, 		   		  
For in the whitespace universe the secrets rise and fall.		  		 	 	
Now look beyond the stanzas where the silence makes its stand,				 		 	 
And read the binary language with a careful, patient hand.    		   	
Each line concludes with meaning that your eye cannot perceive, 		  	    
Unless you strip the whitespace out and simply just believe.		  		 			
The flag awaits the patient one who sees what is not there,  		 	 			
A void within the visible, a ghost beyond compare.		  		 	  
Remember that in hacking truth is hidden in plain sight, 		 		   	
The answer is the whitespace at the end of every night.	 		   	 	
So run your script and parse each line and gather every bit,				 			 	
And when the bytes assemble you will know that this is it.   			  	 
The journey through the void is long but always leads you home, 			 	 	 	
The flag is in the silence where the lone researcher roams.		 	   		 
Now calculate the binary and convert it ASCII clear,	    					
And shout the flag aloud because the finish line is here. 	        
```

There is trailing whitespace in the file which maps to binary. Each `space` gets mapped to `0` and each `tab` gets mapped to `1`:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> with open('poem.txt', 'r') as f:  
...     binary_string = ""  
...     for line in f:  
...         # strip newlines. keep trailing spaces and tabs  
...         raw = line.rstrip('\n\r')  
...  
...         # remove text  
...         visible_text = raw.rstrip(' \t')  
...         trailing_whitespace = raw[len(visible_text):]  
...  
...         bits = trailing_whitespace.replace(' ', '0').replace('\t', '1')  
...         binary_string += bits  
...  
... flag = "".join([chr(int(binary_string[i:i+8], 2)) for i in range(0, len(binary_string), 8)])  
...  
... print(f"Binary: {binary_string}")  
... print(f"Flag: {flag}")  
...  
Binary: 0011000001111000010101100011000001001001010001000111101101110111011010000011000101110100001100110111001101110000001101000110001100110011010111110110100000110001011001000011001101110011010111110011010001101100011011000101111101110100011100100111010101110100011010000111110100000000  
Flag: 0xV0ID{wh1t3sp4c3_h1d3s_4ll_truth}
```

`0xV0ID{wh1t3sp4c3_h1d3s_4ll_truth}`

___

## Quiet Note

**Category**: misc

**Author**: 0xkhaled

**Description**: This challenge allows only **1 flag attempt**. Submit only when you are sure. A small artifact is provided. Inspect it carefully and recover the single valid flag. Flag format : `0xV01D{......}`. Submit the complete flag exactly as shown by the format, including the prefix `0xV01D` and the braces.

I get the challenge file:
```shell
cat letter.txt  
0  The archive team left a calm sentence here.  
x  The archive team left a calm sentence here.  
V  The archive team left a calm sentence here.  
0  The archive team left a calm sentence here.  
1  The archive team left a calm sentence here.  
D  The archive team left a calm sentence here.  
{  The archive team left a calm sentence here.  
F  The archive team left a calm sentence here.  
I  The archive team left a calm sentence here.  
R  The archive team left a calm sentence here.  
S  The archive team left a calm sentence here.  
T  The archive team left a calm sentence here.  
_  The archive team left a calm sentence here.  
L  The archive team left a calm sentence here.  
E  The archive team left a calm sentence here.  
T  The archive team left a calm sentence here.  
T  The archive team left a calm sentence here.  
E  The archive team left a calm sentence here.  
R  The archive team left a calm sentence here.  
S  The archive team left a calm sentence here.  
_  The archive team left a calm sentence here.  
N  The archive team left a calm sentence here.  
E  The archive team left a calm sentence here.  
V  The archive team left a calm sentence here.  
E  The archive team left a calm sentence here.  
R  The archive team left a calm sentence here.  
_  The archive team left a calm sentence here.  
L  The archive team left a calm sentence here.  
I  The archive team left a calm sentence here.  
E  The archive team left a calm sentence here.  
}  The archive team left a calm sentence here.
```

```shell
cut -c 1 letter.txt | tr -d '\n'  
0xV01D{FIRST_LETTERS_NEVER_LIE}
```

`0xV01D{FIRST_LETTERS_NEVER_LIE}`

___

## Acrostic

**Category**: misc

**Description**: This challenge allows only **1 flag attempt**. Submit only when you are sure. A mysterious message was intercepted from the network. Something is hidden in plain sight. Read carefully — the first letter of each line reveals the secret. Flag format : `0xV0ID{......}`

I get the challenge file:
```shell
cat message.txt  
Forgotten echoes drift through the network at midnight.  
Invisibly, packets cross the wire unseen.  
Routing tables shift and reshape the data paths.  
Silence fills the void between each transmission.  
Time stamps record every byte that passes.  
Signals propagate at the speed of light.  
Topology defines how nodes find each other.  
Encryption wraps the payload in darkness.  
Persistence is the key to every challenge.
```

Simply take the first letter of each line:
```shell
cut -c 1 message.txt | tr -d '\n'  
FIRSTSTEP
```

`0xV0ID{FIRSTSTEP}`

___

## Single Byte

**Category**: misc

**Description**: This challenge allows only **2 flag attempt**. Submit only when you are sure. A binary blob was extracted from RAM. It's not plaintext — but it's close. Single-byte operations are often reversible. Try all 256 possibilities. Flag format : `0xV0ID{......}`

I got the challenge file `secret.bin`:
```shell
file secret.bin  
secret.bin: data  
```

```shell
xxd secret.bin  
00000000: 723a 1472 0b06 393a 7230 1d29 713b 1d24  r:.r..9:r0.)q;.$  
00000010: 7237 2c26 3f                             r7,&?
```

I put it into cyberchef and did an XOR brute force. It found the xor key `0x42`.

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=XOR(%7B'option':'Hex','string':'0x42'%7D,'Standard',false)&input=cjoUcgsGOTpyMB0pcTsdJHI3LCY/&oeol=VT)

`0xV0ID{x0r_k3y_f0und}`

___

## Time Machine

**Category**: misc

**Description**: An old container image has been recovered from an unknown source. The contents may reveal more than expected. Explore carefully and uncover the hidden secret.
```bash
docker pull jinx69/timemachine:latest
```

I start with pulling the image:
```shell 
docker pull docker.io/jinx69/timemachine:latest  
Trying to pull docker.io/jinx69/timemachine:latest...  
Getting image source signatures  
Copying blob 1ffee909ebce done   |  
Copying blob 966c395d29cb done   |  
Copying blob 71eff466177c done   |  
Copying blob 8406f059c313 done   |  
Copying blob 5d78a4c92bc4 done   |  
Copying blob 0e0d109ace69 done   |  
Copying config 0e9a77b492 done   |  
Writing manifest to image destination  
0e9a77b492cc2be4f670591c59a07bb6a02dd57e7c6bf8b421f78d404c519e86
```

I looked at the history. The was one command containing a password for a different user:
```shell
podman history --no-trunc jinx69/timemachine:latest  

... etc ...

/bin/sh -c useradd -m void && echo "Setting default credentials..." && echo "void:trave1er" | chpasswd

... etc ...
```

Run the image, become the user `void` with the password `trave1er`, and get the flag:
```shell
docker run -it --rm jinx69/timemachine:latest /bin/bash
player@90145a9ded20:/$ id  
uid=1002(player) gid=1002(player) groups=1002(player)
player@90145a9ded20:/$ su void  
Password:  
$ whoami  
void  
$ id  
uid=1001(void) gid=1001(void) groups=1001(void)  
$ cat opt/flag.sh  
echo "0xVO1D{h1st0ry_n3v3r_li35}"
```

`0xVO1D{h1st0ry_n3v3r_li35}`

___

## VoidNotes.apk

**Category**: misc

**Author**: 0xkhaled

**Description**: Our threat intel team recovered this note-taking app from a compromised employee device. The developer insists the notes are "encrypted" and completely safe from prying eyes.

We're not so sure. Can you recover the secret developer note? Flag format: 0xV0ID{...}

I get the challenge file:
```shell
file VoidNotes.apk  
VoidNotes.apk: Android package (APK), with AndroidManifest.xml, with APK Signing Block
```

Unzip it:
```shell
unzip VoidNotes.apk  
Archive:  VoidNotes.apk  
  inflating: AndroidManifest.xml  
 extracting: resources.arsc  
  inflating: classes.dex  
 extracting: assets/secret_note.bin  
  inflating: META-INF/DEBUG.SF  
  inflating: META-INF/DEBUG.RSA  
  inflating: META-INF/MANIFEST.MF
```

```shell
xxd assets/secret_note.bin  
00000000: 652d 0365 1c11 2e3d 6127 3136 6531 6631  e-.e...=a'16e1f1  
00000010: 0a61 2626 6621 260a 6127 660a 2127 6126  .a&&f!&.a'f.!'a&  
00000020: 3d28                                     =(
```

You need to XOR `secret_note.bin` with the key `0x55`. [cyberchef link](https://gchq.github.io/CyberChef/#recipe=XOR(%7B'option':'Hex','string':'0x55'%7D,'Standard',false)&input=ZS0DZRwRLj1hJzE2ZTFmMQphJiZmISYKYSdmCiEnYSY9KA)

`0xV0ID{h4rdc0d3d_4ss3ts_4r3_tr4sh}`

___

## Signal Loss

**Category**: crypto

**Author**: Jinx

**Description**: A short radio transmission was intercepted before the signal abruptly disappeared. Analysts believe the message has been encoded in multiple layers to conceal its true contents. Recover the original message. Flag Format: 0xV0ID{...}

I get the challenge file:
```shell
file secret.wav  
secret.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 8 bit, mono 8000 Hz
```

I opened the file and it sounded like morse code. I put the file into [https://morsecode.world/international/decoder/audio-decoder-adaptive.html](https://morsecode.world/international/decoder/audio-decoder-adaptive.html) and it output message:
```shell
3 0 7 8 5 6 3 0 4 9 4 4 7 B 7 3 3 1 6 7 6 3 V 6 C 5 F 6 4 3 3 6 3 3 0 6 4 3 3 6 4 5 F 6 C 3 4 7 9 3 3 7 2 5 F 6 2 7 9 5 F 6 C 3 4 7 9 3 3 7 2 7 D 0 A 0 A 0 E
```

I decoded it:
```shell
echo '3078563049447B73316763V6C5F643363306433645F6C347933725F62795F6C347933727D' | xxd -r -p  
0xV0ID{s1gcl_d3c0d3d_l4y3r_by_l4y3r}
```
It got the morse code slightly wrong but I was able to guess `n4l` instead of `cl`!

`0xV0ID{s1gn4l_d3c0d3d_l4y3r_by_l4y3r}`

___

## FirstStep

**Category**: crypto

**Description**: Everyone walks through the same door to get here. The question is whether you know how to open it. Welcome. Flag format: 0xV01D{...}

I get the challenge file:
```shell
cat cipher.txt  
723a147273063915710e01720f711d16721d0116043f
```

In cyberchef do `from hex` then `XOR` with key `0x42`.

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')XOR(%7B'option':'Hex','string':'0x42'%7D,'Standard',false)&input=NzIzYTE0NzI3MzA2MzkxNTcxMGUwMTcyMGY3MTFkMTY3MjFkMDExNjA0M2Y)

`0xV01D{W3LC0M3_T0_CTF}`
