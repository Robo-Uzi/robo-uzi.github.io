---
layout: post
title:  "el camel"
date:   2025-09-21 10:06:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /K17-2025-el-camel/
---

**Title:** el camel

**Category:** beginner crypto

**Description:** This El-Camel guy's looking a bit weird...

`elcamel.k17.kctf.cloud:1337`

I get this file:
```shell
from secrets import randbelow  
from sympy import isprime  
  
def findGenerator():  
   while True:  
       h = randbelow(p)  
       if pow(h, q, p) != 1:  
           continue  
  
       g = pow(h, 2, p)  
       if g != 1:  
           return g  
  
def rng(key):  
   r = randbelow(p)  
      
   c = pow(g, r * x, p)  
   c += key  
  
   return c % p  
  
if __name__ == "__main__":  
   from secret import FLAG, p, q  
  
   assert isprime(p) and isprime(q)  
  
   g = findGenerator()  
   x = randbelow(q)  
  
   print(f"""The Mystical El-Camel is in town!  
Beat their game to win a special prize...  
  
{p}  
{q}  
""")  
  
   m0 = int(input("How tall do you want the coin to be?> "))  
   m1 = int(input("How long do you want the coin to be?> "))  
  
   m = [m0, m1]  
   score = 0  
   symbols_to_index = {'H': 0, 'T': 1}  
  
   for _ in range(50):  
       i = randbelow(2)  
       c = rng(m[i])  
       print(c)  
  
       print("The coin has been tossed...")  
       guess = input("Heads or Tails! (H or T)> ")  
       guess = symbols_to_index.get(guess.upper())  
  
       if guess == i:  
           print("That's correct!\n")  
           score += 1  
       else:  
           print("Incorrect!\n")  
      
   if score > 37:  
       print("ElCamel is impressed! Here is your prize...")  
       print(FLAG)  
   else:  
       print("Better luck next time!")
```
The challenge script is a coin toss game with a weak random number generator seeded by the input size of the coin. You need to correctly guess heads vs tails at least 38/50 times to get the flag.

This is what I see when connecting:
```shell
nc elcamel.k17.kctf.cloud 1337  
The Mystical El-Camel is in town!  
Beat their game to win a special prize...  
  
105608229528789142631775387999866508795427570969327268682242933574329621868658889587933321616030362833282519833361858498474406413117739010260519171752737365794083267084947959753119521857995326318387692391204942232454020824055547810363275147860366550417464576387096707442458291867676812955650597093180103492943  
52804114764394571315887693999933254397713785484663634341121466787164810934329444793966660808015181416641259916680929249237203206558869505130259585876368682897041633542473979876559760928997663159193846195602471116227010412027773905181637573930183275208732288193548353721229145933838406477825298546590051746471
  
How tall do you want the coin to be?> 10  
How long do you want the coin to be?> 15  
21570446694854487231872376230283224473394748405507221485569114077771354377397711973695057701958259516284169326835152297908692242100511681854083811963214459086560550165464429024376047270296764656324559546537320118251800704148452024130998035654595453302383692448710706141331445697900989904808748322872852263478  
The coin has been tossed...  
Heads or Tails! (H or T)> H  
Incorrect!  
  
98884892485985781810115010988814460725732140446414737980363583590626300141070912429040823123311626419868202243862728664108325486498872033397266633508752785199949386624987556082204956646043771186764793857257135132119781456538809283581328418288495013429374017754917519962865673112172106348859415786236744752338  
The coin has been tossed...  
Heads or Tails! (H or T)> ^C
```

