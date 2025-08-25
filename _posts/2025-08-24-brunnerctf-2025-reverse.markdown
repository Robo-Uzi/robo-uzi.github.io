---
layout: post
title:  "BrunnerCTF 2025 Reverse Engineering Challenges"
date:   2025-08-24 21:48:10 -0400
author: robo.uzi
tags: [CTF, reverse]
permalink: /brunnerctf-2025-reverse/
---
* TOC
{:toc}

## Baker Brian
**Category:** Reverse Engineering

**Difficulty:** Beginner  
**Author:** Nissen
{% raw %}
Baker Brian says he has a plan to make him super rich, but he refuses to share any details 😠 Can you access his Cake Vault where he keeps all his business secrets?

**Info:** The challenge has both a file download and a server to connect to.

I am given this file:
```t
print("""  
               
                                           
               Baker Brian's Cake Vault    
                                           
               
""")  
  
# Make sure nobody else tries to enter my vault  
username = input("Enter Username:\n> ")  
if username != "Br14n_th3_b3st_c4k3_b4k3r":  
   print("❌ Go away, only Baker Brian has access!")  
   exit()  
  
# Password check if anybody guesses my username  
# Naturally complies with all modern standards, nothing weak like "Tr0ub4dor&3"  
password = input("\nEnter password:\n> ")  
  
# Check each word separately  
words = password.split("-")  
  
# Word 1  
if not (  
   len(words) > 0 and  
   words[0] == "red"  
):  
   print("❌ Word 1: Wrong - get out!")  
   exit()  
else:  
   print("✅ Word 1: Correct!")  
  
# Word 2  
if not (  
   len(words) > 1 and  
   words[1][::-1] == "yromem"  
):  
   print("❌ Word 2: Wrong - get out!")  
   exit()  
else:  
   print("✅ Word 2: Correct!")  
  
# Word 3  
if not (  
   len(words) > 2 and  
   len(words[2]) == 5 and  
   words[2][0] == "b" and  
   words[2][1] == "e" and  
   words[2][2:4] == "r" * 2 and  
   words[2][-1] == words[1][-1]  
):  
   print("❌ Word 3: Wrong - get out!")  
   exit()  
else:  
   print("✅ Word 3: Correct!")  
  
# Word 4  
if not (  
   len(words) > 3 and  
   words[3] == words[0][:2] + words[1][:3] + words[2][:3]  
):  
   print("❌ Word 4: Wrong - get out!")  
   exit()  
else:  
   print("✅ Word 4: Correct!")  
  
# Password length  
if len(password) != len(username):  
   print("❌ Wrong password length, get out!")  
   exit()  
  
# Nobody will crack that password, access can be granted  
print("\nWelcome back, Brian! Your vault has been opened:\n")  
with open("cake_vault.txt") as f:  
   print(f.read())
```

```shell
ncat --ssl baker-brian-8936bc67ab34387d.challs.brunnerne.xyz 443  
  
               
                                           
               Baker Brian's Cake Vault    
                                           
               
  
Enter Username:  
> Br14n_th3_b3st_c4k3_b4k3r  
  
Enter password:  
> red-memory-berry-remember  
✅ Word 1: Correct!  
✅ Word 2: Correct!  
✅ Word 3: Correct!  
✅ Word 4: Correct!  
  
Welcome back, Brian! Your vault has been opened:  
  
     
                                                       
     Plan: Based on serious, cutting-edge research:    
        https://pubmed.ncbi.nlm.nih.gov/29956364/      
     start selling cookies to university professors    
       they can give students for better ratings       
                                                       
            Daily XKCD: https://xkcd.com/936/          
                                                       
        Flag: brunner{b4k3r_br14n_w1ll_b3_r1ch!}       
                                                       
   
```
You can see the username hardcoded. 

- `w1 = red`
- `w2[::-1]` = `yromem` > `w2 = memory`
- w3 = `b` `e` `rr` last char = last of w2 (`y`) > `berry`
- w4 = `w1[:2]+w2[:3]+w3[:3]` > `re`+`mem`+`ber` = `remember`  
Full password `red-memory-berry-remember` is length 25, matching the username length check.
{% endraw %}

___

## Rolling Pin
**Category:** Reverse Engineering

**Difficulty:** Beginner  
**Author:** rvsmvs

The head baker's gone rogue and locked up the recipe for the perfect pastry swirl inside a secret code. Can you knead your way through layers of fluffy obfuscation and figure out the exact mix of bytes to make it rise just right?

I get this file:
```shell
file rolling_pin  
rolling_pin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=808d9dac072400d5c61e0fae9cb72c12ee  
a22fe2, for GNU/Linux 3.2.0, with debug_info, not stripped
```

