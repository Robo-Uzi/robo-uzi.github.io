---
layout: post
title:  "Pascal 2026 Crypto Challenges"
date:   2026-02-01 09:29:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /pascalctf-2026-crypto-challenges/
---
* TOC
{:toc}

### XorD

**Category**: Crypto

**Author**: Filippo Boschi `@pllossi`

**Description**: I just discovered bitwise operators, so I guess 1 XOR 1 = 1?

I get the challenge files:
```python
import os  
import random  
  
def xor(a, b):  
    return bytes([a ^ b])  
  
flag = os.getenv('FLAG', 'pascalCTF{REDACTED}')  
encripted_flag = b''  
random.seed(1337)  
  
for i in range(len(flag)):  
    random_key = random.randint(0, 255)  
    encripted_flag += xor(ord(flag[i]), random_key)  
  
with open('output.txt', 'w') as f:  
    f.write(encripted_flag.hex())
```

```shell
cat output.txt  
cb35d9a7d9f18b3cfc4ce8b852edfaa2e83dcd4fb44a35909ff3395a2656e1756f3b505bf53b949335ceec1b70e0
```

Solve script:
```python
import random  
  
cipher_hex = "cb35d9a7d9f18b3cfc4ce8b852edfaa2e83dcd4fb44a35909ff3395a2656e1756f3b505bf53b949335ceec1b70e0"  
cipher = bytes.fromhex(cipher_hex)  
  
random.seed(1337)  
  
flag = b''  
  
for b in cipher:  
    key = random.randint(0, 255)  
    flag += bytes([b ^ key])  
  
print(flag.decode())
```

The flag is encrypted using XOR with a not so random PRNG keystream (`random.seed(1337)`). Regenerating the same keystream and XORing it with the ciphertext recovers the original flag.

`pascalCTF{1ts_4lw4ys_4b0ut_x0r1ng_4nd_s33d1ng}`

___

### Ice Cramer

**Category**: Crypto

**Author**: Alan Davide Bovo `@AlBovo`

**Description**: Elia’s swamped with algebra but craving a new ice-cream flavor, help him crack these equations so he can trade books for a cone! `nc cramer.ctf.pascalctf.it 5002`

I get the challenge file `main.py`:
```python
import os  
from random import randint  
  
def generate_variable():  
    flag = os.getenv("FLAG", "pascalCTF{REDACTED}") # The default value is a placeholder NOT the actual flag  
    flag = flag.replace("pascalCTF{", "").replace("}", "")  
    x = [ord(i) for i in flag]  
    return x  
  
def generate_system(values):  
    for _ in values:  
        eq = []  
        sol = 0  
        for i in range(len(values)):  
            k = randint(-100, 100)  
            eq.append(f"{k}*x_{i}")  
            sol += k * values[i]  
  
        streq = " + ".join(eq) + " = " + str(sol)  
        print(streq)  
  
  
def main():  
    x = generate_variable()  
    generate_system(x)  
    print("\nSolve the system of equations to find the flag!")  
  
if __name__ == "__main__":  
    main()
```

I connect to the server and put the equations in a file:
```shell
nc cramer.ctf.pascalctf.it 5002 > equations.txt

cat equations.txt  
19*x_0 + 79*x_1 + -51*x_2 + 12*x_3 + 55*x_4 + -3*x_5 + 92*x_6 + 42*x_7 + 90*x_8 + 54*x_9 + 24*x_10 + 31*x_11 + -54*x_12 + -57*x_13 + -42*x_14 + -44*x_15 + 29*x_16 + 83*x_17 + -29*x_18 + 69*x_19 + -10*x_20 + 81*x_21 + 44*x_22 + 45*x_23 + -41*x_24 + -49*x_25 + 63*x_26 + -64*x_27 = 34824  
-97*x_0 + 58*x_1 + -57*x_2 + 2*x_3 + -79*x_4 + -78*x_5 + -7*x_6 + 84*x_7 + 57*x_8 + -11*x_9 + 74*x_10 + 14*x_11 + 71*x_12 + -57*x_13 + 31*x_14 + 10*x_15 + -60*x_16 + -81*x_17 + 5*x_18 + 11*x_19 + 0*x_20 + 0*x_21 + 23*x_22 + -98*x_23 + 22*x_24 + -18*x_25 + -61*x_26 + 81*x_27 = -24120  
-42*x_0 + 47*x_1 + -76*x_2 + 9*x_3 + 32*x_4 + -28*x_5 + -33*x_6 + 7*x_7 + 89*x_8 + 32*x_9 + 21*x_10 + -47*x_11 + 62*x_12 + -14*x_13 + -41*x_14 + 9*x_15 + 28*x_16 + 41*x_17 + - 42*x_18 + 74*x_19 + 97*x_20 + 5*x_21 + -46*x_22 + -83*x_23 + 70*x_24 + -56*x_25 + 95*x_26 + -32*x_27 = 16332
etc etc etc...
```

