---
layout: post
title:  "Crypto Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RAMunchers-CTF-2026-crypto/
---
* TOC
{:toc}

## Escalation Protocol

**Author**: not given

**Description**: Rumor has it that steelsecure.ai is gearing up to grant more RAM to gibson in order to expand it's reasoning capabilities. Multiple employees have gone missing, all with this strange memo left at their desks. We need you to find out what is going on at the halls of steelsecure.

Flag format: RMCTF{name of department}

I get the challenge file `msg.enc`:
```shell
cat msg.enc  
Gtrq: JMX. Fb: Zlfbim Acus  
  
Ws GST ceqrzhutmky.wf dmqnrcjkz  
  
  
Tz cgtvj nj hcdp ws boap btevbv ISF xkprr, yh mgcm npovrtt lh bk ygebf anzo cblzfajjzfz 20% nf vki hhktoyp rsugsknhakr's gptqufmnc uahd dwfimu dnr ilfxuu'b RBH. Jygly sem vpnxryklz pzd hwyj kiga uhlm jiy h ykjdvgt.  
  
KUH Qpzntoxxfzpww:  
  
Om qtuxjoeke RBO xwguaveemgwdd qho wzadpu wlfz gxec fvdhk mymi kdmptb bost lp hgtt xhl lombetulrl nqkczz'g glwkczo dnr urqj vn, zd oab gy tkrp wptnhfe. Hyf jwab tx hkscc uq hjvhzcwpzg B3D0kS1055L4DJ1R3 mwa adbqtijaga.  
  
Rihphh:........ Sqkw:.......
```

This is a progessive vigenere cipher. Its always a hint towards a substitution cipher when you see things like puncuation and casing are preserved. A standard vigenere repeats a short keyword. This one uses a "running key" that increments with every character position.

To find the key, first compare the first 4 characters in the ciphertext with the expected word (from). By calculating the distance between characters, you can extract key values (`key = cipher - plain`):

- Cipher `G` (6), `t` (19), `r` (17), `q` (16)
- Plain: `F` (5), `r` (17), `o` (14), `m` (12)
- Shifts: `+1, +2, +3, +4`

The shift counter does not pause for non alphabetic characters. Spaces, colons, and newlines all increase the value.

I run this python to get the full plaintext:
```python
def decode_full(text, start_offset):
    alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    res = ""
    alpha_idx = start_offset
    for char in text:
        if char.isalpha():
            is_upper = char.isupper()
            c_idx = alphabet.index(char.upper())
            # subtract the current progressive shift
            p_idx = (c_idx - alpha_idx) % 26
            res += alphabet[p_idx] if is_upper else alphabet[p_idx].lower()
        else:
            res += char
        
        # increment shift for every character
        alpha_idx = (alpha_idx + 1) % 26
    return res

msg = """Gtrq: JMX. Fb: Zlfbim Acus

Ws GST ceqrzhutmky.wf dmqnrcjkz


Tz cgtvj nj hcdp ws boap btevbv ISF xkprr, yh mgcm npovrtt lh bk ygebf anzo cblzfajjzfz 20% nf vki hhktoyp rsugsknhakr's gptqufmnc uahd dwfimu dnr ilfxuu'b RBH. Jygly sem vpnxryklz pzd hwyj kiga uhlm jiy h ykjdvgt.

KUH Qpzntoxxfzpww:

Om qtuxjoeke RBO xwguaveemgwdd qho wzadpu wlfz gxec fvdhk mymi kdmptb bost lp hgtt xhl lombetulrl nqkczz'g glwkczo dnr urqj vn, zd oab gy tkrp wptnhfe. Hyf jwab tx hkscc uq hjvhzcwpzg B3D0kS1055L4DJ1R3 mwa adbqtijaga.

Rihphh:........ Sqkw:......."""

print(decode_full(msg, 1))
```

Output:
```shell
python3 solve.py  
From: CEO. To: Junior Devs  
  
To ALL steelsecure.ai employees  
  
  
In order to keep up with rising RAM costs, we have decided to go ahead with transmuting 20% of the backend ddepartment's employees into memory for gibson's GPU. Those who volunteer for this role will get a payrise.  
  
RAM Transmutation:  
  
By accepting RAM transmutation you accept that your short term memory will be used for processing gibson's queries for some of, or all of your workday. You will be moved to department M3M0rY1055M4CH1N3 for processing.  
  
Tjhofe:........ Ebuf:.......
```

