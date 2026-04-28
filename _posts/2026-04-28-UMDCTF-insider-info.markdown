---
layout: post
title:  "insider-info"
date:   2026-04-28 14:24:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMDCTF-2026-insider-info/
---

**Category**: Misc

**Author**: cephi-sui

**Description**: What better way to hide inside info on the Internet? `nc challs.umdctf.io 32323`

I get the challenge file `dns_server.py`:
```python
#!/usr/local/bin/python -u

import time, sys, socket, random, string
from dnslib.server import DNSServer, DNSLogger, BaseResolver
from dnslib import RR, TXT, QTYPE, RCODE
from flag import flag

k = 819
secret = random.choices(string.ascii_letters, k=k)
subdomain = '.'.join([''.join(secret[i:i+63]) for i in range(0, k - 63 + 1, 63)])

class Resolver(BaseResolver):
    def resolve(self, request, handler):
        reply = request.reply()

        for question in request.questions:
            qname = question.qname
            if question.qtype != QTYPE.TXT:
                reply.header.rcode = RCODE.reverse['NXDOMAIN']
                continue

            if qname == subdomain + ".inside.info":
                reply.questions = []
                reply.add_answer(RR("flag.inside.info", QTYPE.TXT, rdata=TXT(flag)))
            elif qname.matchWildcard("*.inside.info"):
                try:
                    index = int(qname.label[0])
                    reply.add_answer(RR(qname, QTYPE.TXT, rdata=TXT(secret[index])))
                except ValueError, IndexError:
                    reply.header.rcode = RCODE.reverse['NXDOMAIN']
                    reply.questions = []
            else:
                reply.header.rcode = RCODE.reverse['NXDOMAIN']

        return reply

server = DNSServer(Resolver(),
                   port=1337,
                   logger=DNSLogger(logf=DNSLogger.log_pass))
server.start_thread()

for _ in range(2):
    s = sys.stdin.buffer.read(2)
    s = int.from_bytes(s)
    q = sys.stdin.buffer.read(s)

    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as sock:
        sock.sendto(q, ("127.0.0.1", 1337))
        a = sock.recv(2**16)
        a = len(a).to_bytes(2) + a
        sys.stdout.buffer.write(a)
```

A random 819 character secret is created from ascii letters: 
```python
k = 819
random.choices(string.ascii_letters, k=k)
```

The subdomain is built like this:
```shell
range(0, k - 63 + 1, 63) = range(0, 757, 63) = [0, 63, 126, 189, 252, 315, 378, 441, 504, 567, 630, 693, 756]
```
It will be 13 chunks of 63 characters (`chunk0.chunk1...chunk12`) each covering indices `0-818`. 

The server resolves characters by index (their `qname`):
```python
index = int(qname.label[0])
reply.add_answer(RR(qname, QTYPE.TXT, rdata=TXT(secret[index])))
```

The final flag is behind the exact domain `subdomain.inside.info`.

The server accepts 2 DNS queries total:
```python
for _ in range(2):
    s = sys.stdin.buffer.read(2)
    s = int.from_bytes(s)
    q = sys.stdin.buffer.read(s)
```
The first should get the secret (getting all 819 characters in one DNS packet), and the second will query for the flag.

Recap:

Query 1: A single DNS packet containing 819 questions for each index `0.inside.info-818.inside.info`. The resolver will process all questions and reply with 819 TXT answers. Each answer is one character of the secret. Then parse the answers by looking at the `qname` of each to map the index to the character. That is the full secret.

Query 2: Split the secret into 13 chunks of 63 characters and join with dots. Then make a raw DNS TXT query for `subdomain.inside.info`. Since the full domain name is 843 bytes (exceeds 255 byte limit) I need to manually build the packet and encode each label with its length prefix. The `dnslib` library was rejecting it but the server will accept it. After sending the second query it should return the flag.

