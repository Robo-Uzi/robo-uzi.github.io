---
layout: post
title:  "Forensics Challenges"
date:   2026-02-14 16:20:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /0xfun-2026-forensics-challenges/
---
* TOC
{:toc}

## TLSB

**Category**: forensics

**Author**: x03e

**Description**: You might know about Least Significant Bit (LSB) steganography, but have you ever heard of Third Least Significant Bit (TLSB) steganography? (Probably not, I invented it for this challenge).

I get the challenge file:
```shell
file TLSB  
TLSB: PC bitmap, Windows 3.x format, 16 x 16 x 24, resolution 16 x 16 px/m, cbSize 822, bits offset 54
```

I need to extract the data from the 3rd least significant bit. Theres 8 bits in a byte:
```
bit index:  7 6 5 4 3 2 1 0
            ^             ^
           MSB           LSB
```
So they hiding the data in bit index 2. 

In the python instead of doing normal LSB (`(byte >> 0) & 1`) I need to do `(byte >> 2) & 1`.

python script to extract 3rd least significant bit:
```python
#!/usr/bin/env python3
import sys

def extract_tlsb(filename):
    with open(filename, "rb") as f:
        data = f.read()

    # BMP pixel data offset (bytes 10–14, little endian)
    pixel_offset = int.from_bytes(data[10:14], "little")
    pixel_data = data[pixel_offset:]

    bits = []

    for byte in pixel_data:
        tlsb = (byte >> 2) & 1
        bits.append(str(tlsb))

    # Convert bit stream to bytes
    bitstream = "".join(bits)
    output = []

    for i in range(0, len(bitstream), 8):
        byte = bitstream[i:i+8]
        if len(byte) == 8:
            output.append(chr(int(byte, 2)))

    return "".join(output)

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <bmp_file>")
        sys.exit(1)

    result = extract_tlsb(sys.argv[1])
    print(result)
```

Run the script and decode the base64 we get:
```shell
python3 solve1.py TLSB  
Hope you had fun :). The Flag is: `MHhmdW57VGg0dDVfbjB0X0wzNDV0X1MxZ24xZjFjNG50X2IxdF81dDNnfQ=='  

