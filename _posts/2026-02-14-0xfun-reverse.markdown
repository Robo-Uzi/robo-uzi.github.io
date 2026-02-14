---
layout: post
title:  "Reverse Challenges"
date:   2026-02-14 16:40:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /0xfun-2026-reverse-challenges/
---
* TOC
{:toc}

## Guess The Seed

**Category**: rev

**Author**: Rick 2>/dev/null

**Description**: I've created the ultimate number guessing game! No body can guess my completely unpredictable numbers. If you can somehow beat these mathematical odds and guess all 5 numbers correctly. I'll give you the flag. Do you think you can predict the unpredictable?

I get the challenge file:
```shell
file guess_the_seed  
guess_the_seed: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, stripped
```

I run the program and see this:
```shell
./guess_the_seed  
╔════════════════════════════════════════════════╗  
║         GUESS THE SEED CHALLENGE             ║  
║                                                ║  
║   Can you predict the future? Or the past?     ║  
╚════════════════════════════════════════════════╝  
  
I've generated 5 random numbers using a secret seed...  
If you can guess all 5 numbers correctly, you'll get the flag!  
  
 Hint: The seed was generated at the start of this program.  
 Hint: Time is of the essence... or was it?  
  
Enter your 5 guesses (space-separated):
```

I put the binary in ghidra and find this function:
```shell
undefined4 FUN_00101230(void)

{
  int iVar1;
  int iVar2;
  int iVar3;
  int iVar4;
  int iVar5;
  int iVar6;
  time_t tVar7;
  undefined4 uVar8;
  char *__format;
  uint local_44;
  uint local_40;
  uint local_3c;
  uint local_38;
  uint local_34;
  
  FUN_00101180();
  tVar7 = time((time_t *)0x0);
  srand((uint)tVar7);
  iVar1 = rand();
  iVar2 = rand();
  iVar3 = rand();
  iVar4 = rand();
  iVar5 = rand();
  FUN_00101830(&DAT_00104b0a,&DAT_0010431c);
  printf(&DAT_00104b0a);
  FUN_001018b0(&DAT_00104b43,&DAT_00104381);
  printf(&DAT_00104b43);
  FUN_00101930(&DAT_00104b85,&DAT_001043f2);
  printf(&DAT_00104b85);
  FUN_001019d0(&DAT_00104bc7,&DAT_00104466);
  printf(&DAT_00104bc7);
  FUN_00101a50(&DAT_00104bfa,&DAT_001044c3);
  printf(&DAT_00104bfa);
  FUN_00101ab0(&DAT_00104c24,&DAT_00104512);
  iVar6 = __isoc99_scanf(&DAT_00104c24,&local_34,&local_38,&local_3c,&local_40,&local_44);
  if (iVar6 == 5) {
    if ((((local_34 == iVar1 % 1000) && (local_38 == iVar2 % 1000)) && (local_3c == iVar3 % 1000))
       && ((local_40 == iVar4 % 1000 && (local_44 == iVar5 % 1000)))) {
      FUN_00101ba0(&DAT_00104c55,&DAT_001045a0);
      uVar8 = 0;
      printf(&DAT_00104c55);
      FUN_00101c30(&DAT_00104ce1,&DAT_00104658);
      printf(&DAT_00104ce1);
      FUN_00101cb0(&DAT_00104d10,&DAT_001046ba);
      printf(&DAT_00104d10);
      __format = &DAT_00104d9c;
      FUN_00101d30(&DAT_00104d9c,&DAT_0010476d);
    }
    else {
      FUN_00101db0(&DAT_00104dd4,&DAT_001047da);
      uVar8 = 0;
      printf(&DAT_00104dd4,(ulong)(uint)(iVar1 % 1000),(ulong)(uint)(iVar2 % 1000),
             (ulong)(uint)(iVar3 % 1000),(ulong)(uint)(iVar4 % 1000),(ulong)(uint)(iVar5 % 1000));
      FUN_00101e40(&DAT_00104e07,&DAT_00104840);
      printf(&DAT_00104e07);
      FUN_00101ea0(&DAT_00104e24,&DAT_00104893);
      printf(&DAT_00104e24);
      __format = &DAT_00104e5b;
      FUN_00101f20(&DAT_00104e5b,&DAT_001048ed);
    }
  }
  else {
    __format = &DAT_00104c34;
    FUN_00101b10(&DAT_00104c34,&DAT_00104553);
    uVar8 = 1;
  }
  printf(__format);
  return uVar8;
}
```
So the seed is based on the current UNIX timestamp when the program starts. If I start the program and compute the same seed locally within plus or minus 1-2 seconds I should get the same numbers. This works because `rand()` in glibc is deterministic.