Solve script:
```python
#!/usr/bin/env python3
from pwn import remote, log
import re

HOST, PORT = "elcamel.k17.kctf.cloud", 1337
ROUNDS = 50

log.level = "info"

def read_bigints(text):
    return re.findall(r"\d{10,}", text)

def recv_bigint_line(r):
    while True:
        line = r.recvline(timeout=8)
        if not line:
            raise EOFError("Connection closed.")
        s = line.decode(errors="ignore").strip()
        if re.fullmatch(r"\d{10,}", s):
            return int(s)

def main():
    r = remote(HOST, PORT, timeout=15)
    banner = r.recvuntil(b"How tall do you want the coin to be?>", drop=False).decode(errors="ignore")
    print(banner, end="")

    nums = read_bigints(banner)
    if len(nums) < 2:
        raise SystemExit("Failed to get p and q from banner.")
    p, q = int(nums[0]), int(nums[1])
    log.info(f"Parsed p (digits={len(nums[0])}) and q (digits={len(nums[1])})")

    # Send m0=0 and m1=1
    r.sendline(b"0")
    r.recvuntil(b"How long do you want the coin to be?>")
    r.sendline(b"1")

    for i in range(ROUNDS):
        c = recv_bigint_line(r)
        log.info(f"Round {i+1}: got c = {str(c)[:40]}...")

        # candidates for heads/tails
        cand0 = (c - 0) % p
        cand1 = (c - 1) % p

        # pow(candidate, q, p)
        guess = b"H" if pow(cand0, q, p) == 1 else b"T"
        r.sendline(guess)

        # print server feedback
        try:
            resp = r.recvuntil(b"The coin has been tossed...", timeout=3)
            print(resp.decode(errors="ignore"), end="")
        except Exception:
            pass

    try:
        final = r.recvall(timeout=5).decode(errors="ignore")
    except Exception:
        final = ""
    print(final, end="")
    r.close()

if __name__ == "__main__":
    main()
```
The script connects, reads and extracts the initial banner which contains `p` and `q`. `p` is the large prime modulus and `q` is the subgroup order. It sends two numbers for the coin sides. `m0=0` and `m1=1`. These numbers must be different. 

Then it reads the encrypted coin toss output `c` from the server (`c = g^(r*x) + m[i] mod p`). `r` is random, `x` is secret, `m[i]` is either `m0` or `m1`. By knowing m0 and m1, the difference between Heads/Tails is known. Subtracting `m0`/`m1` gives the random part `g^(r*x)`.

So `guess = b"H" if pow(cand0, q, p) == 1 else b"T"` only passes for the correct candidate. If `cand0` is valid the server chose Heads > send `H`.  Otherwise the server chose Tails > send `T`.

