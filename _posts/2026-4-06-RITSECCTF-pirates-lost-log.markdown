---
layout: post
title:  "Pirate's Lost Log"
date:   2026-04-06 18:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RITSECCTF-2026-pirates-lost-log/
---

**Category**: misc

**Author:** @5731la

**Description**: Rumors have spread across the seven seas of a legendary pirate's log, hidden not on a physical island, but within the digital currents of the Domain Name System. This log, said to contain the coordinates of a vast buried treasure, was split into fragments and scattered across a secluded zone (`linksnsec.stellasec.com`) by a cunning old sea dog.

The fragments are concealed within the DNS, each leading to the next like a chain of clues on a treasure map. Our best guess is that the final piece of the puzzle, the one that reveals the treasure's location, will be the longest entry in the entire record, standing out from the rest.

Your mission: navigate the hidden paths within the pirate's domain. Follow the digital breadcrumbs to uncover every fragment. Find the single data record with the longest content to recover the complete log and claim the treasure.

_The DNS server responds to TCP only, either query it over TCP or use a public recursive resolver (1.1.1.1, etc) to access it. This challenge is exclusively DNS and **not** web!_

After a while I get the idea to look at any record for `0.linksnsec.stellasec.com`:
```shell
dig +dnssec +tcp @129.21.21.95 0.linksnsec.stellasec.com ANY  
  
; <<>> DiG 9.20.21-1~deb13u1-Debian <<>> +dnssec +tcp @129.21.21.95 0.linksnsec.stellasec.com ANY  
; (1 server found)  
;; global options: +cmd  
;; Got answer:  
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 13564  
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 4, ADDITIONAL: 1  
;; WARNING: recursion requested but not available  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags: do; udp: 1232  
; COOKIE: 5d5be291ade80b570100000069d0bbf723dc596a421e1036 (good)  
;; QUESTION SECTION:  
;0.linksnsec.stellasec.com.     IN      ANY  
  
;; AUTHORITY SECTION:  
linksnsec.stellasec.com. 86400  IN      SOA     linksnsecns.stellasec.com. stella.stellasec.com. 1774639332 604800 86400 2419200 604800  
linksnsec.stellasec.com. 86400  IN      RRSIG   SOA 14 3 86400 20260426182212 20260327182212 17032 linksnsec.stellasec.com. 1taOUWidgxhzZ5etk4DYW6milm2mOZOow8VV10bn3/o+mfXV2OP  
XtdKB ctm0ufDkpsumo6/bbSTPpnKTx0KTcbXKZ8V2HnSqxa5QH/tlJ3WUYzyZ /oT88giFJoZDXfqn  
linksnsec.stellasec.com. 86400  IN      NSEC    000n96.linksnsec.stellasec.com. NS SOA RRSIG NSEC DNSKEY  
linksnsec.stellasec.com. 86400  IN      RRSIG   NSEC 14 3 86400 20260426182212 20260327182212 17032 linksnsec.stellasec.com. +s+aWx7AUdOqEKSt0QKB3cpjrESAfhAIG4KDy9g8XrIs1+9pEz  
V6XcFu PV0n1Gcdy6FWan5HB8b5x8o9HR4EZ2yZSQlUp8EE3UPCtIThdcTsimau fL/+F7ngE8U8UYlB  
  
;; Query time: 303 msec  
;; SERVER: 129.21.21.95#53(129.21.21.95) (TCP)  
;; WHEN: Sat Apr 04 03:21:28 EDT 2026  
;; MSG SIZE  rcvd: 505
```

