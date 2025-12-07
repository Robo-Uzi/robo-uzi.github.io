---
layout: post
title:  "Witching Hour"
date:   2025-12-07 16:50:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /nullCTF-witching-hour/
---

**Title**: Witching Hour

**Category**: Crypto

**Author**: xseven

**Description**: Long before prime numbers were tamed by modern cryptographers, the geomancers of the Old Continent whispered of a curve that could bind infinity to a single point. Forged from immense prime-born fractions and marked by coordinates chosen from the deepest reaches of number-space, this ancient shape reveals its secret only to those who can reconstruct the hidden parameter that defines it.

I download the challenge files:
```shell
unzip witching_hour.zip  
Archive:  witching_hour.zip  
 inflating: ciphertext.hex             
 inflating: gen.py                     
 inflating: points.txt
```

```shell
cat ciphertext.hex  
f5870fd25563ab45ee1b40757ce93a93f3c63ee4b8306bb69178bd04007141cdf3ad53d86956aa509437167a4dbb66d39c876ee2812628dfd022be6c43660d98aec053880520f9089176077225bb6ed69f8366e2822721dfd626bb26
```

```shell
cat points.txt  
x=31729929709531991238858692267790989938148294477274382186973702091108444710594484374868433100490725567245070026456718857777925311703224902592006961093706383382943098897971950... (shortened), y=134015481029686172419082680976198608922106366768555506435253876368025598036093331558617899388254103305079467098476328811174723478216554308788021790225823223965155767114633951864251164954219991656156886529961607559199322940344234329422832087678942854765... (shortened)
x=21333104245641355247933922762015740962632523057151152094229152525911554466689627637039263673579033637616603207134960447485304119298347214454952681778494543883073817451762053... (shortened), y=134015481029686172419082680976198608922106366768555506435253876368025598036093331558617899388254103305079467098476328811174723478216554308788021790225823223965155767114633951864251164954219991656156886529961607559199322940344234329422832087678942854765... (shortened)
```
`points.txt` contains:
```
x=<big_prime>
y=<numerator>/<denominator>
```

`gen.py`:
```python
#!/usr/bin/env python3  
from fractions import Fraction  
from hashlib import sha256  
from Cryptodome.Util import number  
  
bits = 2048  
a = Fraction(number.getPrime(bits), number.getPrime(bits))  
  
x_vals = [number.getPrime(bits), number.getPrime(bits)]  
  
def frac_square(frac):  
    return Fraction(frac.numerator*frac.numerator, frac.denominator*frac.denominator)  
  
def frac_cube(frac):  
    return Fraction(frac.numerator**3, frac.denominator**3)  
  
  
points = []  
for x in x_vals:  
    y = frac_cube(a) / (x*x + frac_square(a))  
    points.append((x, y))  
  
key = sha256(f"{a.numerator}/{a.denominator}".encode()).digest()  
  
flag = b"nullctf{secret}"  
  
ct = bytes([flag[i] ^ key[i % len(key)] for i in range(len(flag))])  
  
print("# points.txt")  
  
lines = []  
for x, y in points:  
    line = f"x={x}, y={y.numerator}/{y.denominator}"  
    print(line)  
    lines.append(line)  
  
points_text = "\n".join(lines)  
  
with open("points.txt", "w") as f:  
    f.write(points_text)  
  
print()  
print("# ciphertext.hex")  
  
cipher_hex = ct.hex()  
print(cipher_hex)  
  
with open("ciphertext.hex", "w") as f:  
    f.write(cipher_hex)
```
`gen.py` generates two "points" based on a secret `a`, then uses `a` to derive a key with sha256 and XOR encrypt the flag. `points.txt` contains `x,y` values derived from `a`. `ciphertext.hex` has the XORed flag.

So the equation is: `y = a^3/(x^2 + a^2)`

- `a`: secret fraction `p/q` (both 2048 bit primes)
- `x`: random prime (given in `points.txt`)
- `y`: computed fraction (given as numerator/denominator)

Both points share the same secret `a`, so two equations let you eliminate `a^3` and solve for `a^2`.