Solve script:
```python
#!/usr/bin/env python3
import socket
import struct
from dnslib import DNSRecord, DNSHeader, DNSQuestion, QTYPE

HOST = "challs.umdctf.io"
PORT = 32323
SECRET_LEN = 819
CHUNK_SIZE = 63

def recvall(s: socket.socket, n: int) -> bytes:
    # reliably receive exactly n bytes from socket
    buf = b""
    while len(buf) < n:
        chunk = s.recv(n - len(buf))
        if not chunk:
            raise RuntimeError(f"Connection closed early: got {len(buf)}/{n} bytes")
        buf += chunk
    return buf

def send_query(s: socket.socket, query_data: bytes) -> bytes:
    # send a length prefixed DNS query and return the length prefixed response
    s.sendall(struct.pack(">H", len(query_data)) + query_data)
    resp_len = struct.unpack(">H", recvall(s, 2))[0]
    return recvall(s, resp_len)

def build_secret_query() -> bytes:
    # build one DNS packet with 819 questions. one per secret index
    # the server handles multiple questions per packet and returns one TXT
    # answer per question with the character at that index.
    hdr = DNSHeader(id=0x1234, qr=0, opcode=0, aa=0, tc=0, rd=1, ra=0, z=0, rcode=0)
    dns = DNSRecord(header=hdr)
    for i in range(SECRET_LEN):
        dns.add_question(DNSQuestion(f"{i}.inside.info", QTYPE.TXT))
    return dns.pack()

def build_flag_query(subdomain: str) -> bytes:
    # build a raw DNS TXT query for  <subdomain>.inside.info
    # the full name is 13 labels x 64 bytes + TLD which exceeds the DNS 255-byte spec limit 
    # so build it manually rather than going through dnslib
    full_domain = subdomain + ".inside.info"
    # DNS header: ID=0x5678, QR=0 (query), RD=1, QDCOUNT=1
    header = struct.pack(">HHHHHH", 0x5678, 0x0100, 1, 0, 0, 0)
    qname = b""
    for label in full_domain.split("."):
        encoded = label.encode("ascii")
        if len(encoded) > 63:
            raise ValueError(f"Label too long ({len(encoded)} bytes): {label!r}")
        qname += bytes([len(encoded)]) + encoded
    qname += b"\x00"                    # root label
    qtype  = struct.pack(">H", 16)      # TXT  = 0x0010
    qclass = struct.pack(">H", 1)       # IN   = 0x0001
    return header + qname + qtype + qclass

def main():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((HOST, PORT))
        print(f"Connected to {HOST}:{PORT}")

        # get all 819 secret characters
        print("Sending query 1: 819 individual character lookups...")
        q1 = build_secret_query()
        r1 = send_query(s, q1)
        print(f"Response 1: {len(r1)} bytes")

        # parse the answers. use QNAME to map index to character (order safe)
        dns1 = DNSRecord.parse(r1)
        char_map: dict[int, str] = {}
        for rr in dns1.rr:
            if rr.rtype == QTYPE.TXT:
                try:
                    idx = int(str(rr.rname).split(".")[0])
                    char_map[idx] = b"".join(rr.rdata.data).decode()
                except (ValueError, IndexError):
                    pass

        if len(char_map) < SECRET_LEN:
            print(f"Only received {len(char_map)}/{SECRET_LEN} characters.")
            missing = [i for i in range(SECRET_LEN) if i not in char_map]
            print(f"Missing indices: {missing[:20]}{'...' if len(missing) > 20 else ''}")
            if len(char_map) < SECRET_LEN:
                print("Cannot reconstruct full secret :(")
                return

        secret = "".join(char_map[i] for i in range(SECRET_LEN))
        print(f"Secret recovered (length {len(secret)}): {secret[:100]}...")

        # reconstruct subdomain exactly as the server does:
        #   range(0, k - 63 + 1, 63)  >  range(0, 757, 63)  >  13 chunks
        chunks = [
            secret[i : i + CHUNK_SIZE]
            for i in range(0, SECRET_LEN - CHUNK_SIZE + 1, CHUNK_SIZE)
        ]
        subdomain = ".".join(chunks)
        print(f"Subdomain: {len(chunks)} labels of {CHUNK_SIZE} chars each")

        # retrieve the flag
        print("Sending query 2: flag lookup...")
        q2 = build_flag_query(subdomain)
        r2 = send_query(s, q2)
        print(f"Response 2: {len(r2)} bytes")

        dns2 = DNSRecord.parse(r2)
        for rr in dns2.rr:
            if rr.rtype == QTYPE.TXT:
                flag = b"".join(rr.rdata.data).decode()
                print(f"\nFlag: {flag}")
                return

        print("Flag TXT record not found in response :(")
        print(f"Full response:\n{dns2}")


if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve.py  
Connected to challs.umdctf.io:32323  
Sending query 1: 819 individual character lookups...  
Response 1: 14661 bytes  
Secret recovered (length 819): oYXJNiCowrqOfkeGwcbuxRRaPmsPFtugvctNtYfZeOKdpKyttqsiiEshPDIUUNrnwaGMHJontZXycwHvouwcJkouDVhWRAlsMcJZ...  
Subdomain: 13 labels of 63 chars each  
Sending query 2: flag lookup...  
Response 2: 93 bytes  
  
Flag: UMDCTF{5Ur31Y_N0_0N3_W111_N071C3_MY_1N51D3r_7r4D1N6}
```

`UMDCTF{5Ur31Y_N0_0N3_W111_N071C3_MY_1N51D3r_7r4D1N6}`
