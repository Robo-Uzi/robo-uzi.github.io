---
layout: post
title:  "Crypto Challenges"
date:   2026-06-08 14:36:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /GPN-CTF-2026-crypto/
---
* TOC
{:toc}
## COMpetition

**Category**: Crypto

**Author**: uxxct

**Description**: This week's special: A chance to compete against our esteemed guest, the rock-paper-scissors grand master. If you manage to beat them 100 out of 100 times we will reward you with a flag specially made for you.

I download the challenge files:
```shell
ls -la  
total 16  
drwxr-xr-x 1 user user   86 Jun  5 05:15 .  
drwxrwxr-x 1 user user  568 Jun  5 11:08 ..  
-rw-r--r-- 1 user user 2046 Jun  5 05:04 main.py  
-rw-r--r-- 1 user user  142 Jun  5 05:04 pyproject.toml  
-rw-r--r-- 1 user user    5 Jun  5 05:04 .python-version  
-rw-r--r-- 1 user user  133 Jun  5 05:04 uv.lock
```

Contents of `main.py`:
```python
from hashlib import sha256
from os import environ
from secrets import choice


NUM_ROUNDS = 100


def verify(commitment: bytes, message: bytes, unveil_info: tuple[bytes, bytes]) -> bool:
    """
    Check if a commitment was correctly unveiled.

    :param commitment: The commitment that was unveiled
    :param message: The message the commitment was unveiled to
    :param unveil_info: The unveil information used to unveil the commitment

    :return: Whether the commitment was correctly unveiled
    """

    r1, r2 = unveil_info  # two is better than one, right?

    return commitment == sha256(r1 + message + r2).digest()


def main():
    print("I want to play a game...")
    already_seen = {}
    for _ in range(NUM_ROUNDS):
        com = bytes.fromhex(input("Commitment (hex): "))
        my_choice = choice(["rock", "paper", "scissors"])
        print(f"I choose {my_choice}.")
        your_choice = input("What did you choose? ")
        if your_choice not in {"rock", "paper", "scissors"}:
            print("*Your opponent just staress at you, seeming very confused*")
            return
        unveil_info = tuple(bytes.fromhex(x) for x in input("Proof (hex): ").split())
        if not verify(com, your_choice.encode("ascii"), unveil_info):
            print("Hey, no cheating! Do that again and I will eat all your flags")
            return
        elif my_choice == your_choice:
            print("Sorry, that was a draw. No flag for you")
            return
        elif (my_choice, your_choice) in {
            ("rock", "scissors"),
            ("scissors", "paper"),
            ("paper", "rock"),
        }:
            print("Sorry, you lose. No flag for you")
            return
        elif com in already_seen and already_seen[com] != your_choice:
            print("Something fishy is going on here. What are you doing?")
            return
        already_seen[com] = your_choice
    print(f"How can that be? Well, a deal is a deal. Here is your flag: {environ['FLAG']}")


if __name__ == "__main__":
    main()
```

Connecting to the server:
```shell
ncat --ssl smoked-celery-marinated-in-roasted-curry-dk8u.gpn24.ctf.kitctf.de 443  
I want to play a game...  
Commitment (hex):
```

The vulnerability is in the commitment:
```python
return commitment == sha256(r1 + message + r2).digest()
```

There are no delimiters between `r1`, the message, and `r2`, so you can choose a single long string that contains all three possible moves as substrings:
```python
S = prefix + b"paperscissorsrock" + suffix
```

Once the server sends its random choice, you can pick the winning move. Split `S` into `(r1, r2)` so that `r1 + winning_move + r2 = S`.

- To send `"paper"`:  
`r1 = prefix = b"AB"`  
`r2 = b"scissorsrock" + suffix = b"scissorsrockCD"`  
Check: `r1 + b"paper" + r2 = b"AB" + b"paper" + b"scissorsrockCD" = b"ABpaperscissorsrockCD"`

- To send `"scissors"`:
`r1 = prefix + b"paper" = b"ABpaper"`  
`r2 = b"rock" + suffix = b"rockCD"`  
Check: `b"ABpaper" + b"scissors" + b"rockCD" = b"ABpaperscissorsrockCD"`