I open the binary in ghidra. Here is the main function:
```decompiled

undefined8 main(void)

{
  char cVar1;
  char *pcVar2;
  long in_FS_OFFSET;
  size_t local_68;
  undefined8 local_60;
  char local_58 [72];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  puts("Roll the dough:");
  pcVar2 = fgets(local_58,0x40,stdin);
  if (pcVar2 != (char *)0x0) {
    local_68 = strlen(local_58);
    if ((local_68 != 0) && (local_58[local_68 - 1] == '\n')) {
      local_68 = local_68 - 1;
    }
    if (local_68 == 0x19) {
      for (local_60 = 0; local_60 < 0x19; local_60 = local_60 + 1) {
        cVar1 = rotl(local_58[local_60],(uint)local_60 & 7);
        if (cVar1 != (&baked)[local_60]) {
          puts("Not quite done yet");
          goto LAB_004012da;
        }
      }
      puts("Good job!");
    }
    else {
      puts("Not quite done yet");
    }
  }
LAB_004012da:
  if (local_10 == *(long *)(in_FS_OFFSET + 0x28)) {
    return 0;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

My final script to get the flag:
```python
from pwn import *

elf = ELF('./rolling_pin')
rodata_section = elf.get_section_by_name('.rodata')
rodata_offset = rodata_section.header.sh_offset
rodata_size   = rodata_section.header.sh_size

with open('./rolling_pin', 'rb') as f:
    f.seek(rodata_offset)
    rodata = f.read(rodata_size)

# Known prefix of baked array
prefix = bytes([0x62, 0xe4, 0xd5, 0x73])

# Find start of baked array
start = rodata.find(prefix)
if start == -1:
    print("Could not find baked array in .rodata")
    exit()

baked = rodata[start:start+25]
print("Baked array bytes:", list(baked))

def rotr(byte, n):
    n %= 8
    return ((byte >> n) | ((byte << (8 - n)) & 0xff)) & 0xff

flag_bytes = [rotr(baked[i], i) for i in range(len(baked))]
flag = bytes(flag_bytes).decode()
print("Flag:", flag)
```
The script loads the binary, it locates the `.rodata` section which contains data like strings and constant arrays (here its the `baked` array) and reads it and extracts the full 25 byte array. The first bytes I knew because I could see them in ghidra. Each byte of the `baked` array was rotated left by its index in the original program. To recover the original, rotate right the same amount. 

`brunner{r0t4t3_th3_d0ugh}`

___

## Grandma's Predictable Cookies
**Category:** Reverse Engineering

**Difficulty:** Easy-Medium  
**Author:** H4N5

Grandma encrypted her secret cookie recipe using her "special ingredient" a random number generator seeded with the exact time she baked it. She thought it was uncrackable. But little did she know: Using the same oven clock every time makes your cookies easy to reverse-engineer. Can you recover her delicious secret?

I an given `flag.enc`, `flag.txt`, and `brunner`:
```shell
cat flag.enc  
Encrypted flag: 3ec63cc41f1ac1980651726ab3ce2948882b879c19671269963e39103c83ebd6ef173d60c76ee5  
Encryption time (approx): 1755860000

cat flag.txt  
brunner{REDACTED}

file brunner  
brunner: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=5921cf888d3426176894905d8f1a04fda5f8f4  
51, for GNU/Linux 3.2.0, with debug_info, not stripped

./brunner  
Encrypted flag: fdf1e61aa69cdb74e145534d70ceae39c0d9  
Encryption time (approx): 1755970000
```

I open the binary in ghidra. The main function looks like this:
```decompile
undefined8 main(void)