Solve script:
```python
#!/usr/bin/env python3
import time
import ctypes
import subprocess

BINARY = "./guess_the_seed"
libc = ctypes.CDLL("libc.so.6")
seed = int(time.time())
libc.srand(seed)

# generate the 5 numbers
nums = [libc.rand() % 1000 for _ in range(5)]
guess_line = " ".join(map(str, nums))

print(f"Using seed: {seed}")
print(f"Guessing: {guess_line}")

proc = subprocess.Popen(
    [BINARY],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True
)

output, _ = proc.communicate(guess_line + "\n")

print(output)
```

Solve script output:
```shell
python3 solve5.py  
Using seed: 1771015530  
Guessing: 572 580 118 542 894  
╔════════════════════════════════════════════════╗  
║         GUESS THE SEED CHALLENGE            ║  
║                                                ║  
║   Can you predict the future? Or the past?     ║  
╚════════════════════════════════════════════════╝  
  
I've generated 5 random numbers using a secret seed...  
If you can guess all 5 numbers correctly, you'll get the flag!  
  
 Hint: The seed was generated at the start of this program.  
 Hint: Time is of the essence... or was it?  
  
Enter your 5 guesses (space-separated):  
 ══════════════════════════════════════════   
CONGRATULATIONS! You've cracked the code!  
 ══════════════════════════════════════════   
  
 FLAG: 0xfun{W3l1_7h4t_w4S_Fun_4235328752619125}
```

`0xfun{W3l1_7h4t_w4S_Fun_4235328752619125}`

___

## Nanom-dinam???itee?

**Category**: rev

**Author**: call4pwn

**Description**: Don't trust what you see, trust what happens when no one is looking.

I get the challenge file:
```shell
file nanom-dinam-ite__iteee  
nanom-dinam-ite__iteee: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=54b8d730ae50b53670b611e1afa86da782cf6006, for GNU/Linux 3.2.0, stripped
```

Running the binary I see this:
```shell
./nanom-dinam-ite__iteee  
Pass: password  
Oh sure, here is your flag: 0xfun{1_10v3_M1LF}
```
It prints a dummy flag.

I put the binary in ghidra and see this:
```shell
/* WARNING: Removing unreachable block (ram,0x00101494) */

void FUN_0010131b(void)

{
  long lVar1;
  char *pcVar2;
  size_t sVar3;
  long in_FS_OFFSET;
  undefined4 local_8f;
  undefined4 uStack_8b;
  undefined8 local_20;
  
  local_20 = *(undefined8 *)(in_FS_OFFSET + 0x28);
  lVar1 = ptrace(PTRACE_TRACEME,0,0,0);
  if (lVar1 < 0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  raise(0x13);
  memset((void *)((long)&uStack_8b + 3),0,100);
  local_8f = 0x73736150;
  uStack_8b = CONCAT13(uStack_8b._3_1_,0x203a);
  write(1,&local_8f,6);
  pcVar2 = fgets((char *)((long)&uStack_8b + 3),100,stdin);
  if (pcVar2 != (char *)0x0) {
    sVar3 = strcspn((char *)((long)&uStack_8b + 3),"\n");
    *(undefined *)((long)&uStack_8b + sVar3 + 3) = 0;
    sVar3 = strlen((char *)((long)&uStack_8b + 3));
    if (sVar3 == 0x28) {
      FUN_001012a9((long)&uStack_8b + 3,1,0xcbf29ce484222325);
      do {
        invalidInstructionException();
      } while( true );
    }
    puts("Oh sure, here is your flag: 0xfun{1_10v3_M1LF}");
                    /* WARNING: Subroutine does not return */
    exit(99);
  }
                    /* WARNING: Subroutine does not return */
  exit(1);
}
```
The child allows itself to be traced, then stops. 