Solve script:
```python
import sympy as sp
import re

with open("equations.txt") as f:
    lines = [line.strip() for line in f if "= " in line]

n = 28
A = []
b = []

for line in lines:
    coeffs = [0]*n

    # Extract terms like "-31*x_0"
    terms = re.findall(r'([+-]?\d+)\*x_(\d+)', line)
    for coef, idx in terms:
        coeffs[int(idx)] = int(coef)

    A.append(coeffs)

    # RHS
    rhs = int(line.split("=")[1].strip())
    b.append(rhs)

A = sp.Matrix(A)
b = sp.Matrix(b)

solution = A.LUsolve(b)

flag_body = ''.join(chr(int(v)) for v in solution)
print("pascalCTF{" + flag_body + "}")
```
`solution` will be a vector of integers which map to ASCII.

`pascalCTF{0h_My_G0DD0_too_much_m4th_:O}`

___

### Linux Penguin

**Category**: Crypto

**Author**: Alan Davide Bovo `@AlBovo`

**Description**: I've just installed Arch Linux and I couldn't be any happier :)
`nc penguin.ctf.pascalctf.it 5003`

I get the challenge file `penguin.py`:
```python
from Crypto.Cipher import AES  
import random  
import os  
  
key = os.urandom(16)  
cipher = AES.new(key, AES.MODE_ECB)  
  
words = [  
    "biocompatibility", "biodegradability", "characterization", "contraindication",  
    "counterbalancing", "counterintuitive", "decentralization", "disproportionate",  
    "electrochemistry", "electromagnetism", "environmentalist", "internationality",  
    "internationalism", "institutionalize", "microlithography", "microphotography",  
    "misappropriation", "mischaracterized", "miscommunication", "misunderstanding",  
    "photolithography", "phonocardiograph", "psychophysiology", "rationalizations",  
    "representational", "responsibilities", "transcontinental", "unconstitutional"  
]  
  
def print_flag():  
    flag = os.getenv("FLAG", "pascalCTF{REDACTED}")  
    print(flag)  
  
def encrypt_words(wordst: list[str]) -> list[str]:  
    encrypted_words = []  
    for word in wordst:  
        padded_word = word.ljust(16)  
        encrypted = cipher.encrypt(padded_word.encode()).hex()  
        encrypted_words.append(encrypted)  
    return encrypted_words  
  
def wallpaper():  
    print("""  
                .88888888:.  
               88888888.88888.  
             .8888888888888888.  
             888888888888888888  
             88' _`88'_  `88888  
             88 88 88 88  88888  
             88_88_::_88_:88888  
             88:::,::,:::::8888  
             88`:::::::::'`8888  
            .88  `::::'    8:88.  
           8888            `8:888.  
         .8888'             `888888.  
        .8888:..  .::.  ...:'8888888:.  
       .8888.'     :'     `'::`88:88888  
      .8888        '         `.888:8888.  
     888:8         .           888:88888  
   .888:88        .:           888:88888:  
   8888888.       ::           88:888888  
   `.::.888.      ::          .88888888  
  .::::::.888.    ::         :::`8888'.:.  
 ::::::::::.888   '         .::::::::::::  
 ::::::::::::.8    '      .:8::::::::::::.  