Get the flag:
```shell
python3 solve.py  
[+] Opening connection to elcamel.k17.kctf.cloud on port 1337: Done  
The Mystical El-Camel is in town!  
Beat their game to win a special prize...  
  
105608229528789142631775387999866508795427570969327268682242933574329621868658889587933321616030362833282519833361858498474406413117739010260519171752737365794083267084947959753119521857995326318387692391204942232454020824055547810363275147860366550417464576387096707442458291867676812955650597093180103492943  
52804114764394571315887693999933254397713785484663634341121466787164810934329444793966660808015181416641259916680929249237203206558869505130259585876368682897041633542473979876559760928997663159193846195602471116227010412027773905181637573930183275208732288193548353721229145933838406477825298546590051746471  
  
How tall do you want the coin to be?>[*] Parsed p (digits=309) and q (digits=308)  
Round 1: got c = 9878741017509955795129321551787820854931...  
The coin has been tossed...[*] Round 2: got c = 3292187163799255621989973583358192135614...  
The coin has been tossed...[*] Round 3: got c = 8881639816174063428845145210886175857369...  
The coin has been tossed...[*] Round 4: got c = 2617185088313942295549863806440732380204...  
The coin has been tossed...[*] Round 5: got c = 3793231363605188933515997126079711083102...  
The coin has been tossed...[*] Round 6: got c = 3406600927198116060481655152760113820078...  
The coin has been tossed...[*] Round 7: got c = 2933695424533158189961181837384647323163...  
The coin has been tossed...[*] Round 8: got c = 1715802740173262116487082282436612126719...  
The coin has been tossed...[*] Round 9: got c = 7791717769797566691085417418511156806258...  
The coin has been tossed...[*] Round 10: got c = 5856526222659160300371277965610028239326...  
The coin has been tossed...[*] Round 11: got c = 5167682077218264286533799322312754278643...  
The coin has been tossed...[*] Round 12: got c = 8730777625158703637313123702652319888558...  
The coin has been tossed...[*] Round 13: got c = 7039395829215732712004953613051630487197...  
The coin has been tossed...[*] Round 14: got c = 1625797961135851122237682231067896828606...  
The coin has been tossed...[*] Round 15: got c = 1957546298723548028734851220164581831704...  
The coin has been tossed...[*] Round 16: got c = 7973004379978150123177187364862928657084...  
The coin has been tossed...[*] Round 17: got c = 4222029958858033796534510583585060696404...  
The coin has been tossed...[*] Round 18: got c = 1340397690431890405090846390789268293227...  
The coin has been tossed...[*] Round 19: got c = 6479102854934577745306515888497145717178...  
The coin has been tossed...[*] Round 20: got c = 4105148281324763489165872717188634044961...  
The coin has been tossed...[*] Round 21: got c = 8999619373679892055083213428304516724351...  
The coin has been tossed...[*] Round 22: got c = 1431591662193862990709460331392427618375...  
The coin has been tossed...[*] Round 23: got c = 1840578012596550508374945335344937409976...  
The coin has been tossed...[*] Round 24: got c = 6497605708142931986727527012469539525931...  
The coin has been tossed...[*] Round 25: got c = 2155803510638547145875495138186458862508...  
The coin has been tossed...[*] Round 26: got c = 1022250200957621863737058780502107370965...  
The coin has been tossed...[*] Round 27: got c = 7458299694982183948054037968796448006250...  
The coin has been tossed...[*] Round 28: got c = 6863777976371123619743990742846194218608...  
The coin has been tossed...[*] Round 29: got c = 4879964134670036907045999888618824836830...  
The coin has been tossed...[*] Round 30: got c = 1009289290595734824117697043106966419594...  
The coin has been tossed...[*] Round 31: got c = 4609516111317565880020954965263257362329...  
The coin has been tossed...[*] Round 32: got c = 3522796826837325256107195191537779784402...  
The coin has been tossed...[*] Round 33: got c = 9379816746479221623813609785780107543954...  
The coin has been tossed...[*] Round 34: got c = 8959000224511914032717994637052297709993...  
The coin has been tossed...[*] Round 35: got c = 3150764905184741096090403336121131837222...  
The coin has been tossed...[*] Round 36: got c = 3359002670865230595898103825481189662505...  
The coin has been tossed...[*] Round 37: got c = 1785424330356323370045030474014417977886...  
The coin has been tossed...[*] Round 38: got c = 6712806235419111500258601562663763869203...  
The coin has been tossed...[*] Round 39: got c = 4611982460138910166756060178713128027255...  
The coin has been tossed...[*] Round 40: got c = 4711333795009763888097533655847944837355...  
The coin has been tossed...[*] Round 41: got c = 9588961843865309542264363234365893193701...  
The coin has been tossed...[*] Round 42: got c = 6980631003124122264005117165008827988255...  
The coin has been tossed...[*] Round 43: got c = 8770592690339980648721333859074037879655...  
The coin has been tossed...[*] Round 44: got c = 1917712202620687391366331880107383858205...  
The coin has been tossed...[*] Round 45: got c = 1006819767181797391127522158081398683754...  
The coin has been tossed...[*] Round 46: got c = 1698839832533390135722246478836496514775...  
The coin has been tossed...[*] Round 47: got c = 1813360371924378708888526800496208141121...  
The coin has been tossed...[*] Round 48: got c = 4518456030667092188330814460180158592318...  
The coin has been tossed...[*] Round 49: got c = 7881236018958080750899554921518224647265...  
The coin has been tossed...[*] Round 50: got c = 9772682015032354059109590783698546295857...  
The coin has been tossed...[+] Receiving all data: Done (129B)  
[*] Closed connection to elcamel.k17.kctf.cloud port 1337  
  
Heads or Tails! (H or T)> Incorrect!  
  
ElCamel is impressed! Here is your prize...  
K17{el_gamal_th3_cam3l_h4s_s0me_nic3_squ4r3s}
```

`K17{el_gamal_th3_cam3l_h4s_s0me_nic3_squ4r3s}`