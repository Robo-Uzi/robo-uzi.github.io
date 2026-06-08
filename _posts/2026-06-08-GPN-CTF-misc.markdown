---
layout: post
title:  "Misc Challenges"
date:   2026-06-08 14:36:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /GPN-CTF-2026-misc/
---

**Title**: Double Fried

**Category**: Miscellaneous

**Author**: MisterPine

**Description**: I was planning to go to dinner with a friend but somthing felt off. Can you help me sort everything out?

I download the challenge file:
```shell
file kitchen_log.pcap  
kitchen_log.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

I open the pcap in wireshark and see a lot of `Syslog` packets. I notice a packet with the message ID `R0016` with the message `G`. This is the start of the flag. It gets concatenated from `R0016` to `R0067`. I followed the UDP stream and saved the entire stream to a text file. Then I ran this python:
```python
#!/usr/bin/env python3
import re

def main():
    with open("udp-stream-0.txt") as f:
        data = f.read()

    pattern = re.compile(r'R(\d+) - (.)')
    entries = [(int(m[0]), m[1]) for m in pattern.findall(data) if 16 <= int(m[0]) <= 67]
    entries.sort()
    print(''.join(ch for _, ch in entries))

if __name__ == "__main__":
    main()
    
```

output:
```shell
python3 solve.py  
GPNCTF{NIC3, yoU FOUnd 0uT Wh0 D1D n0t BEloNG thER3}
```

`GPNCTF{NIC3, yoU FOUnd 0uT Wh0 D1D n0t BEloNG thER3}`