Solve script:
```python
#!/usr/bin/env python3
from fractions import Fraction
from hashlib import sha256
import sys

def load_points(filename):
    # get x and y coordinates from points.txt
    points = []
    with open(filename, 'r') as f:
        for line in f:
            if not line.strip():
                continue
            x_part, y_part = line.strip().split(', ')
            x = int(x_part.split('=')[1])
            y_numer, y_denom = map(int, y_part.split('=')[1].split('/'))
            points.append((x, Fraction(y_numer, y_denom)))
    return points

def isqrt(n):
    # int square root using Newton's method for large integers
    if n < 0:
        raise ValueError("Square root of negative number")
    if n == 0:
        return 0
    x = n
    y = (x + 1) // 2
    while y < x:
        x = y
        y = (x + n // x) // 2
    return x

def is_perfect_square(n):
    # check if n is a perfect square for large integers
    if n < 0:
        return False
    root = isqrt(n)
    return root * root == n

def solve_for_a(points):
    # solve for the unknown fraction 'a' from two points
    (x1, y1), (x2, y2) = points[0], points[1]
    
    # Compute a^2 = (y1*x1^2 - y2*x2^2) / (y2 - y1)
    numer = y1 * (x1 * x1) - y2 * (x2 * x2)
    denom = y2 - y1
    a_sq = Fraction(numer, denom)
    
    # since a = p/q where p and q are primes from gen.py,
    # a^2 = p^2/q^2, so numerator and denominator must be perfect squares
    a_num = isqrt(a_sq.numerator)
    a_den = isqrt(a_sq.denominator)
    a = Fraction(a_num, a_den)
    print(f"Recovered a = {a.numerator}/{a.denominator}")
    return a    

def decrypt_flag(a, ciphertext_hex):
    # generate key from a and decrypt ciphertext
    key_input = f"{a.numerator}/{a.denominator}".encode()
    key = sha256(key_input).digest()
    
    with open(ciphertext_hex, 'r') as f:
        ct_hex = f.read().strip()
    ciphertext = bytes.fromhex(ct_hex)
    
    flag_bytes = bytearray()
    for i in range(len(ciphertext)):
        flag_bytes.append(ciphertext[i] ^ key[i % len(key)])
    
    try:
        return flag_bytes.decode('utf-8')
    except UnicodeDecodeError:
        return flag_bytes.hex()

def main():
    points_file = "points.txt"
    ciphertext_file = "ciphertext.hex"
    
    points = load_points(points_file)
    a = solve_for_a(points)
    flag = decrypt_flag(a, ciphertext_file)
    print(f"Flag: {flag}")

if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve.py  
Recovered a = 23752992038465519553259702023759702117591375512998026538167551614783277010089403372123734947184220141234443671402968990205512643991245993003712325877107190131291745694851295329126280151714789209090131881191736924278757102620648078458958204087754833840669102438841107868678805791407270396360428434739328800495156459416885660395213154882731129343438672963686873643337089119370520924519738999836319708237103077100872421591622944244699480545383119547844671830280013639709896283230620142533340801354598940077774252999435598900339496823418028206418014420691263781598036260132275282539812844588412382809981441404691771718483/23721614605166063345441207023571530640001212440271183933675601795424635214170267025162622052228196813198788932754900911254554352599760921873317159429162764985005643412110998341882342091543847690396417213904879755459345339230402488608357041759454788466713749005830437842544953331274094454912602833948385293144425196588698863062470922655342138236346118765966161042757434756289974997762020110631223169659974846946363302018134583434282986987118816419651894345585502437421151684148106460230674307057287713838408249223711591767735680273881877401044046359799103965394734285507347905468467558944353561607924780618498309112063  
Flag: nullctf{I_w0nder_wh0_!s_th3_w!tch_0f_Agn3s!?_6920686f7065207468652063746620776173206e696365}
```

`nullctf{I_w0nder_wh0_!s_th3_w!tch_0f_Agn3s!?_6920686f7065207468652063746620776173206e696365}`