{
  byte bVar1;
  long lVar2;
  int iVar3;
  undefined8 uVar4;
  ulong uVar5;
  undefined8 uStack_170;
  byte local_168 [260];
  int local_64;
  byte *local_60;
  long local_58;
  uint local_4c;
  long local_48;
  size_t local_40;
  FILE *local_38;
  ulong local_30;
  ulong local_28;
  int local_1c;
  
  uStack_170 = 0x4012a8;
  local_38 = fopen("flag.txt","rb");
  if (local_38 == (FILE *)0x0) {
    uStack_170 = 0x4012bd;
    perror("Error opening flag.txt");
    uVar4 = 1;
  }
  else {
    uStack_170 = 0x4012e7;
    local_40 = fread(local_168,1,0xff,local_38);
    uStack_170 = 0x4012f7;
    fclose(local_38);
    local_168[local_40] = 0;
    uStack_170 = 0x401312;
    local_48 = get_current_time_danish();
    local_4c = (int)(local_48 / 10000) * 10000;
    uStack_170 = 0x40134e;
    srand((uint)local_48);
    for (local_1c = 0; local_1c < 1000; local_1c = local_1c + 1) {
      uStack_170 = 0x40135c;
      rand();
    }
    local_58 = local_40 - 1;
    lVar2 = ((local_40 + 0xf) / 0x10) * -0x10;
    local_60 = local_168 + lVar2;
    for (local_28 = 0; local_28 < local_40; local_28 = local_28 + 1) {
      *(undefined8 *)(local_168 + lVar2 + -8) = 0x4013ae;
      iVar3 = rand();
      local_64 = iVar3 % 0x100;
      local_60[local_28] = local_168[local_28] ^ (byte)(iVar3 % 0x100);
    }
    *(undefined8 *)(local_168 + lVar2 + -8) = 0x401405;
    printf("Encrypted flag: ");
    for (local_30 = 0; local_30 < local_40; local_30 = local_30 + 1) {
      bVar1 = local_60[local_30];
      *(undefined8 *)(local_168 + lVar2 + -8) = 0x401431;
      printf("%02x",(ulong)bVar1);
    }
    uVar5 = (ulong)local_4c;
    *(undefined8 *)(local_168 + lVar2 + -8) = 0x401454;
    printf("\nEncryption time (approx): %ld\n",uVar5);
    uVar4 = 0;
  }
  return uVar4;
}
```
- It reads `flag.txt` into `local_168`.
- Gets the time from `get_current_time_danish()`, then rounds it: `local_4c = (int)(local_48 / 10000) * 10000;`
- Seeds `srand()` with that time: `srand((uint)local_48);`
- Calls `rand()` 1000 times to advance the PRNG.
- Encrypts each byte of the flag with: `local_60[local_28] = local_168[local_28] ^ (byte)(rand() % 0x100);`
- Prints the encrypted bytes in hex and the encryption time `local_4c`.

Now I understand how the main function works. I know `flag.enc` and the timestamp `1755860000`. `srand()` is seeded with a guessable value. `rand()` is used in the C standard library PRNG which is deterministic. The XOR is reversible: `flag_byte = encrypted_byte ^ rand_byte` 

I make this python script to get the flag:
```python
import ctypes

libc = ctypes.CDLL("libc.so.6")
enc_hex = "3ec63cc41f1ac1980651726ab3ce2948882b879c19671269963e39103c83ebd6ef173d60c76ee5"
enc_bytes = bytes.fromhex(enc_hex)
approx_time = 1755860000

for seed in range(approx_time - 10000, approx_time + 10000):
    libc.srand(seed)
    for _ in range(1000):
        libc.rand()
    decrypted = bytes([b ^ (libc.rand() % 256) for b in enc_bytes])
    if b"brunner{" in decrypted:
        print("Seed:", seed)
        print("Flag:", decrypted.decode())
        break
```
`brunner{t1me_wr4p_prng_1s_pred1ct4ble}`

___

## Trippi Troppa Chaos
**Category:** Reverse Engineering
**Difficulty:** Medium  
**Author:** ha1fdan

The baker got infected with Italian brainrot and obfuscated our flag encoder with memes and nested functions. Can you help us find our flag?

I get two files. `output.txt` which contains: 
```
qjuA_QZVI_ua24NQ}fM1hX4ecdyVShKb2vJjeQJ@Jz=zws0^9Enr1fR+Em_5w2j=p4)2<#m3EZ?m3Oo@
```

And `trippa_troppa_sus.py` which is fairly long and contains brainrot words that serve as obfuscation. Every word...

The script does these operations in order on the bytes of `flag.txt` and prints the result as a string:
1. Derives a key: `K = SHA256("skibidi" + "skibidi").digest()[:len("skibidi")]` (7 bytes).
2. XOR the flag bytes with the repeating key `K`.
3. Multiply every resulting byte by `7` modulo `256` (i.e. `(b*7) % 256` for each byte).
4. Reverse the byte sequence.
5. Base85 encode the reversed bytes and print the resulting text.

This python script gets the flag by running the operations in reverse:
```python
import base64, hashlib, itertools  
  
s = "qjuA_QZVI_ua24NQ}fM1hX4ecdyVShKb2vJjeQJ@Jz=zws0^9Enr1fR+Em_5w2j=p4)2<#m3EZ?m3Oo@"  
  
decoded = base64.b85decode(s)  
reversed_bytes = decoded[::-1]  
inv7 = pow(7, -1, 256)  
step_c = bytes((b * inv7) % 256 for b in reversed_bytes)  
  
key = b"skibidi"  
K = hashlib.sha256((key.decode()+key.decode()).encode()).digest()[:len(key)]  
flag_bytes = bytes(a ^ b for a, b in zip(step_c, itertools.cycle(K)))  
  
flag = flag_bytes.decode('utf-8')  
print(flag)
```
`brunner{tr4l4l3r0_b0mb4rd1r0_r3v3rs3_3ng1n33r1ng_sk1b1d1_m4st3r}`

___