echo "MHhmdW57VGg0dDVfbjB0X0wzNDV0X1MxZ24xZjFjNG50X2IxdF81dDNnfQ==" | base64 -d  
0xfun{Th4t5_n0t_L345t_S1gn1f1c4nt_b1t_5t3g}
```

`0xfun{Th4t5_n0t_L345t_S1gn1f1c4nt_b1t_5t3g}`

___

## Nothing Expected

**Category**: forensics

**Author**: x03e

**Description**: Here’s a small drawing I put together. There isn’t really anything in it, as you can tell.

I get the challenge file:

![Alt text](/images/file.png)

`strings` mentions `excalidraw`:
```shell
strings -n 30 file.png  
tEXtapplication/vnd.excalidraw+json  
{"version":"1","encoding":"bstring","compressed":true,"encoded":"x
```

I go to [https://excalidraw.com/](https://excalidraw.com/) and open the file. Then I see the flag:

![Alt text](/images/excalidraw.png)

`0xfun{th3_sw0rd_0f_k1ng_4rthur}`

___

## PrintedParts

**Category**: forensics

**Author**: x03e

**Description**: A friend of mine 3D printed something interesting.

I get the challenge file `3D.gcode` which is a large file for 3D printing:
```shell
cat 3D.gcode | wc -l  
1164616  

file 3D.gcode  
3D.gcode: ASCII text, with CRLF line terminators
```

I find some sort of `flag.stl` file inside the gcode:
```shell
cat 3D.gcode | grep flag | head  
;MESH:flag.stl  
;MESH:flag.stl  
;MESH:flag.stl  
;MESH:flag.stl  
;MESH:flag.stl  
;MESH:flag.stl  
;MESH:flag.stl  
;MESH:flag.stl  
;MESH:flag.stl  
;MESH:flag.stl
```

I rendered `3D.gcode` in the site [https://gcode.ws/](https://gcode.ws/) and I can see the beginning of the flag but the rest of the flag is hidden down inside the print:

![Alt text](/images/3dprint1.png)

I had AI write a python script to extract out the code related to `flag.stl` but write it to a file called `flag.gcode`:
```python
with open('3D.gcode', 'r') as infile, open('flag.gcode', 'w') as outfile:
    wrote_header = False
    in_flag = False
    for line in infile:
        stripped = line.strip()
        # Copy the full header once
        if not wrote_header:
            outfile.write(line)
            if stripped == ';END_OF_HEADER':
                wrote_header = True
            continue
        if stripped == ';MESH:flag.stl':
            in_flag = True
            outfile.write(line)
            continue
        if in_flag and stripped.startswith(';MESH:') and 'flag.stl' not in stripped:
            in_flag = False
            continue
        if in_flag:
            outfile.write(line)
    outfile.write('\nM140 S0\nM104 S0\nM107\n;End of Gcode\n')
```

When rendering the file `flag.gcode` in 3D on the site [https://gcode.ws/](https://gcode.ws/) I can see the flag!

![Alt text](/images/3dprint2.png)

I tried doing this by going through the layers in 2D but the flag seems to be at multiple different angles in the print so it was impossible to make out otherwise. 

`0xfun{this_monkey_has_a_flag}`

___

## kd>

**Category**: forensics

**Author**: N!L

**Description**: something crashed. something was left behind.

I get the challenge files:
```shell
ls  
config.dat  crypter.dmp  events.xml  transcript.enc  

file *  
config.dat:     data  
crypter.dmp:    Mini DuMP crash report, 18 streams, Wed Feb 11 14:05:53 2026, 0x61826 type  
events.xml:     XML 1.0 document, ASCII text, with CRLF line terminators  
transcript.enc: data
```

I was thinking I needed `WinDbg` or `volatility` but who needs it when you have `strings` and `grep`: 
```shell
strings crypter.dmp | grep "0xfun"  
0xfun{wh0_n33ds_sl33p_wh3n_y0u_h4v3_cr4sh_dumps}
```

`0xfun{wh0_n33ds_sl33p_wh3n_y0u_h4v3_cr4sh_dumps}`

___

## Tesla

**Category**: forensics

**Author**: x03e

**Description**: Flipper Zero, often referred to as a hacking device, is designed to capture frequencies and execute commands. It's considered a risky tool to have, as it is illegal in some countries.

I get the challenge file:
```shell
file Tesla.sub  
Tesla.sub: ASCII text, with very long lines (26352), with CRLF line terminators

cat Tesla.sub  
Filetype: Bad Usb 0xfun  
Version: 1  
Frequency: 433920000  
Preset: FuriHalSubGhz  
Protocol: RAW  
RAW_Data: 11111111 11111110 00100110 01000000 01100011 01101100 01110011 00100110 01000000 01110011 01100101 01110100 00100000 00100010 01001001 01101100 11000011 01100011 00111101 01110000 01100101 01110011 01100010 01001101 01010101 01010001 01101100 00110111 00110011 01101111 01010111 01101110 01110001 01000100 00111001 01110010 01000001 01110110 01000110 01010010 01001011 01011010 01100001 01100110 00110000 01101000 01001111 00110101 01000000 01100100 01000010 01001110 00110100 01110101 01010011 01111010 01000011 01110100 01000111 01101010 01000101 00100000 01011001 01111000 01001001 01010100 01110111 01011000 01101001 01010110 01101101 00110001 01001010 01100011 01100111 01111001 00110010 00110110 01001100 01101011 01001000 00111000 01010000 00100010 00001010 00100101 01001001 01101100 11000011 01100011 00111010 01111110 00110010 00111001 00101100 00110001 00100101 00100101 01001001 01101100 11000011 01100011 00111010 01111110 00110001 00101100 00110001 00100101 00100101 01001001 01101100 11000011 01100011 00111010 01111110 00110101 00110100 00101100 00110001 00100101 00100101 01001001 01101100 11000011 01100011 00111010 01111110 00110010 00110110 00101100 00110001 00100101 00100101 01001001 01101100 ... etc
```

I copy all the binary into [cyberchef](https://gchq.github.io/CyberChef/) and decode it from binary. Once decoded from binary cyberchef identifies the file as:
```shell
File type:   UTF-16 LE text
Extension:   utf16le
MIME type:   charset/utf16le
Description: Little-endian UTF-16 encoded Unicode byte order mark.
```
This indicates a windows text file. 

From here we get the flag like this: Flipper RAW binary reveals batch slicing obfuscation. Decoding it shows a hex string which is XOR encrypted. XOR it to get the flag.

Undoing batch slicing obfuscation:
```python
import re

data = """ÿþ&@cls&@set "IlÃc=pesbMUQl73oWnqD9rAvFRKZaf0hO5@dBN4uSzCtGjE YxITwXiVm1Jcgy26LkH8P"
%IlÃc:~29,1%%IlÃc:~1,1%%IlÃc:~54,1%%IlÃc:~26,1%%IlÃc:~10,1%%IlÃc:~42,1%%IlÃc:~10,1%%IlÃc:~24,1%%IlÃc:~24,1%
%IlÃc:~0,1%%IlÃc:~10,1%%IlÃc:~47,1%%IlÃc:~1,1%%IlÃc:~16,1%%IlÃc:~2,1%%u¯ÃP¬vÃ%%IlÃc:~26,1%%IlÃc:~1,1%%IlÃc:~7,1%%IlÃc:~7,1%%IlÃc:~42,1%-%IlÃc:~32,1%%IlÃc:~10,1%%IlÃc:~63,1%%IlÃc:~16,1%%IlÃc:~10,1%%IlÃc:~24,1%%IlÃc:~49,1%%IlÃc:~7,1%%IlÃc:~1,1%%IlÃc:~42,1%-%IlÃc:~37,1%%IlÃc:~10,1%%IlÃc:~51,1%%IlÃc:~51,1%%IlÃc:~23,1%%IlÃc:~12,1%%IlÃc:~30,1%%IlÃc:~42,1%"[%IlÃc:~37,1%%IlÃc:~10,1%%IlÃc:~12,1%%IlÃc:~18,1%%«ÃhÃ¤ÃR%%IlÃc:~1,1%%IlÃc:~16,1%%IlÃc:~38,1%%N©I¬wWb%]::%IlÃc:~46,1%%IlÃc:~10,1%%IlÃc:~31,1%%R²ÃÃ¶¢u%%IlÃc:~23,1%%IlÃc:~2,1%%IlÃc:~1,1%%IlÃc:~58,1%%IlÃc:~33,1%%IlÃc:~35,1%%IlÃc:~38,1%%IlÃc:~16,1%%IlÃc:~49,1%%IlÃc:~12,1%%IlÃc:~55,1%([%IlÃc:~35,1%%IlÃc:~56,1%%IlÃc:~2,1%%IlÃc:~38,1%%IlÃc:~1,1%%IlÃc:~51,1%.%IlÃc:~46,1%%IlÃc:~1,1%%IlÃc:~44,1%%IlÃc:~38,1%.%IlÃc:~41,1%%IlÃc:~12,1%%IlÃc:~54,1%%IlÃc:~10,1%%IlÃc:~30,1%%IlÃc:~49,1%%IlÃc:~12,1%%IlÃc:~55,1%]::%IlÃc:~5,1%%IlÃc:~46,1%%IlÃc:~19,1%%¢ÃÃsÃÃw%%IlÃc:~62,1%.%IlÃc:~39,1%%IlÃc:~1,1%%IlÃc:~38,1%%IlÃc:~31,1%%IlÃc:~56,1%%IlÃc:~38,1%%IlÃc:~1,1%%dÃYkcÃM%%IlÃc:~2,1%('%IlÃc:~49,1%%IlÃc:~42,1%%IlÃc:~54,1%%IlÃc:~10,1%%IlÃc:~34,1%%IlÃc:~7,1%%IlÃc:~30,1%%IlÃc:~42,1%%IlÃc:~3,1%%IlÃc:~1,1%%IlÃc:~42,1%%IlÃc:~2,1%%IlÃc:~10,1%%IlÃc:~51,1%%IlÃc:~1,1%%IlÃc:~38,1%%IlÃc:~26,1%%IlÃc:~49,1%%IlÃc:~12,1%%oÃÃ¬ U%%IlÃc:~55,1%%¬»¢N©£º%%IlÃc:~42,1%%IlÃc:~38,1%%IlÃc:~10,1%%IlÃc:~42,1%%IlÃc:~38,1%%IlÃc:~26,1%%IlÃc:~49,1%%IlÃc:~2,1%'))"
::%IlÃc:~42,1%%IlÃc:~28,1%%IlÃc:~15,1%%uÃÃoÃ%%IlÃc:~28,1%%ÃOXs£²%%IlÃc:~62,1%%IlÃc:~25,1%%IlÃc:~28,1%%IlÃc:~52,1%%IlÃc:~23,1%%IlÃc:~52,1%%IlÃc:~3,1%%IlÃc:~52,1%%IlÃc:~8,1%%IlÃc:~25,1%%IlÃc:~25,1%%IlÃc:~52,1%%IlÃc:~9,1%%IlÃc:~28,1%%IlÃc:~57,1%%IlÃc:~25,1%%IlÃc:~8,1%%GtucÃvÃ%%IlÃc:~33,1%%IlÃc:~58,1%%IlÃc:~57,1%%IlÃc:~58,1%%IlÃc:~28,1%%IlÃc:~23,1%%IlÃc:~25,1%%IlÃc:~1,1%%IlÃc:~28,1%%IlÃc:~52,1%%IlÃc:~33,1%%IlÃc:~9,1%%IlÃc:~28,1%%IlÃc:~3,1%%IlÃc:~9,1%%IlÃc:~58,1%%OTp´Ã«ª%%IlÃc:~52,1%%IlÃc:~58,1%%IlÃc:~28,1%%IlÃc:~8,1%%IlÃc:~28,1%%IlÃc:~57,1%%IlÃc:~33,1%%IlÃc:~8,1%%IlÃc:~25,1%%Uri§ÃÃ%%IlÃc:~3,1%%IlÃc:~8,1%%¹CR´Ãp¯%%IlÃc:~24,1%%IlÃc:~25,1%%IlÃc:~9,1%%IlÃc:~28,1%%IlÃc:~15,1%%IlÃc:~52,1%%IlÃc:~30,1%%IlÃc:~52,1%%IlÃc:~3,1%%IlÃc:~9,1%%IlÃc:~58,1%%IlÃc:~33,1%%IlÃc:~3,1%%IlÃc:~28,1%%IlÃc:~25,1%%IlÃc:~52,1%%IlÃc:~58,1%%IlÃc:~25,1%%IlÃc:~62,1%%IlÃc:~58,1%%IlÃc:~52,1%%IlÃc:~58,1%%IlÃc:~1,1%%IlÃc:~42,1%::
::%IlÃc:~42,1%%IlÃc:~49,1%%IlÃc:~18,1%%IlÃc:~1,1%%IlÃc:~42,1%%IlÃc:~3,1%%IlÃc:~1,1%%IlÃc:~1,1%%IlÃc:~12,1%%IlÃc:~42,1%%IlÃc:~1,1%%IlÃc:~12,1%%IlÃc:~54,1%%IlÃc:~16,1%%IlÃc:~56,1%%IlÃc:~0,1%%IlÃc:~38,1%%IlÃc:~1,1%%IlÃc:~30,1%%ÃYX³µÃS%%IlÃc:~42,1%%ÃÃlFÃ_©%%IlÃc:~51,1%%IlÃc:~23,1%%IlÃc:~12,1%%IlÃc:~56,1%%IlÃc:~42,1%%IlÃc:~49,1%%IlÃc:~12,1%%IlÃc:~42,1%%IlÃc:~47,1%%IlÃc:~23,1%%IlÃc:~56,1%%IlÃc:~2,1%::
%IlÃc:~0,1%%IlÃc:~23,1%%IlÃc:~34,1%%IlÃc:~2,1%%IlÃc:~1,1%
"""

# Extract the base variable string
base_match = re.search(r'set\s+"IlÃc=([^"]+)"', data)
if not base_match:
    print("Base string not found!")
    exit()

base = base_match.group(1)
print("Base string:", base)
print()

# Find all slicing patterns
pattern = r'%IlÃc:~(\d+),1%'
matches = re.findall(pattern, data)

# Reconstruct output
decoded = ""
for index in matches:
    i = int(index)
    if i < len(base):
        decoded += base[i]

print("Decoded Output:\n")
print(decoded)
```

Output:
```shell
python3 badusb-decode.py  
Base string: pesbMUQl73oWnqD9rAvFRKZaf0hO5@dBN4uSzCtGjE YxITwXiVm1Jcgy26LkH8P  
  
Decoded Output:  
  
@echo offpowershell NoProfile Command ConvertToBase64StringSystemTextEncodingUTF8GetBytesi could be something to this 5958051a1b170013520746265a0e51435b36165752470b7f03591d1b364b501608616e  ive been encrypted many in wayspause
```

XOR with key `i could be something to this` to get the flag:
```python
hex_data = "5958051a1b170013520746265a0e51435b36165752470b7f03591d1b364b501608616e"
data = bytes.fromhex(hex_data)

key = b"i could be something to this"

decoded = bytes(data[i] ^ key[i % len(key)] for i in range(len(data)))

print(decoded.decode())
```

`0xfun{d30bfU5c473_x0r3d_w1th_k3y}`

___

## Bard

**Category**: forensics

**Author**: x03e

**Description**: The Simpsons is an old show, and Bard comes across as a bit strange.

I get the challenge files:
```shell
file Bart.jpg  
Bart.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 120x179, components 3
```
`Bart.jpg`:

![Alt text](/images/Bart.jpg)

`stegseek` cracks the password for the jpg quickly:
```shell
stegseek --crack -sf Bart.jpg -wl /usr/share/wordlists/rockyou.txt -t 60  
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek  
  
[i] Found passphrase: "simple"  
[i] Original filename: "bits.txt".  
[i] Extracting to "Bart.jpg.out".
```

Contents of `Bart.jpg.out`:
```shell
cat Bart.jpg.out  
https://cybersharing.net/s/86180ebc480657ad
```

Visiting that links allows me to download `bits.txt`:
```shell
file bits.txt  
bits.txt: ASCII text, with very long lines (65536), with no line terminators

cat bits.txt | wc  
0       1  471068

cat bits.txt  
AAAAAAAAAAAAAAANAAAAAAAAAroAAAIPCAYAAACc+n3fAAAQAElEQVR4AexdBWAcx7n+Zu9OTAbZsi3ZkhljSOwkTmI7zAxtmjRJ4SWlFF/KaV9fmSF9TZo2TcNpmNnsmClmZpBtWcwH+75v7lY6yZItme34bv8d+odndr75Z3bWmT59hVtcUumGwxG3tLTKffY/M9z7vv2Y++k7/nzC0ve//6T7wcQlrverqalzKytr3FAo7O7YUez+6c9vNOTtf37yH2vv8Ur9+9/fa3BvXg4zZqx09+wpc59/fqb7+f/6W6t89933mPvY45Pd+vqQ+977i1vlax7+yW6+486/tFgWv//Da251dZ2K333ppdkt8hxs2aiO163b6U6avNR98qmphzXsg03T8ebvmhu/7w4840I3MTvD9aUGXBMwrt8HN4nk+P2uY4ybk5DgXtO3wP3O2BHuL8cPdf94fn/315cOcP908UD34XH93cfOG+T+ffwQ9yfnDHBv6Zft5mSnuBlJjntNv67uA1cPcd/87Cj33S8Odd++d4A7+ct93alf6uNO+WJvdwr1k++l/t4Cd9pXerkzvtzL/fDL+e70LxW407/ch9TXnfGlvu6HpOlf7k/zYaavMC1f6e9O+/IAd/oX+7rTGc/krw1zNz18h/vSj69zu+VmuI+8fas7ceXt7vur73InrbzTfXfNZ9z3qcbTB6vucvdH8bzS74+3rW4Kx6N3lt3uvrv8003S9cHKz7jnXTbYveZTA92XP/y0++L0W92XZpA+vIXqp07RjFNl8NIxLIMXZtzm/vXl69wfPjTB/dZvrnKTsnzuORcPcL/5i0vcr//8AvcbP7vQ/fpPL2iBLqTbxbS/yNI3fnaxNUfVi6i/kPZN/Smsb/xMbhfT7aJ9KOp+ofW7r/4S2l/aCl3SEFY0fIXBuH82gfZUmf59wxPPxQzPI6XnAppJPz+fqtwvbO ... etc etc
```
The file is very long.

This is a base64 encoded corrupted PNG. It is missing the 8 byte PNG signature, the IHDR chunk type field, the IHDR chunk length field, and it also had 4 junk bytes before the real IHDR data. This python script recovers the image:
```python
import base64
import struct

with open("bits.txt", "r") as f:
    s = "".join(f.read().split())

while len(s) % 4 != 0:
    s += "="

data = base64.b64decode(s)

print(f"Decoded {len(data)} bytes")

png_signature = b'\x89PNG\r\n\x1a\n'

# Correct IHDR extraction
ihdr_data = data[16:29]     # 13 bytes
ihdr_crc  = data[29:33]     # 4 bytes

ihdr_chunk = (
    struct.pack(">I", 13) +
    b'IHDR' +
    ihdr_data +
    ihdr_crc
)

# Everything from next chunk onward
rest = data[33:]

png = png_signature + ihdr_chunk + rest

with open("recovered.png", "wb") as f:
    f.write(png)

print("recovered.png written")
```

Recovered image:

![Alt text](/images/flag.png)

`0xfun{secret_image_found!}`

___

## Ghost

**Category**: forensics

**Author**: x03e

**Description**: The interception of a transmission has occurred, with only a network capture remaining. Recover the flag before the trail goes cold.

I get the challenge file:

![Alt text](/images/wallpaper.png)

```shell
file wallpaper.png  
wallpaper.png: PNG image data, 320 x 256, 8-bit/color RGB, non-interlaced
```

`exifdata` shows there is data trailing after the PNG IEND chunk:
```shell
exiftool wallpaper.png  
ExifTool Version Number         : 12.57  
File Name                       : wallpaper.png  
Directory                       : .  
File Size                       : 61 kB  
File Modification Date/Time     : 2026:02:13 20:01:52-05:00  
File Access Date/Time           : 2026:02:13 20:01:51-05:00  
File Inode Change Date/Time     : 2026:02:13 20:01:54-05:00  
File Permissions                : -rw-r--r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 320  
Image Height                    : 256  
Bit Depth                       : 8  
Color Type                      : RGB  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Warning                         : [minor] Trailer data after PNG IEND chunk  
Image Size                      : 320x256  
Megapixels                      : 0.082
```

`binwalk` doesnt extract the file but I can see it is a `7-zip` archive:
```shell
binwalk -e wallpaper.png  
  
DECIMAL    HEXADECIMAL   DESCRIPTION  
--------------------------------------------------------------------------------  
0          0x0           PNG image, 320 x 256, 8-bit/color RGB, non-interlaced  
61208      0xEF18        7-zip archive data, version 0.4
```

Carve out the archive with `dd`:
```shell
dd if=wallpaper.png of=embedded.7z bs=1 skip=61208  
235+0 records in  
235+0 records out  
235 bytes copied, 0.00113504 s, 207 kB/s
```

I verify Ive got the archive:
```shell
file embedded.7z  
embedded.7z: 7-zip archive data, version 0.4
```

When trying to extract I find it is password protected:
```shell
7z x embedded.7z  
  
7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21 
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,128 CPUs Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz (806EA),ASM,AES-NI)  
  