I also see this function:
```shell
void FUN_001014ad(uint param_1)

{
  long lVar1;
  long *plVar2;
  long *plVar3;
  long in_FS_OFFSET;
  uint local_248;
  int local_244;
  long local_240;
  undefined local_238 [40];
  undefined8 local_210;
  long local_1e8;
  long local_1b8;
  long local_158 [41];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  plVar2 = &DAT_001020a0;
  plVar3 = local_158;
  for (lVar1 = 0x28; lVar1 != 0; lVar1 = lVar1 + -1) {
    *plVar3 = *plVar2;
    plVar2 = plVar2 + 1;
    plVar3 = plVar3 + 1;
  }
  waitpid(param_1,(int *)&local_248,0);
  while( true ) {
    while( true ) {
      if ((local_248 & 0xff) != 0x7f) {
        if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
          __stack_chk_fail();
        }
        return;
      }
      if ((local_248 & 0xff00) == 0x400) break;
      ptrace(PTRACE_CONT,(ulong)param_1,0,(ulong)((int)local_248 >> 8 & 0xff));
      waitpid(param_1,(int *)&local_248,0);
    }
    ptrace(PTRACE_GETREGS,(ulong)param_1,0,local_238);
    local_240 = local_1e8;
    local_244 = (int)local_210;
    if ((local_244 < 0) || (0x27 < local_244)) break;
    if (local_1e8 != local_158[local_244]) {
      ptrace(PTRACE_KILL,(ulong)param_1,0,0);
                    /* WARNING: Subroutine does not return */
      exit(1);
    }
    local_1b8 = local_1b8 + 2;
    ptrace(PTRACE_SETREGS,(ulong)param_1,0,local_238);
    ptrace(PTRACE_CONT,(ulong)param_1,0,0);
    waitpid(param_1,(int *)&local_248,0);
  }
  ptrace(PTRACE_KILL,(ulong)param_1,0,0);
                    /* WARNING: Subroutine does not return */
  exit(1);
}
```

In this function I see the programs forks and the child is being debugged by the parent. In other words, the child asks for input, the parent debugs the child, and the parent checks every executed instruction:
```shell
undefined8 FUN_001016cd(void)

{
  __pid_t _Var1;
  
  _Var1 = fork();
  if (_Var1 == 0) {
    FUN_0010131b();
  }
  else {
    FUN_001014ad(_Var1);
  }
  return 0;
}
```

The hash function:
```shell
ulong FUN_001012a9(long param_1,ulong param_2,ulong param_3)

{
  ulong local_18;
  ulong local_10;
  
  local_18 = param_3;
  for (local_10 = 0; local_10 < param_2; local_10 = local_10 + 1) {
    local_18 = (local_18 ^ *(byte *)(local_10 + param_1)) * 0x100000001b3;
    local_18 = local_18 ^ local_18 >> 0x20;
  }
  return local_18;
}
```
I find out this is FNV-1a 64 bit.

The solution script should extract 40 expected 64 bit hash values from `.rodata`. Then reimplement FNV-1a and bruteforce one character at a time (it checks `local_158[index]`. Each expected value represents the cumulative FNV-1a hash of the flag prefix up to that index.). Solve script:
```python
#!/usr/bin/env python3
import sys
import struct

FNV_OFFSET = 0xcbf29ce484222325
FNV_PRIME  = 0x100000001b3

def fnv1a(data, seed=FNV_OFFSET):
    h = seed
    for b in data:
        h ^= b
        h = (h * FNV_PRIME) & 0xffffffffffffffff
        h ^= (h >> 32)
    return h

def extract_hashes(path, offset=0x20a0, count=40):
    hashes = []
    with open(path, "rb") as f:
        f.seek(offset)
        for _ in range(count):
            hashes.append(struct.unpack("<Q", f.read(8))[0])
    return hashes

def solve(binary_path):
    expected = extract_hashes(binary_path)

    flag = b""
    for i in range(len(expected)):
        for c in range(32, 127):  # printable ASCII
            candidate = flag + bytes([c])
            if fnv1a(candidate) == expected[i]:
                flag = candidate
                print(f"Found char {i}: {chr(c)}")
                break
        else:
            raise Exception(f"Failed at position {i}")

    return flag.decode()

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <binary>")
        sys.exit(1)

    print("\nFlag:", solve(sys.argv[1]))
```