I find `000n96.linksnsec.stellasec.com`:
```shell
dig +dnssec +tcp @129.21.21.95 000n96.linksnsec.stellasec.com ANY  
  
; <<>> DiG 9.20.21-1~deb13u1-Debian <<>> +dnssec +tcp @129.21.21.95 000n96.linksnsec.stellasec.com ANY  
; (1 server found)  
;; global options: +cmd  
;; Got answer:  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15381  
;; flags: qr aa rd; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1  
;; WARNING: recursion requested but not available  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags: do; udp: 1232  
; COOKIE: c531b862c9704e670100000069d0c30e32e55e96ab30b78d (good)  
;; QUESTION SECTION:  
;000n96.linksnsec.stellasec.com.        IN      ANY  
  
;; ANSWER SECTION:  
000n96.linksnsec.stellasec.com. 86400 IN TXT    "exaggerateswoodiestcadavers"  
000n96.linksnsec.stellasec.com. 86400 IN RRSIG  TXT 14 4 86400 20260426182212 20260327182212 17032 linksnsec.stellasec.com. gFT8yo2jUCK2tMvDGl6O6gizkrn1AzOjcZo0Wn7ArhEgLhPdXHn  
EwIr/ Hb3Q1vUxj4bbLFe/2csYla8HgxFg1TFRZXPnNb93urlYV5WU/qYfycrY mtSFiXAf+ae96TjK  
000n96.linksnsec.stellasec.com. 86400 IN NSEC   002xg1.linksnsec.stellasec.com. TXT RRSIG NSEC  
000n96.linksnsec.stellasec.com. 86400 IN RRSIG  NSEC 14 4 86400 20260426182212 20260327182212 17032 linksnsec.stellasec.com. RGns+ZJz62/NNhV4T+CnYeqO9uCo+5liHlfWKX7k7bvva8/2Gs  
Hcn9HT kf1yn7kt/s7ZDYLYRTwYu8gh/eaBtPOb4CUaXacT6TNP+5XCec/+QO8B Nb0l4c9f50goJxTa  
  
;; Query time: 300 msec  
;; SERVER: 129.21.21.95#53(129.21.21.95) (TCP)  
;; WHEN: Sat Apr 04 03:51:43 EDT 2026  
;; MSG SIZE  rcvd: 481
```
Running `dig` on this one shows me a TXT record and a new subdomain `002xg1.linksnsec.stellasec.com`. I need to do DNSSEC zone walking.

I write a script to go through all of these and find the longest TXT record:
```python
import subprocess
import re

SERVER = "129.21.21.95"
START = "linksnsec.stellasec.com."

current = START
seen = set()
txt_records = []

while current not in seen:

    print("Querying:", current)
    seen.add(current)

    try:
        out = subprocess.check_output(
            ["dig", "+dnssec", "+tcp", f"@{SERVER}", current, "ANY"],
            text=True
        )
    except subprocess.CalledProcessError:
        break

    next_domain = None

    for line in out.splitlines():

        # Extract TXT
        txt_match = re.search(r'TXT\s+"([^"]+)"', line)
        if txt_match:
            txt = txt_match.group(1)
            txt_records.append(txt)

            if "RS{" in txt:
                print("\nFLAG FOUND:", txt)
                exit()

        # Extract NSEC next domain
        nsec_match = re.search(
            r'NSEC\s+([a-z0-9]+\.' + re.escape("linksnsec.stellasec.com.") + ')',
            line
        )

        if nsec_match:
            next_domain = nsec_match.group(1)

    if not next_domain:
        print("No next domain found, stopping.")
        break

    current = next_domain

print("\nTotal TXT records:", len(txt_records))

if txt_records:
    longest = max(txt_records, key=len)
    print("Longest TXT:", longest)
```

Output:
```shell
python3 solve2.py  
Querying: linksnsec.stellasec.com.  
Querying: 000n96.linksnsec.stellasec.com.  
Querying: 002xg1.linksnsec.stellasec.com.  
Querying: 03ad8h.linksnsec.stellasec.com.  
Querying: 05gx1e.linksnsec.stellasec.com.  
Querying: 05knqz.linksnsec.stellasec.com.  
Querying: 0735gx.linksnsec.stellasec.com.  
Querying: 07tvmd.linksnsec.stellasec.com.  
Querying: 0ambj4.linksnsec.stellasec.com.  
Querying: 0bfjr2.linksnsec.stellasec.com.  
Querying: 0et9r7.linksnsec.stellasec.com.
... etc
Querying: zqxs4f.linksnsec.stellasec.com.  
Querying: zr65gy.linksnsec.stellasec.com.  
Querying: ztgtkd.linksnsec.stellasec.com.  
Querying: zu755n.linksnsec.stellasec.com.  
Querying: zv9lk8.linksnsec.stellasec.com.  
Querying: zwsrkc.linksnsec.stellasec.com.  
Querying: zxv5gy.linksnsec.stellasec.com.  
Querying: zzp4xc.linksnsec.stellasec.com.  
Querying: zzsniz.linksnsec.stellasec.com.  
Querying: zzwknn.linksnsec.stellasec.com.  
No next domain found, stopping.  
  
Total TXT records: 1001  
Longest TXT: thebartentersawcaptainjackwalkintothebarwiththeshipswheelaroundhisnutsthebartenderaskedhimwhatwasgoingoncaptainjackrepliedyaaritsdrivingmenuts
```

`RS{thebartentersawcaptainjackwalkintothebarwiththeshipswheelaroundhisnutsthebartenderaskedhimwhatwasgoingoncaptainjackrepliedyaaritsdrivingmenuts}`
