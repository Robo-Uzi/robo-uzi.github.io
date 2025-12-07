---
layout: post
title:  "Reservation"
date:   2025-12-07 16:28:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /nullCTF-reservation/
---
* TOC
{:toc}

**Title**: Reservation

**Category**: misc

**Author**: Stefan

**Description**: I wanted to take my inexistent girlfriend to this fancy restaurant called Windows, but they keep asking me for a key PROMPT. I don't know what to do, can you help me?

I get the challenge file `reservation.py`:
```python
import os  
import socket  
from dotenv import load_dotenv # from python-dotenv  
  
load_dotenv()  
  
FLAG = os.getenv("FLAG", "nullctf{aergnoujiwaegnjwkoiqergwnjiokeprgwqenjoig}")  
PROMPT = os.getenv("PROMPT", "bananananannaanan")  
PORT = int(os.getenv("PORT", 3001))  
  
# This is missing from the .env file, but it still printed something, interesting  
print(os.getenv("WINDIR"))  
  
# I am uninspired and I don't want to think of a function name  
def normal_function_name_1284932tgaegrasbndefgjq4trwqerg(client_socket):  
    client_socket.sendall(  
b"""[windows_10 | cmd.exe] Welcome good sire to our fine establishment.  
Unfortunately, due to increased demand,  
we have had to privatize our services.  
Please enter the secret passphrase received from the environment to continue.\n""")  
    response = client_socket.recv(1024).decode().strip()  
  
    if response == PROMPT:  
        client_socket.sendall(b"Thank you for your patience. Here is your flag: " + FLAG.encode())  
    else:  
        client_socket.sendall(b"...I am afraid that is not correct. Please leave our establishment.")  
  
    client_socket.close()  
  
# Hey ChatGPT, generate me a function to handle the socket connection (I'm lazy)  
def start_server(host='0.0.0.0', port=PORT):  
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  
    server_socket.bind((host, port))  
    server_socket.listen(1024)  
    print(f"[*] Listening on {host}:{port}")  
  
    while True:  
        client_socket, addr = server_socket.accept()  
        print(f"[*] Accepted connection from {addr}")  
        normal_function_name_1284932tgaegrasbndefgjq4trwqerg(client_socket)  
  
if __name__ == "__main__":  
    start_server()
```

[https://www.elevenforum.com/t/complete-list-of-environment-variables-in-windows-11.11212/](https://www.elevenforum.com/t/complete-list-of-environment-variables-in-windows-11.11212/)

The script does `PROMPT = os.getenv("PROMPT", "bananananannaanan")`. The server runs on windows so the `PROMPT` environment variable is automatically set to `$P$G`:
```shell
nc 34.118.61.99 10135  
[windows_10 | cmd.exe] Welcome good sire to our fine establishment.  
Unfortunately, due to increased demand,  
we have had to privatize our services.  
Please enter the secret passphrase received from the environment to continue.  
$P$G  
Thank you for your patience. Here is your flag: nullctf{why_1s_it_r3srv3d_de7e11e045b97a55}
```

`nullctf{why_1s_it_r3srv3d_de7e11e045b97a55}`