`0xfun{unr3adabl3_c0d3_is_s3cur3_c0d3_XD}`

___

## pingpong

**Category**: rev

**Author**: blknova

**Description**: ONLY attempt if you love table tennis... you have been warned.

I get the challenge file:
```shell
file pingpong  
pingpong: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=a2e053370553c495a52d4080e20de94cf5d93985, stripped
```

Running it doesnt display anything:
```shell
./pingpong
```

Through `strace` I see it is listening on localhost on port `9768`:
```shell
bind(3, {sa_family=AF_INET, sin_port=htons(9768), sin_addr=inet_addr("127.0.0.1")}, 16) = 0
```

Its waiting for UDP packets. When I run it and connect with `nc` I see this:
```shell
nc -u 127.0.0.1 9768  
pingpong

./pingpong  
IP Blocked!!! A personal message to 127.0.0.1:53583:  
If you come to my table, you are going to get served.
```
It says it blocks my connection. 

Opening the binary in ghidra I spot this. This one is written in rust so there was a ton of functions to look through:
```shell
  FUN_00118950(local_508,
               "0149545b5f4b5d1e5c545d1a55036c5700404b46505d426e02001b4909030957414a7b7a48Invalid he x string.:112.105.110.103112.111.110.103IP Blocked!!! A personal message to :\nIf you  come to my table, you are going to get served.\nNow you\'re pinging the pong!\n"
               ,0x4a);
```
The hex string is the encrypted flag. The XOR key is the "IP addresses" concatenated: `112.105.110.103112.111.110.103`. 

In ghidra I see the XOR instructions here:
```shell
              local_430 = (undefined8 **)
                          CONCAT71(local_430._1_7_,(byte)local_430 ^ (byte)*local_528);
              if (local_530 != (undefined8 *)0x0) {
                free(local_528);
              }
              uVar11 = 1;
              uVar12 = 0;
              do {
                local_530 = (undefined8 *)0x0;
                local_528 = (code *)&DAT_00000001;
                local_520 = 0;
                local_508 = (undefined8 **)0xe0000020;
                local_518 = (undefined **)&local_530;
                uStack_510 = &PTR_FUN_0015f8b0;
                    /* try { // try from 00119689 to 00119693 has its CatchHandler @ 00119a19 */
                cVar5 = FUN_001579b0((char *)((long)local_540 + uVar12 * 0x11),&local_518);
                if (cVar5 != '\0') goto LAB_001199a5;
                if ((uVar11 | uVar4) >> 0x20 == 0) {
                  uVar7 = (uVar11 & 0xffffffff) % (uVar4 & 0xffffffff);
                }
                else {
                  uVar7 = uVar11 % uVar4;
                }
                if (local_520 <= uVar7) goto LAB_0011987b;
                *(byte *)((long)&local_430 + uVar11) =
                     *(byte *)((long)&local_430 + uVar11) ^ (byte)local_528[uVar7];
                if (local_530 != (undefined8 *)0x0) {
                  free(local_528);
                }
                if (uVar7 == 0) {
                  uVar12 = (ulong)(~(uint)uVar12 & 1);
                }
                uVar11 = uVar11 + 1;
              } while (uVar11 != 0x25);
```

Solve script:
```python
#!/usr/bin/env python3

encrypted_hex = "0149545b5f4b5d1e5c545d1a55036c5700404b46505d426e02001b4909030957414a7b7a48"
encrypted = bytes.fromhex(encrypted_hex)

print(f"\nEncrypted data: {encrypted_hex}")
print(f"Length: {len(encrypted)} bytes\n")

key = b"112.105.110.103112.111.110.103"

# Decrypt using repeating XOR
flag = bytearray()
for i, byte in enumerate(encrypted):
    key_byte = key[i % len(key)]
    flag.append(byte ^ key_byte)

print(f"Flag: {bytes(flag).decode('ascii')}")
```

Solve script output:
```shell
python3 solve6.py  
  
Encrypted data: 0149545b5f4b5d1e5c545d1a55036c5700404b46505d426e02001b4909030957414a7b7a48  
Length: 37 bytes  
  
Flag: 0xfun{h0mem4d3_f1rewall_305x908fsdJJ}
```

`0xfun{h0mem4d3_f1rewall_305x908fsdJJ}`

___
