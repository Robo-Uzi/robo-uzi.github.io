---
layout: post
title:  "Beginner Challenges"
date:   2026-08-04 17:35:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /L3AK-CTF-beginner/
---
* TOC
{:toc}
## me fr

**Author**: JAGIC

**Category**: Beginner misc

**Description**: monkey type has nothing on me fr no cap

I get the challenge file:
```shell
cat me_fr.txt  
Jo! Sp O was tjomlomg/// tu[omg os kist sp jard mpwadaus! :pplomg at upir jamds wjo;e upi tu[e os omfiroatomg. Amuwaus. jeres tje f;ag" :3AL}WJU+D-+1+dP+TJ1S+s-+-f83M///|
```

In this cipher the right hand is shifted one key to the right and the left hand types normally. To decode:
- Left hand keys (`qwert`, `asdfg`, etc.) stay the same
- Right hand keys (`yuiop`, `hjkl;`, `nm,./`, `6-0`, etc) are moved one key to the left on the keyboard
Example: `Jo!` > `Hi!` (because `J` > `H`, `o` > `i`).

Solve script:
```python
msg = "Jo! Sp O was tjomlomg/// tu[omg os kist sp jard mpwadaus! :pplomg at upir jamds wjo;e upi tu[e os omfiroatomg. Amuwaus. jeres tje f;ag\" :3AL}WJU+D-+1+dP+TJ1S+s-+-f83M///|"

# standard qwerty right hand key sequences (lowercase and shifted)
rows = [
    "67890-=", "^&*()_+",
    "yuiop[]\\", "YUIOP{}|",
    "hjkl;'", "HJKL:\"",
    "nm,./", "NM<>?"
]

# map every right hand key to the key immediately to its left
decode_map = str.maketrans({
    row[i]: row[i-1] 
    for row in rows 
    for i in range(1, len(row))
})

print(msg.translate(decode_map))
```

Output:
```shell
python3 solve2.py  
Hi! So I was thinking... typing is just so hard nowadays! Looking at your hands while you type is infuriating, Anyways, heres the flag: L3AK{WHY_D0_1_dO_TH1S_s0_0f73N...}
```

`L3AK{WHY_D0_1_dO_TH1S_s0_0f73N...}`

___

## BabyLCG

**Author**: aresinheaven

**Category**: Beginner crypto

**Description**: We intercepted this custom encryption script from our Jr Developers miikie and iris. They thought they could use basic math as a secure random number generator. Can you break it?

I downloaded the challenge files and got `chall.py` and `output.txt`. Contents of `chall.py`:
```python
import os
FLAG = b"L3AK{??????????????????????????}"
class LCG:
    def __init__(self, m):
        self.a = int.from_bytes(os.urandom(16), "big") % m
        self.c = int.from_bytes(os.urandom(16), "big") % m
        self.state = int.from_bytes(os.urandom(16), "big") % m
        self.m = m
    def next(self):
        self.state = (self.a * self.state + self.c) % self.m
        return self.state
m = 115792089237316195423570985008687907853269984665640564039457584007913129640233
rng = LCG(m)
s0 = rng.next()
s1 = rng.next()
s2 = rng.next()
key = rng.next()
flag_int = int.from_bytes(FLAG, "big")
ciphertext = flag_int ^ key
print(f"m = {m}")
print(f"s0 = {s0}")
print(f"s1 = {s1}")
print(f"s2 = {s2}")
print(f"ct = {ciphertext}")
```

`output.txt`:
```shell
m = 88044978735773602913395349457408066612245192322881563734438993831688084200491
s0 = 4452065008288242560629390669208864932242141417756588067313178112477164149842
s1 = 30356301725547557665274966292036883630163427635439138410477840356169747135880
s2 = 33330863090985168864945055645699247424789280002692545918305324950320521259312
ct = 8850041716144071587274828779665113489634774808247082181515445941038495956603515
```

Solve script:
```python
m = 88044978735773602913395349457408066612245192322881563734438993831688084200491
s0 = 4452065008288242560629390669208864932242141417756588067313178112477164149842
s1 = 30356301725547557665274966292036883630163427635439138410477840356169747135880
s2 = 33330863090985168864945055645699247424789280002692545918305324950320521259312
ct = 8850041716144071587274828779665113489634774808247082181515445941038495956603515

# a = (s2 - s1) * inv(s1 - s0) mod m
diff_s2_s1 = (s2 - s1) % m
diff_s1_s0 = (s1 - s0) % m
a = (diff_s2_s1 * pow(diff_s1_s0, -1, m)) % m

# c = (s1 - a * s0) mod m
c = (s1 - (a * s0)) % m

# generate the key
key = (a * s2 + c) % m

# decrypt the ciphertext via xor
flag_int = ct ^ key

# convert the integer back to bytes
flag_bytes = flag_int.to_bytes((flag_int.bit_length() + 7) // 8, byteorder='big')

print(f"Recovered a: {a}")
print(f"Recovered c: {c}")
print(f"Key        : {key}")
print(f"Flag       : {flag_bytes.decode('utf-8')}")
```

Output:
```shell
python3 solve3.py  
Recovered a: 61922762077714954724422339487460325520  
Recovered c: 268484757612868661764668536000873505602  
Key        : 42268852666721492671716026186655701701015530700633888549581788881595140956166  
Flag       : L3AK{n3v3r_trU5t_b4s1c_LCG5_frfr}
```

`L3AK{n3v3r_trU5t_b4s1c_LCG5_frfr}`

___

## Crossroads

**Author**: Eternal

**Category**: Beginner osint

**Description**: A breathtakingly scenic intersection! I hope we can get cell service out here.

I go to the challenge link `https://geosint.ctf.l3ak.team/crossroads` and I get the location:

![Alt text](/images/leakgeosint2.png)

When I zoomed in, I could see a sign that said `TRAIL CR RD`. From another sign I know this is in `Idaho`. 

I found the location on google maps here: [google maps link](https://www.google.com/maps/place/Trail+Creek+Rd,+Idaho/@44.064643,-113.8787572,1934a,75y,161.02h,90t/data=!3m7!1e1!3m5!1sste1s9A3HgqlsMkjlG-_PA!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D0%26panoid%3Dste1s9A3HgqlsMkjlG-_PA%26yaw%3D161.02480324788843!7i16384!8i8192!4m6!3m5!1s0x54a9c19acd1c6995:0xb67ffaf389db921c!8m2!3d43.9034808!4d-114.1410511!16s%2Fg%2F1trcg4c6?entry=ttu&g_ep=EgoyMDI2MDcyOS4wIKXMDSoASAFQAw%3D%3D)

`L3AK{S1gNs_M4k3_051Nt_RaTh3R_SimPLE!}`