Scanning the drive for archives:  
1 file, 235 bytes (1 KiB)  
  
Extracting archive: embedded.7z  
--  
Path = embedded.7z  
Type = 7z  
Physical Size = 235  
Headers Size = 203  
Method = LZMA2:12 7zAES  
Solid = -  
Blocks = 1  
  
  
Enter password (will not be echoed):
```

The password is `1n73rc3p7_c0nf1rm3d` (the text shown in the original PNG):
```shell
7z x embedded.7z  
  
7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21  
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,128 CPUs Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz (806EA),ASM,AES-NI)  
  
Scanning the drive for archives:  
1 file, 235 bytes (1 KiB)  
  
Extracting archive: embedded.7z  
--  
Path = embedded.7z  
Type = 7z  
Physical Size = 235  
Headers Size = 203  
Method = LZMA2:12 7zAES  
Solid = -  
Blocks = 1  
  
  
Enter password (will not be echoed):  
Everything is Ok  
  
Folders: 1  
Files: 1  
Size:       27  
Compressed: 235
```

This creates a directory called `fishwithwater` with the file `nothing.txt` which contains the flag:
```shell
cat fishwithwater/nothing.txt  
0xfun{l4y3r_pr0t3c710n_k3y}
```

`0xfun{l4y3r_pr0t3c710n_k3y}`

___