- To send `"rock"`:
`r1 = prefix + b"paperscissors" = b"ABpaperscissors"`  
`r2 = suffix = b"CD"`  
Check: `b"ABpaperscissors" + b"rock" + b"CD" = b"ABpaperscissorsrockCD"`

The random `prefix` and `suffix` are needed because the server checks `already_seen[com]`. If you used the same `com` twice with a different move it will reject your move.

Solve script:
```python
from pwn import *
import hashlib
import os

HOST = "smoked-celery-marinated-in-roasted-curry-dk8u.gpn24.ctf.kitctf.de"
PORT = 443
BASE = b"paperscissorsrock"

def main():
    r = remote(HOST, PORT, ssl=True)
    # "I want to play a game..."
    r.recvline()

    for _ in range(100):
        # commit
        r.recvuntil(b"Commitment (hex): ")
        prefix = os.urandom(8)
        suffix = os.urandom(8)
        S = prefix + BASE + suffix
        com = hashlib.sha256(S).hexdigest()
        r.sendline(com.encode())

        # servers move
        line = r.recvline().decode().strip()
        server_choice = line.split()[2].rstrip('.')
        if server_choice == 'rock':
            our_choice = 'paper'
        elif server_choice == 'paper':
            our_choice = 'scissors'
        else:
            our_choice = 'rock'

        # my move
        r.recvuntil(b"What did you choose? ")
        r.sendline(our_choice.encode())

        # proof
        r.recvuntil(b"Proof (hex): ")
        if our_choice == 'paper':
            r1 = prefix
            r2 = b'scissorsrock' + suffix
        elif our_choice == 'scissors':
            r1 = prefix + b'paper'
            r2 = b'rock' + suffix
        else:  # rock
            r1 = prefix + b'paperscissors'
            r2 = suffix
        proof = r1.hex() + ' ' + r2.hex()
        r.sendline(proof.encode())

    # after winning 100 rounds, the server will print the flag
    flag_line = r.recvline().decode().strip()
    print(flag_line)
    r.close()

if __name__ == "__main__":
    main()
```

What is being sent:
```shell
## first 5
Round 1  
Commitment (hex): d4d5354d94adfa814f973e58802731981107e064ec5cef4239a900cb1d6735f1  
I choose paper.  
Server chose paper, we choose scissors  
What did you choose? scissors  
Proof (hex): 89235e992a8ff4af7061706572 726f636b01e8370f5297508d  
  
Round 2  
Commitment (hex): 48f78143297b7039bc22e29eef262d147d5a080d388fbba472a1ec0177753721  
I choose rock.  
Server chose rock, we choose paper  
What did you choose? paper  
Proof (hex): 9d1dfcfb832af082 73636973736f7273726f636b34d6dce5aafb56f7  
  
Round 3  
Commitment (hex): f71de0c8085a5d5fb43ee5e7f43873f2d2112d4b2ac3a358da17ba476cd4e497  
I choose scissors.  
Server chose scissors, we choose rock  
What did you choose? rock  
Proof (hex): 8970fcde304ce32e706170657273636973736f7273 1dd342f487176e42  
  
Round 4  
Commitment (hex): 18880dbdc403bb08bcbd39f6fc2fd1291fd6a16563b76d7c3349d525e52190c5  
I choose rock.  
Server chose rock, we choose paper  
What did you choose? paper  
Proof (hex): 5853c9761bcd43e7 73636973736f7273726f636b4165ae2eedd87523  
  
Round 5  
Commitment (hex): 1dc94096bc3b237a94c1530812c1de5f46c6cbca6e8ee8650d63a49b6a89c7c3  
I choose rock.  
Server chose rock, we choose paper  
What did you choose? paper  
Proof (hex): 0722b1993743e724 73636973736f7273726f636b76209b27586516b1

...etc... until round 100...
```

Output:
```shell
python3 solve.py  
[+] Opening connection to smoked-celery-marinated-in-roasted-curry-dk8u.gpn24.ctf.kitctf.de on port 443: Done  
How can that be? Well, a deal is a deal. Here is your flag: GPNCTF{WAIt, 17'S Not jusT luCk? NeV3r H45 BEen.}  
[*] Closed connection to smoked-celery-marinated-in-roasted-curry-dk8u.gpn24.ctf.kitctf.de port 443
```