`RMCTF{M3M0rY1055M4CH1N3}`

___

## Ea Nasir Secure

**Author**: duppybad

**Description**: gibson has taught the employees at steelsecure to use RSA to secure their work, but we don't trust it entirely.

Enclosed is the source for their encryption, and an output.txt we salvaged from an abandoned laptop

I get the challenge files `chall.py` and `output.txt`. Contents of `chall.py`:
```python
from Crypto.Util.number import getPrime, bytes_to_long
import random

FLAG = b"RMCTF{fake_testing_flag}"


def pad(msg):
    prefix = random.randint(2, 2**16)
    return prefix * (2 ** (8 * len(msg))) + bytes_to_long(msg)


def generate_params():
    e = 3
    p = getPrime(512)
    q = getPrime(512)
    n = p * q
    return n, e


def main():
    n, e = generate_params()
    m = bytes_to_long(FLAG)

    a = random.randint(2, 100)
    b = random.randint(1, 2**32)
    c = random.randint(2, 100)
    d = random.randint(1, 2**32)

    m1 = (a * m + b) % n
    m2 = (c * m + d) % n

    c1 = pow(m1, e, n)
    c2 = pow(m2, e, n)

    print(f"n = {n}")
    print(f"e = {e}")
    print(f"c1 = {c1}")
    print(f"c2 = {c2}")
    print(f"# m1 = {a}*m + {b}")
    print(f"# m2 = {c}*m + {d}")


if __name__ == "__main__":
    main()
```

Contents of `output.txt`:
```shell
n = 98237543086838092972727647602649684412823690703586018468107564793518052420849467378972960087089904634059300894743876081610848224988135902506827923518956599452500642947331481355296570045228580344605876571313298325475476590965722009164468025644000368474389606511878244554300783192096414689616993763058583937333
e = 3
c1 = 10014749067983552801777308442259360701131069253434425322498731759314630313146300050356987559850355269257216025931575623364251535349426572721190934970466052609892864
c2 = 788333017013282582064102996912544428368918059912083172803001714562442559798914489707221771582656784173467830862787901677858526810864572559845148224601602432125
# m1 = 70*m + 2706420314
# m2 = 3*m + 2929618574
```

Solve script:
```python
from Crypto.Util.number import long_to_bytes

n = 98237543086838092972727647602649684412823690703586018468107564793518052420849467378972960087089904634059300894743876081610848224988135902506827923518956599452500642947331481355296570045228580344605876571313298325475476590965722009164468025644000368474389606511878244554300783192096414689616993763058583937333
c1, c2 = 10014749067983552801777308442259360701131069253434425322498731759314630313146300050356987559850355269257216025931575623364251535349426572721190934970466052609892864, 788333017013282582064102996912544428368918059912083172803001714562442559798914489707221771582656784173467830862787901677858526810864572559845148224601602432125
e = 3

# m1 = 70m + 2706420314
# m2 = 3m + 2929618574
params = [(70, 2706420314, c1), (3, 2929618574, c2)]

def get_poly(a, b, c, n):
    # returns coefficients for (ax + b)^3 - c > [a^3, 3a^2b, 3ab^2, b^3 - c]
    return [pow(a, 3, n), (3*a*a*b)%n, (3*a*b*b)%n, (pow(b, 3, n) - c)%n]

def poly_gcd(p1, p2, n):
    while p2:
        while len(p1) >= len(p2):
            mult = (p1[0] * pow(p2[0], -1, n)) % n
            d = len(p1) - len(p2)
            for i in range(len(p2)):
                p1[i] = (p1[i] - mult * p2[i]) % n
            p1.pop(0)
            while p1 and p1[0] == 0: p1.pop(0)
        p1, p2 = p2, p1
    return p1

# attack
f1, f2 = [get_poly(*p, n) for p in params]
res = poly_gcd(f1, f2, n)

# res is [coeff_x, coeff_const] > m = -coeff_const / coeff_x
m = (-res[1] * pow(res[0], -1, n)) % n
print(long_to_bytes(m).decode())
```

Output:
```shell
python3 solve2.py  
RMCTF{C0PP3r_vs_S733L}
```

`RMCTF{C0PP3r_vs_S733L}`
