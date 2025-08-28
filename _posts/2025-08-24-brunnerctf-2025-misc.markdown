---
layout: post
title:  "Brunner 2025 Misc Challenges"
date:   2025-08-24 22:29:10 -0400
author: robo.uzi
tags: [CTF, misc]
permalink: /brunnerctf-2025-misc/
---
* TOC
{:toc}

## The Great Mainframe Bake-Off
**Category:** Misc

**Difficulty:** Beginner  
**Author:** H4N5

It's 1974. Deep inside the crusty vaults of CrumbTrust Bank, a covert team of COBOL developers and pastry chefs created a failsafe for their most prized possession: The Marzipan Reverse Recipe.

This sacred document was too valuable to be stored on paper. So, they did the unthinkable - they encoded it and buried it in the production mainframe under a fake batch job titled IEBCAKED. Only those with knowledge of both banking ops and baking science could ever retrieve it.

Years later, the IEBCAKED job has mysteriously reappeared in the job output spool of an IBM z/OS LPAR. But what's left is a string of mysterious byte codes. It's clearly not hex, not ASCII... maybe something older... something only the graybeards of computing would recognize.

Can you decipher the original recipe and unlock the secret to the perfect butterbyte tart?
`d0-85-97-89-83-85-d9-6d-85-a2-99-85-a5-85-d9-6d-95-81-97-89-a9-99-81-d4-6d-85-88-e3-6d-a2-c9-6d-a2-89-88-e3-6d-84-95-c1-6d-a2-85-94-81-99-c6-95-89-81-d4-6d-95-d6-6d-f0-f7-f9-f1-6d-85-88-e3-6d-95-c9-6d-84-85-a2-e4-6d-a2-81-e6-6d-c3-c9-c4-c3-c2-c5-c0-99-85-95-95-a4-99-82`

Cyberchef recipe: from hex, decode to IBM EBCDIC US-Canada(37), reverse by character.
`brunner{EBCDIC_Was_Used_In_The_1970_On_MainFrames_And_This_Is_The_Marzipan_Reverse_Recipe}`

___

## TheBakingCase
**Category:** Misc

**Difficulty:** Beginner  
**Author:** H4N5
I hid a message for you here, see if you can find it! Take it slow, little by little, bit by bit.

`i UseD to coDE liKe A sLEEp-dEprIVed SqUirRel smasHInG keYs HOPinG BugS would dISApPear THrOugh fEAr tHeN i sPilled cOFfeE On mY LaPTop sCReameD iNTerNALly And bakeD BanaNa bREAd oUt oF PAnIc TuRNs OUT doUGh IS EasIEr tO dEbUG ThaN jaVASCrIPt Now I whIsPeR SWEEt NOtHIngs TO sOurDoUGh StARtERs aNd ThReATEN CrOissaNts IF they DoN'T rIsE My OVeN haS fEWeR CRasHEs tHAN mY oLD DEV sErvER aNd WHeN THInGS BurN i jUSt cAlL iT cARAMElIzEd FeatUReS no moRE meetInGS ThAt coUlD HAVE bEeN emailS JUst MufFInS THAt COulD HAvE BEen CupCAkes i OnCE tRIeD tO GiT PuSh MY cInnAmON rOLLs aND paNICkED WHEn I coUldn't reVErt ThEm NOw i liVe IN PeaCE uNLESs tHe yEast getS IDeas abOVe iTs StATion oR a COOkiE TrIES To sEgfAult my toOTH FILlings`

Make a simple python script to convert uppercase to 1 and lowercase to 0.
```python
text = "i UseD to coDE liKe A sLEEp-dEprIVed..."

binary = ''.join(['1' if c.isupper() else '0' for c in text if c.isalpha()])
message = ''.join([chr(int(binary[i:i+8], 2)) for i in range(0, len(binary), 8)])
print(message)
```

```shell
python3 solve.py  
Here is the flag brunner{I_like_Baking_More_That_Programming} easy peasy ?
```

___

## Based Brunner
**Category:** Misc

**Difficulty:** Beginner  
**Author:** Nissen