.::::::::::::::.        .:888:::::::::::::  
:::::::::::::::88:.__..:88888:::::::::::'  
 `'.:::::::::::88888888888.88:::::::::'  
       `':::_:' -- '' -'-' `':_::::'`     
   """)  
  
    print("Welcome to the Penguin's Challenge!")  
  
  
def main():  
    wallpaper()  
      
    selected_words = random.choices(words, k=5)  
    ciphertext = ' '.join(encrypt_words(selected_words))  
      
    for i in range(7):  
        print("Give me 4 words to encrypt or don't write anything to quit (max 16 chars):")  
      
        user_words = [  
            input(f"Word {j+1}: ").strip() for j in range(4)  
        ]  
  
        if any(len(word) > 16 for word in user_words):  
            print("Invalid input. Please enter words with a maximum of 16 characters.")  
            continue  
  
        if any(word == '' for word in user_words):  
            print("Exiting the challenge.")  
            break  
  
        encrypted_words = encrypt_words(user_words)  
        print(f"Encrypted words: {' '.join(encrypted_words)}")  
  
    print("Can you now guess what are these encrypted words?")  
    print(f"Ciphertext: {ciphertext}")  
  
    for i in range(5):  
        guess = input(f"Guess the word {i+1}: ")  
        if guess not in selected_words:  
            print("Wrong guess. Try again.")  
            return  
        print(f"Correct guess: {guess}")  
        selected_words.remove(guess)  
  
    print_flag()  
  
if __name__ == "__main__":  
    main()(venv)
```

When connecting to the server I see this:
```shell
nc penguin.ctf.pascalctf.it 5003  
  
                .88888888:.  
               88888888.88888.  
             .8888888888888888.  
             888888888888888888  
             88' _`88'_  `88888  
             88 88 88 88  88888  
             88_88_::_88_:88888  
             88:::,::,:::::8888  
             88`:::::::::'`8888  
            .88  `::::'    8:88.  
           8888            `8:888.  
         .8888'             `888888.  
        .8888:..  .::.  ...:'8888888:.  
       .8888.'     :'     `'::`88:88888  
      .8888        '         `.888:8888.  
     888:8         .           888:88888  
   .888:88        .:           888:88888:  
   8888888.       ::           88:888888  
   `.::.888.      ::          .88888888  
  .::::::.888.    ::         :::`8888'.:.  
 ::::::::::.888   '         .::::::::::::  
 ::::::::::::.8    '      .:8::::::::::::.  
.::::::::::::::.        .:888:::::::::::::  
:::::::::::::::88:.__..:88888:::::::::::'  
 `'.:::::::::::88888888888.88:::::::::'  
       `':::_:' -- '' -'-' `':_::::'`     
      
Welcome to the Penguin's Challenge!  
Give me 4 words to encrypt or don't write anything to quit (max 16 chars):  
Word 1: biodegradability  
Word 2: microphotography  
Word 3: representational  
Word 4: transcontinental  
Encrypted words: 4628641e9de0a51390d9ba3ad3b26592 869a546c6300d3e0223ee8ec919a4c36 b7f4a391775abe530590b3aba0d391cb dcde088390fd2c57846a47a7eeb0d712  
Give me 4 words to encrypt or don't write anything to quit (max 16 chars):  
Word 1:
```

Solve script:
```python
from pwn import *
import re

HOST = "penguin.ctf.pascalctf.it"
PORT = 5003
context.log_level = "error"

WORDS = [
    "biocompatibility", "biodegradability", "characterization", "contraindication",
    "counterbalancing", "counterintuitive", "decentralization", "disproportionate",
    "electrochemistry", "electromagnetism", "environmentalist", "internationality",
    "internationalism", "institutionalize", "microlithography", "microphotography",
    "misappropriation", "mischaracterized", "miscommunication", "misunderstanding",
    "photolithography", "phonocardiograph", "psychophysiology", "rationalizations",
    "representational", "responsibilities", "transcontinental", "unconstitutional"
]

def main():
    io = remote(HOST, PORT)
    codebook = {}
    io.recvuntil(b"Welcome to the Penguin's Challenge!")
    # 7 encryption rounds
    idx = 0
    for _ in range(7):
        io.recvuntil(b"Give me 4 words")
        batch = WORDS[idx:idx+4]
        idx += 4
        for w in batch:
            io.sendline(w.encode())
        io.recvuntil(b"Encrypted words:")
        line = io.recvline().decode()
        blocks = re.findall(r"[0-9a-f]{32}", line)
        for w, b in zip(batch, blocks):
            codebook[b] = w
    # final ciphertext
    io.recvuntil(b"Ciphertext:")
    line = io.recvline().decode()
    final_blocks = re.findall(r"[0-9a-f]{32}", line)
    # guess words
    for i, block in enumerate(final_blocks, 1):
        io.recvuntil(f"Guess the word {i}:".encode())
        io.sendline(codebook[block].encode())
        io.recvline()

    print(io.recvline().decode().strip())
    io.close()

if __name__ == "__main__":
    main()
```

This exploits AES-ECB's determinism by encrypting the entire supplied wordlist, building a ciphertest to plaintext map, then decoding the challenge ciphertext block by block.

`pascalCTF{why_4r3_th3_bl0ck_4lw4ys_th3_s4m3???}`

___

### Curve Ball

**Category**: Crypto

**Author**: Alan Davide Bovo `@AlBovo`

**Description**: Our casino's new cryptographic gambling system uses elliptic curves for provably fair betting. We're so confident in our implementation that we even give you an oracle to verify points! 

`nc curve.ctf.pascalctf.it 5004`

I get the challenge file `curve.py`:
```python
#!/usr/bin/env python3  
from Crypto.Util.number import bytes_to_long, inverse  
import os  
  
p = 1844669347765474229  
a = 0  
b = 1  
n = 1844669347765474230  
Gx = 27  
Gy = 728430165157041631  
  
FLAG = os.environ.get('FLAG', 'pascalCTF{REDACTED}')  
  
class Point:  
    def __init__(self, x, y):  
        self.x = x  
        self.y = y  
     
    def __add__(self, other):  
        if self.x is None:  
            return other  
        if other.x is None:  
            return self  
        if self.x == other.x and self.y == (-other.y % p):  
            return Point(None, None)  
        if self.x == other.x:  
            s = (3 * self.x**2 + a) * inverse(2 * self.y, p) % p  
        else:  
            s = (other.y - self.y) * inverse(other.x - self.x, p) % p  
        x3 = (s*s - self.x - other.x) % p  
        y3 = (s * (self.x - x3) - self.y) % p  
        return Point(x3, y3)  
      
    def __rmul__(self, scalar):  
        result = Point(None, None)  
        addend = self  
        while scalar:  
            if scalar & 1:  
                result = result + addend  
            addend = addend + addend  
            scalar >>= 1  
        return result  
  
def main():  
    secret = bytes_to_long(os.urandom(8)) % n  
    G = Point(Gx, Gy)  
    Q = secret * G  
      
    print("Curve Ball")  
    print(f"y^2 = x^3 + 1 (mod {p})")  
    print(f"n = {n}")  
    print(f"G = ({Gx}, {Gy})")  
    print(f"Q = ({Q.x}, {Q.y})\n")  
      
    while True:  
        print("1. Guess secret")  
        print("2. Compute k * P")  
        print("3. Exit")  
        choice = input("> ").strip()  

        if choice == '1':  
            try:  
                guess = input("secret (hex): ").strip()  
                if guess.startswith('0x'):  
                    guess = guess[2:]  
                guess = int(guess, 16)  
                if guess == secret:  
                    print(f"Flag: {FLAG}")  
                    break  
                else:  
                    print("Wrong.\n")  
            except:  
                print("Invalid.\n")  

        elif choice == '2':  
            try:  
                k = int(input("k: ").strip())  
                pt = input("P (G/Q/x,y): ").strip()  
                if pt.upper() == 'G':  
                    P = G  
                elif pt.upper() == 'Q':  
                    P = Q  
                else:  
                    x, y = pt.replace('(','').replace(')','').split(',')  
                    P = Point(int(x), int(y))  
                R = k * P  
                if R.x is None:  
                    print("Result: O (infinity)\n")  
                else:  
                    print(f"Result: ({R.x}, {R.y})\n")  
            except:  
                print("Invalid.\n")  

        elif choice == '3':  
            break  
  
if __name__ == "__main__":  
    main()(venv)
```

Connecting to the server shows this:
```shell
nc curve.ctf.pascalctf.it 5004  
Curve Ball  
y^2 = x^3 + 1 (mod 1844669347765474229)  
n = 1844669347765474230  
G = (27, 728430165157041631)  
Q = (78541145897291068, 824890234714504603)  
  
1. Guess secret  
2. Compute k * P  
3. Exit  
>
```

Solve script:
```python
#!/usr/bin/env python3
from pwn import *
from Crypto.Util.number import inverse
from sympy import factorint
from sympy.ntheory.modular import crt
import re

context.log_level = "info"

# curve parameters
p = 1844669347765474229
n = 1844669347765474230
Gx, Gy = 27, 728430165157041631

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, o):
        if self.x is None: return o
        if o.x is None: return self
        if self.x == o.x and (self.y + o.y) % p == 0:
            return Point(None, None)

        if self.x == o.x:
            s = (3*self.x*self.x) * inverse(2*self.y, p) % p
        else:
            s = (o.y - self.y) * inverse(o.x - self.x, p) % p

        x3 = (s*s - self.x - o.x) % p
        y3 = (s*(self.x - x3) - self.y) % p
        return Point(x3, y3)

    def __rmul__(self, k):
        r = Point(None, None)
        a = self
        while k:
            if k & 1:
                r = r + a
            a = a + a
            k >>= 1
        return r
        
    def __eq__(self, o):
        if self.x is None and o.x is None:
            return True
        if self.x is None or o.x is None:
            return False
        return self.x == o.x and self.y == o.y

# connect
io = remote("curve.ctf.pascalctf.it", 5004)
banner = io.recvuntil(b"\n\n").decode()

m = re.search(r"Q = \((\d+), (\d+)\)", banner)
Qx, Qy = map(int, m.groups())

log.success(f"Q = ({Qx}, {Qy})")

G = Point(Gx, Gy)
Q = Point(Qx, Qy)

mods = []
vals = []

for prime, exp in factorint(n).items():
    q = prime ** exp
    k = n // q

    Gq = k * G
    Qq = k * Q

    # useless subgroup
    if Gq.x is None:
        continue

    found = False
    for i in range(q):
        if i * Gq == Qq:
            mods.append(q)
            vals.append(i)
            found = True
            break

    # IMPORTANT: skip if not found
    if not found:
        continue

secret, _ = crt(mods, vals)
secret = int(secret) % n

log.success(f"Recovered secret = {hex(secret)}")

io.recvuntil(b"> ")
io.sendline(b"1")
io.recvuntil(b"secret (hex): ")
io.sendline(hex(secret).encode())

io.interactive()
```

The AI told me this (I used AI as I dont fully understand to begin):
The key vulnerability is that n is smooth: it fully factors into small prime powers, specifically `n = 2 x 3^2 x 5 x 7 x 11 x 13 x 17 x 19 x 23 x 29 x 31 x 37 x 41 x 43 x 47`. The largest factor is just `47`, and `3^2 = 9` is the only power greater than `1`. This makes the ECDLP easy to solve using the Pohlig-Hellman algorithm, which reduces the problem to solving smaller discrete logs modulo each prime power factor of n, then combining them via the Chinese Remainder Theorem (CRT).

Output of solve script:
```shell
python3 solve4.py  
[+] Opening connection to curve.ctf.pascalctf.it on port 5004: Done  
[+] Q = (51931123183815012, 648995545655499058)  
[+] Recovered secret = 0xc44e63bf9fa5ad5  
[*] Switching to interactive mode  
Flag: pascalCTF{sm00th_0rd3rs_m4k3_3cc_n0t_s0_h4rd_4ft3r_4ll}  
[*] Got EOF while reading in interactive  
$    
[*] Interrupted  
[*] Closed connection to curve.ctf.pascalctf.it port 5004
```

`pascalCTF{sm00th_0rd3rs_m4k3_3cc_n0t_s0_h4rd_4ft3r_4ll}`

___