`GPNCTF{WAIt, 17'S Not jusT luCk? NeV3r H45 BEen.}`

___

## Easy DSA

**Category**: Crypto

**Author**: uxxct

**Description**: We don't just hand out our prized flags to everyone. Applicants must prove themselves worthy by sharing cool new recipes with us. For security reasons, these recipes of course need to be signed by us.

I have the feeling there might be some logic error here but I just can't figure it out...

I download the challenge files:
```shell
ls -la  
total 24  
drwxr-xr-x 1 user user   86 Jun  5 05:14 .  
drwxrwxr-x 1 user user  676 Jun  5 15:29 ..  
-rw-r--r-- 1 user user 5265 Jun  5 05:04 main.py  
-rw-r--r-- 1 user user  161 Jun  5 05:04 pyproject.toml  
-rw-r--r-- 1 user user    5 Jun  5 05:04 .python-version  
-rw-r--r-- 1 user user 7851 Jun  5 05:04 uv.lock
```

Contents of `main.py`:
```python
from Crypto.PublicKey import ECC
from hashlib import sha256
from uuid import UUID, uuid3

secret_key = ECC.generate(curve="p521")
public_key = secret_key.public_key()
secure_namespace = UUID(bytes=b"kitchenexplosion")

def secure_random(sk: ECC.EccKey, message: bytes) -> int:
    """
    Use sha256 as a secure but deterministic random generator
    to create randomness from the private key and message.

    :param sk: The private signing key
    :param message: The message to sign

    :return: A secure random number
    """

    key_id = uuid3(secure_namespace, sk.export_key(format="PEM")).bytes
    msg_id = uuid3(secure_namespace, message).bytes

    random_generator = sha256(key_id)
    random_generator.update(msg_id)

    return int.from_bytes(random_generator.digest()) % (int(sk._curve.order) - 1) + 1

def hash_message(message: bytes) -> int:
    """
    Create a collision resistant hash for the message to sign.

    :param message: The message to sign

    :return: The hash value
    """

    h = sha256(message).digest()
    return int.from_bytes(h)

def sign(sk: ECC.EccKey, message: bytes) -> tuple[int,int]:
    """
    Sign a message using ECDSA.

    :param sk: The secret key for signing
    :param message: The message to sign

    :return: The signature
    """

    if not sk.has_private():
        raise RuntimeError("That is not a secret key.")

    n = int(sk._curve.order)

    e = hash_message(message)
    z = e & ~(1 << n.bit_length())

    k = secure_random(sk, message)
    P = k * sk._curve.G

    r = P.x % n
    if r == 0:
        raise RuntimeError("Could not create signature")

    s = pow(k, -1, n) * (z + int(r * sk.d)) % n
    if s == 0:
        raise RuntimeError("Could not create signature")

    return (int(r), int(s))

def verify(pk: ECC.EccKey, message: bytes, signature: tuple[int,int]) -> bool:
    """
    Verify an ECDSA signature for a message against a public key.

    :param pk: The public key to verify against
    :param message: The message the signature is for
    :param signature: The signature to verify

    :return: Whether the signature is valid
    """

    Q = pk.pointQ
    r, s = signature
    n = int(pk._curve.order)

    if Q.is_point_at_infinity() or n * Q != Q.point_at_infinity():
        raise RuntimeError("Invalid key")

    if not (1 <= r < n and 1 <= s < n):
        return False

    e = hash_message(message)
    z = e & ~(1 << n.bit_length())

    inv_s = pow(s, -1, n)
    u1 = z * inv_s % n
    u2 = r * inv_s % n

    P = u1 * pk._curve.G + u2 * Q

    if P.is_point_at_infinity():
        return False
    return r == int(P.x % n)

def main():
    from os import environ
    flag = environ.get("FLAG")
    if flag is None:
        raise RuntimeError("Please set the flag as environment variable")

    print(f"Welcome to the Mongolian barbecue.\nUse `sign [hex recipe]` to get your recipe signed by us.\nYou can get the public key to verify signatures using `get pkey`.\nWhen you are ready to taste our freshly made flag, say `flag please`.\nIf you would like to leave, just say `check please`.")
    already_signed = []
    while True:
        command = input("\n> ").lower().split()

        match command:
            case ["sign", recipe]:
                try:
                    recipe = bytes.fromhex(recipe)
                except ValueError:
                    print("That recipe is illegible.")
                s1, s2 = sign(secret_key, recipe)
                already_signed.append(recipe)
                print(f"s1: {hex(s1)}\ns2: {hex(s2)}")

            case ["get", "pkey"]:
                print(f"x: {int(public_key.pointQ.x)}\ny: {int(public_key.pointQ.y)}")

            case ["flag", "please"]:
                print("To decide if you are worthy of our special flag you neeed to provide us a signed recipe we don't already know.")
                recipe = None
                while recipe is None:
                    try:
                        recipe = bytes.fromhex(input("recipe (hex): "))
                    except ValueError:
                        print("That recipe is illegible.")
                if recipe in already_signed:
                    print("We already know that one.")
                    continue
                s1 = None
                while s1 is None:
                    try:
                        s1 = int(input("s1 (hex): "), 16)
                    except ValueError:
                        print("That doesn't look right.")
                s2 = None
                while s2 is None:
                    try:
                        s2 = int(input("s2 (hex): "), 16)
                    except ValueError:
                        print("That doesn't look right.")

                if verify(public_key, recipe, (s1, s2)):
                    print(f"Congratulations. Here is your flag: {flag}")
                else:
                    print("That recipe is not signed properly. No flag for you.")

            case ["check", "please"]:
                print("I hope everything was to your liking. Please visit us again soon.")
                break

            case _:
                print("I CAN'T HEAR YOU WITH ALL THE KITCHEN NOISES.")

if __name__ == "__main__":
    from sys import argv

    main()
```