Brunsviger is just so _[based](https://www.mathsisfun.com/numbers/bases.html)_, I think I could eat it in any form - from binary to decimal!
**Tip:** This might require a bit of programming, I would recommend looking into the `int()` function in Python.

`encode.py`:
```t
def encode_char(ch: str, base: int) -> str:  
   """  
   Encode a single character into a string of digits in the given base  
   """  
   value = ord(ch)  
   digits = []  
   while value > 0:  
       digits.append(str(value % base))  
       value //= base  
  
   return "".join(reversed(digits))  
  
  
with open("flag.txt") as f:  
   text = f.read().strip()  
  
# Encode the text with all bases from decimal to binary  
for base in range(10, 1, -1):  
   text = " ".join(encode_char(ch, base) for ch in text)  
  
with open("based.txt", "w") as f:  
   f.write(text)  
```
`based.txt` contains binary. Each character in the flag has been encoded repeatedly in bases from 10 down to 2. To recover the flag reverse the process:
```python
text = open("based.txt").read().strip()

for base in range(2, 11):
    chars = text.split()
    text = "".join(
        chr(int(c, base))
        for c in chars
    )

print(text)
```

```shell
python3 solve.py  
brunner{1s_b4s3d}
```

___

## The Yeast Key
**Category:** Misc

**Difficulty:** Easy  
**Author:** H4N5

Many years ago, the legendary baker Lionel Poilâne entrusted us with his original sourdough recipe. It's been locked away in the Vault ever since.

Only Poilâne knew the passphrase, and now he's gone. The only clue left behind? A strand of synthetic DNA extracted from his prized baker's yeast.

Can you decode the DNA and recover the vault key? The DNA is: 
```
CGAGCTAGCTCCCGTGCGTGCGCCCTAGCTGTATACCGGCATAACGTGATATCGTACCTTCTAAATAACGGCATACATCACGTGATATCCTTCGTCATCAATCCATCTATATCTAGCCTTATAACGCGCCTTATCCATAACTCCCTAGCGCAATAACTCCATCGCGGACCTTCTAAATCAATCCATCCCTAACGGACTAGATCAATCCATATCCTTATACATCCCCTTCGATCTAGATAAATACATCCATCCATCACGTGATCTCCCGATCACTCCATACATCTAGACATGCATATCTTC
```

I put the string into cyberchef and use 4 find and replace operations. Replace `A` with `00`, `C` with `01`, `G` with `10`, and `T` with `11`. Then convert from binary to get the flag `brunner{1i0n3l_p0i14n3_m4573r_0f_50urd0u6h_p455phr453_15_cr01554n7V4u17!93}`.

___

## Pie Recipe
**Category:** Misc

**Difficulty:** Easy  
**Author:** H4N5

I found this pie recipe. It is simply named "The Recipe of the Golden Phi" by baker Zeckendorf. Can you help me figure out who made it 
```
89|89.21|55.13.5.1|34.13.2|89.8.1|89.13.5.2|34.13.5.1|89.13.5.1|89.8.2|89.21|89.21.5|34.13.3.1|89.8|55.13|55.21.2|89.13|89.1|89.21.8.3.1|55.8.2|89.21.8.2|89.1|55.13|55.21.2|89.21.5.2|55.21.8.3.1|34.13.3.1|55.8.3|89.21.1|55.21.1|55.21.8.2|55.1|89.21.8.1|89.1|89.13.5.1|55.2|34.13.5.2|89.1|55.21.8.3|55.21.2|89.21.3.1|89.1|55.21.8.3|34.13.5.1|89.13.5|89.8.1|34.13.3.1|55.13.5.1|89.13.5.2|89.13|55.21.5|55.5.1|55.5.1
```

These are Fibonacci numbers. It uses `|` as a delimeter. The `.` separates the list of Fibonacci numbers (example `55.21.8.3.1`). If you sum the numbers in each part you get a byte value. Converting those bytes to characters gets you a base64 encoded string. This python script solves it:
```python
import base64 
s = "89|89.21|55.13.5.1|34.13.2|89.8.1|89.13.5.2|34.13.5.1|89.13.5.1|89.8.2|89.21|89.21.5|34.13.3.1|89.8|55.13|55.21.2|89.13|89.1|89.21.8.3.1|55.8.2|89.21.8.2|89.1|55.13|55.21.2|89.21.5.2|55.21.8.3.1|34.13.3.1|55.8.3|89.21.1|55.21.1|55.21.8.2|55.1|89.21.8.1|89.1|89.13.5.1|55.2|34.13.5.2|89.1|55.21.8.3|55.21.2|89.21.3.1|89.1|55.21.8.3|34.13.5.1|89.13.5|89.8.1|34.13.3.1|55.13.5.1|89.13.5.2|89.13|55.21.5|55.5.1|55.5.1"  

tokens = s.split("|")  
bytes = [ sum(map(int,t.split("."))) for t in tokens ]  
b64 = "".join(chr(b) for b in bytes)   
flag = base64.b64decode(b64).decode()  
print(flag)
```
`brunner{7h3_g01d3n_ph1_0f_zeckendorf}`

___

## Bakerman
**Category:** Misc

**Difficulty:** Easy-Medium  
**Author:** H4N5

Oh no, the baker has been going around in his new sports car to deliver brunnere, but he has lost the recipe in the sports car somewhere, can you help find it? I think it's because he has been listening to music on the way.

I unzip the file I am given and see:
```shell
file SportsCar.mp3  
SportsCar.mp3: Zip archive data, at least v2.0 to extract, compression method=deflate
```

I unzip this file also now I have:
```shell
file SportsCar.png  
SportsCar.png: PNG image data, 1536 x 1024, 8-bit/color RGBA, non-interlaced
```

I run `zsteg -a SportsCar.png > output.txt` so I can process the data. Here is a sample of the output:
```shell
cat output.txt | head  
b1,r,lsb,xy         .. text: "SSBsb3ZlIG11c2ljIHdoZW4gSSBiYWtlLCBodHRwczovL3d3dy55b3V0dWJlLmNvbS93YXRjaD92PXh2RlpqbzVQZ0cwDQpVVEpHY2xwVFFtdGlNMVp1WVVSdlRrTnFTWGRKUjJOblpWZFdhR016VVU1RGFrVm5Xa2QzWjJKWGJITmhlWGRuWTIwNWRtSlRRakJhVnpGM1dsaEthR1JJVm5sYVVUQkxUa1JCWjFwNVFtbGtXRkl3"  
b1,r,msb,xy         .. text: "&6JN*vb*"  
b1,g,lsb,xy         .. text: "V2xoSlRrTnFSV2RhVjJSdVJGRnZNRTFEUW01SlNFNHhXakpHZVVSUmIzZE1hbFYzU1VoU2VtTkRRbnBaVjNnd1JGRnZlRWxJUW5CaWJVNXZTVWRrZVdJelZuVmFRMEpxV1ZoS2ExbFhNWFppVVRCTFRXcFZkMGxIWTJka01taHNXVmhSWjFwdGVIWmtXRWxPUTJjd1MxbHVTakZpYlRWc1kyNTBUMDFJWkdabFZFSXhXREIwZFUx"  
b1,b,lsb,xy         .. text: "SVpHWldSMmQ2V0RGSmVsbDZSbmROTXpCT1EyY3dTMUp0YkhOaVIyeDFXbnB2VGtOcVJURk5RMEp1U1VkS2VXSXpaSFZKU0U0eFdqSkdlVVJSYjNoT1ZFRm5XbmxDYVdSWVVqQmFXRWx6U1VoS2RtSXlNR2RrUjFaMFkwZFdlVmxZVWpGamJWVk9RMnBGWjJSSVRuZEpSMlI1WWpOV2RWcERRbXBoVnpWMVdWY3hkbUpuTUVzPQ=="  
b1,rgba,lsb,xy      .. file: old packed data  
b2,g,msb,xy         .. text: "`}%,;\r5("  
b3,b,lsb,xy         .. text: "4\nM&CM%C"  
b3,rgba,lsb,xy      .. text: "g\nrojuo\nt"  
b3p,r,lsb,xy        .. text: "\#$.I)oNEOdoW"  
b3p,g,lsb,xy        .. text: " #\t)`jFkjjk"
```
I pull all base64 encoded strings from the file with regex: `grep -oP '"\K[A-Za-z0-9+/=]{20,}(?=")' output.txt > b64_strings.txt`.

I decode them all in cyberchef to see which ones are a waste of time. The first time I decode them I see `I love music when I bake, https://www.youtube.com/watch?v=xvFZjo5PgG0`. A rickroll. On the right path. I take the only string still not decoded and base64 decode it 2 more times. That gets the flag:
```shell
echo 'Q2FrZSBkb3VnaDoNCjIwIGcgeWVhc3QNCjEgZGwgbWlsaywgcm9vbSB0ZW1wZXJhdHVyZQ0KNDAgZyBidXR0ZXINCjEgZWdnDQo0MCBnIHN1Z2FyDQowLjUwIHRzcCBzYWx0DQoxIHBpbmNoIGdyb3VuZCBjYXJkYW1vbQ0KMjUwIGcgd2hlYXQgZmxvdXINCg0KYnJ1bm5lcntOMHdfeTB1X0tuMHdfVGgzX1IzYzFwM30NCg0KRmlsbGluZzoNCjE1MCBnIGJyb3duIHN1Z2FyDQoxNTAgZyBidXR0ZXIsIHJvb20gdGVtcGVyYXR1cmUNCjEgdHNwIGdyb3VuZCBjaW5uYW1vbg0K' | base64 -d  
Cake dough:  
20 g yeast  
1 dl milk, room temperature  
40 g butter  
1 egg  
40 g sugar  
0.50 tsp salt  
1 pinch ground cardamom  
250 g wheat flour  
  
brunner{N0w_y0u_Kn0w_Th3_R3c1p3}  
  
Filling:  
150 g brown sugar  
150 g butter, room temperature  
1 tsp ground cinnamon
```

___

