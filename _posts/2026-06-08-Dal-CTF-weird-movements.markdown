---
layout: post
title:  "Weird Movements"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-weird-movements/
---

**Title**: Weird Movements

**Author**: kvasir

**Category**: Forensics

**Description**: Have you ever heard of Sergey Ushanka? They told me about him last AtHack

I download the challenge file:
```shell
file capture.pcapng  
capture.pcapng: pcapng capture file - version 1.0
```

I opened the pcap in wireshark and there is about 8600 USB packets. I need a script to extract mouse movements and visualize them. My script needs to parse the pcap block by block using the pcapng structure. It should only processes blocks of type `0x00000006` (actual captured packets). 

From each packet the script will read the USB transfer header:
- `urb_type` (byte 8): expects `0x43` (`URB_COMPLETE`)
- `xfer_type` (byte 9): expects `1` (interrupt transfer)
- `devnum` (byte 11): expects `40` (target device)
- `len_cap` (bytes 36‑40): length of the HID report

Then it needs to parse the HID mouse report. It will skip to offset 64 (where the payload starts) and read:
- `buttons` (1st byte): not used in movement
- `dx` (2nd byte, signed): X movement
- `dy` (3rd byte, signed): Y movement

Once the script gathers the coordinates, plot them and flip the y axis (screen coordinates are inverted relative to the raw data). 

Solve script:
```python
import struct, matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

PCAPNG = "capture.pcapng"

def read_block(f):
    hdr = f.read(8)
    if len(hdr) < 8:
        return None, None, None
    btype, blen = struct.unpack('<II', hdr)
    if blen < 12:
        return btype, blen, b''
    body = f.read(blen - 12)
    f.read(4)
    return btype, blen, body

movements = []

with open(PCAPNG, 'rb') as f:
    while True:
        btype, blen, body = read_block(f)
        if btype is None:
            break
        if btype != 0x00000006:
            continue
        
        iface_id, ts_hi, ts_lo, cap_len, orig_len = struct.unpack('<IIIII', body[:20])
        pkt_data = body[20:20+cap_len]
        
        if len(pkt_data) < 64:
            continue
        
        urb_type = pkt_data[8]
        xfer_type = pkt_data[9]
        epnum = pkt_data[10]
        devnum = pkt_data[11]
        len_cap = struct.unpack('<I', pkt_data[36:40])[0]
        
        # interrupt Complete, device 40, endpoint 0x81, with data
        if xfer_type == 1 and urb_type == 0x43 and devnum == 40 and len_cap >= 3:
            hid = pkt_data[64:64+len_cap]
            buttons = hid[0]
            dx = struct.unpack('b', bytes([hid[1]]))[0]
            dy = struct.unpack('b', bytes([hid[2]]))[0]
            movements.append((buttons, dx, dy))

print(f"Total movements: {len(movements)}")
print(f"Sample: {movements[:5]}")

# Accumulate positions
x, y = 0, 0
xs, ys = [0], [0]
for btn, dx, dy in movements:
    x += dx
    y += dy
    xs.append(x)
    ys.append(y)

print(f"X range: {min(xs)} to {max(xs)}")
print(f"Y range: {min(ys)} to {max(ys)}")

# flip Y because screen coordinates are inverted
fig, ax = plt.subplots(figsize=(20, 6), facecolor='black')
ax.set_facecolor('black')
ax.plot(xs, [-y for y in ys], color='lime', linewidth=0.5, alpha=0.9)
ax.set_aspect('equal')
ax.axis('off')
ax.set_title('USB Mouse Movements', color='white', fontsize=14)
plt.tight_layout()
plt.savefig('mouse-movements.png', dpi=150, bbox_inches='tight', facecolor='black')
plt.close()
print("Saved mouse-movements.png")
```

Output:
```shell
python3 solve.py  
Total movements: 4254  
Sample: [(0, -3, 0), (0, -8, 1), (0, -7, 1), (0, -7, 1), (0, -7, 1)]  
X range: -1021 to 340  
Y range: -197 to 99  
Saved mouse-movements.png
```

`mouse-movements.png`:

![Alt text](/images/mouse-movements.png)

The flag needs `rot13`ed:
```shell
echo 'QNYPGS{U3U3_V_F2_C41AG}' | rot13  
DALCTF{H3H3_I_S2_P41NT}
```

`DALCTF{H3H3_I_S2_P41NT}`