Connecting to the server:
```shell
ncat --ssl glazed-kimchi-on-sliced-butter-stij.gpn24.ctf.kitctf.de 443  
Welcome to the Mongolian barbecue.  
Use `sign [hex recipe]` to get your recipe signed by us.  
You can get the public key to verify signatures using `get pkey`.  
When you are ready to taste our freshly made flag, say `flag please`.  
If you would like to leave, just say `check please`.  
  
> get pkey  
x: 3245657005649673346498157697685753014255647504710783129088674024300291821095928242983641600423447724969334064954380850847402292810465243241409643624964609061  
y: 3866588839117987915800979345349478106168359889233080079296551222878251883517452940306420851628028476046037487265357569753744085408035277128827026860873152614  
  
> flag please  
To decide if you are worthy of our special flag you neeed to provide us a signed recipe we don't already know.  
recipe (hex): 
```

The logic error has to do with using `uuid3` inside `secure_random`:
```python
def secure_random(sk: ECC.EccKey, message: bytes) -> int:
    """
    Use sha256 as a secure but deterministic random generator
    to create randomness from the private key and message.

    :param sk: The private signing key
    :param message: The message to sign

    :return: A secure random number
    """

    key_id = uuid3(secure_namespace, sk.export_key(format="PEM")).bytes
    msg_id = uuid3(secure_namespace, message).bytes

    random_generator = sha256(key_id)
    random_generator.update(msg_id)

    return int.from_bytes(random_generator.digest()) % (int(sk._curve.order) - 1) + 1
```

`uuid3` relies on md5 to hash `namespace + message` which is considered cryptographically broken. By using `fastcoll` to find two distinct recipes that result in an MD5 collision with the `kitchenexplosion` prefix, you can force the server to reuse the same nonce (`k`) for two different messages. 

Solve script:
```python
#!/usr/bin/env python3
from pwn import *
import subprocess
import os
import hashlib
from Crypto.PublicKey import ECC

# generate MD5 collision with prefix "kitchenexplosion"
prefix = b"kitchenexplosion"
with open("prefix.bin", "wb") as f:
    f.write(prefix)

print("Generating MD5 collision with fastcoll...")
subprocess.run([
    "docker", "run", "--rm",
    "-v", f"{os.getcwd()}:/work",
    "-w", "/work",
    "docker.io/brimstone/fastcoll",
    "--prefixfile", "prefix.bin",
    "-o", "msg1.bin", "msg2.bin"
], check=True)

with open("msg1.bin", "rb") as f:
    data1 = f.read()
with open("msg2.bin", "rb") as f:
    data2 = f.read()

# extract the two colliding recipes
recipe1 = data1[len(prefix):]
recipe2 = data2[len(prefix):]

os.remove("prefix.bin")
os.remove("msg1.bin")
os.remove("msg2.bin")

# connect to server and get signatures
r = remote('slow-roasted-foie-gras-marinated-in-roasted-creme-fra-che-jjai.gpn24.ctf.kitctf.de', 443, ssl=True)
r.recvuntil(b'> ')

def sign_recipe(recipe):
    # send command
    r.sendline(f'sign {recipe.hex()}'.encode())
    
    r.recvuntil(b's1: ')
    s1_val = int(r.recvline().strip(), 16)
    
    r.recvuntil(b's2: ')
    s2_val = int(r.recvline().strip(), 16)
    
    r.recvuntil(b'> ')
    return s1_val, s2_val

print("Requesting signature for recipe 1...")
r1, s1 = sign_recipe(recipe1)

print("Requesting signature for recipe 2...")
r2, s2 = sign_recipe(recipe2)

assert r1 == r2, "Collision failed: r values differ!"
r_val = r1

# recover private key
print("Recovering private key...")
curve = ECC._curves['p521']
n = int(curve.order)

z1 = int.from_bytes(hashlib.sha256(recipe1).digest(), 'big')
z2 = int.from_bytes(hashlib.sha256(recipe2).digest(), 'big')

diff_s = (s1 - s2) % n
diff_z = (z1 - z2) % n
k = diff_z * pow(diff_s, -1, n) % n
d = ((s1 * k - z1) % n) * pow(r_val, -1, n) % n

print(f"Recovered private key d: {d}")

# forge sig for a new message
print("Forging signature...")
# construct the ECC key using the recovered private key
sk = ECC.construct(curve='p521', d=d)

message = b"give_me_the_flag_please"
e = int.from_bytes(hashlib.sha256(message).digest(), 'big')
z = e & ~(1 << n.bit_length())

# now you can pick any arbitrary valid nonce; k_new = 1 makes it easier
k_new = 1
P = k_new * sk._curve.G
r_new = int(P.x) % n
s_new = (pow(k_new, -1, n) * (z + r_new * d)) % n

# submit and get flag
print("Submitting forged signature to the server...")
r.sendline(b'flag please')
r.recvuntil(b'recipe (hex): ')
r.sendline(message.hex().encode())
r.recvuntil(b's1 (hex): ')
r.sendline(hex(r_new)[2:].encode())
r.recvuntil(b's2 (hex): ')
r.sendline(hex(s_new)[2:].encode())

print("Server Response:")
print(r.recvall().decode())
```

Output:
```shell
python3 solve.py  
Generating MD5 collision with fastcoll...  
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.  
ERRO[0000] User-selected graph driver "overlay" overwritten by graph driver "vfs" from database - delete libpod local files ("/home/user/.local/share/containers/storage") to resolve.  May prevent use of images created by other tools  
MD5 collision generator v1.5  
by Marc Stevens (http://www.win.tue.nl/hashclash/)  
  
Using output filenames: 'msg1.bin' and 'msg2.bin'  
Using prefixfile: 'prefix.bin'  
Using initial value: 0d2f1c9ba85986b69e1cd13a7359534d  
  
Generating first block: ......  
Generating second block: W......  
Running time: 0.567211 s  
[+] Opening connection to slow-roasted-foie-gras-marinated-in-roasted-creme-fra-che-jjai.gpn24.ctf.kitctf.de on port 443: Done  
Requesting signature for recipe 1...  
Requesting signature for recipe 2...  
Recovering private key...  
Recovered private key d: 5460587366942485309332639199203524758572493902730966765267042978009152632005381844246000303641796892690038281915342990825209596761963824800109315478310328363  
Forging signature...  
Submitting forged signature to the server...  
Server Response:  
[+] Receiving all data: Done (90B)  
[*] Closed connection to slow-roasted-foie-gras-marinated-in-roasted-creme-fra-che-jjai.gpn24.ctf.kitctf.de port 443  
Congratulations. Here is your flag: GPNCTF{MaY8e WE sHou1D hAvE hiR3d a pRof35sIonA1?}  
  
>
```

`GPNCTF{MaY8e WE sHou1D hAvE hiR3d a pRof35sIonA1?}`
