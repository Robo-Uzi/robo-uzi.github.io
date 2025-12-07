---
layout: post
title:  "iuesbitaipsi"
date:   2025-12-07 16:45:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /nullCTF-iuesbitaipsi/
---

**Title**: iuesbitaipsi

**Category**: Forensics

**Author**: mrbgd

**Description**: ayay uat is iuor taip for iuesbi?

I get the challenge files:
```shell
unzip iuesbitaipsi.zip  
Archive:  iuesbitaipsi.zip  
 inflating: README.md                  
 inflating: nullctf.pcapng
_________________________________________________________________________________
cat README.md  
name: iuesbitaipsi  
  
category: Forensics  
  
dificulty: easy  
  
description: ayay uat is iuor taip for iuesbi?
_________________________________________________________________________________
file nullctf.pcapng  
nullctf.pcapng: pcapng capture file - version 1.0
```

I open the pcap in wireshark and see a lot of USB packets. The traffic comes from a USB hub that other devices are connected to. There is a keyboard and mouse combo at address 1, address 2 is the hub, address 3 is a generic HID device, address 4 is another keyboard and mouse, and address 5 has an audio device.

I started with pulling the audio from the pcap and opening it in audacity:
```shell
file audio.wav  
audio.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 48000 Hz
```
I could not see any morse code and the spectrogram view was unhelpful. I cant tell what they are typing just from sound...

I get the HID data from the keyboard:
```shell
tshark -r nullctf.pcapng -Y "usb.device_address == 1 && usb.endpoint_address == 0x81" -T fields -e usbhid.data > hid_data_all.txt
```

Looks good:
```shell
cat hid_data_all.txt | head  
0000110000000000  
  
0000000000000000  
  
0000180000000000  
  
0000000000000000  
  
00000f0000000000
```

I run this python to get the flag:
```python
#!/usr/bin/env python3
import re

# USB HID keycode mapping
keymap = {
    0x04: 'a', 0x05: 'b', 0x06: 'c', 0x07: 'd', 0x08: 'e', 0x09: 'f',
    0x0A: 'g', 0x0B: 'h', 0x0C: 'i', 0x0D: 'j', 0x0E: 'k', 0x0F: 'l',
    0x10: 'm', 0x11: 'n', 0x12: 'o', 0x13: 'p', 0x14: 'q', 0x15: 'r',
    0x16: 's', 0x17: 't', 0x18: 'u', 0x19: 'v', 0x1A: 'w', 0x1B: 'x',
    0x1C: 'y', 0x1D: 'z', 0x1E: '1', 0x1F: '2', 0x20: '3', 0x21: '4',
    0x22: '5', 0x23: '6', 0x24: '7', 0x25: '8', 0x26: '9', 0x27: '0',
    0x28: '\n', 0x2A: '[BACKSPACE]', 0x2B: '\t', 0x2C: ' ', 0x2D: '-',
    0x2E: '=', 0x2F: '[', 0x30: ']', 0x31: '\\', 0x33: ';', 0x34: "'",
    0x35: '`', 0x36: ',', 0x37: '.', 0x38: '/'
}

shifted_keymap = {
    0x04: 'A', 0x05: 'B', 0x06: 'C', 0x07: 'D', 0x08: 'E', 0x09: 'F',
    0x0A: 'G', 0x0B: 'H', 0x0C: 'I', 0x0D: 'J', 0x0E: 'K', 0x0F: 'L',
    0x10: 'M', 0x11: 'N', 0x12: 'O', 0x13: 'P', 0x14: 'Q', 0x15: 'R',
    0x16: 'S', 0x17: 'T', 0x18: 'U', 0x19: 'V', 0x1A: 'W', 0x1B: 'X',
    0x1C: 'Y', 0x1D: 'Z', 0x1E: '!', 0x1F: '@', 0x20: '#', 0x21: '$',
    0x22: '%', 0x23: '^', 0x24: '&', 0x25: '*', 0x26: '(', 0x27: ')',
    0x2C: ' ', 0x2D: '_', 0x2E: '+', 0x2F: '{', 0x30: '}', 0x31: '|',
    0x33: ':', 0x34: '"', 0x35: '~', 0x36: '<', 0x37: '>', 0x38: '?'
}

output = ""
last_data = None

with open('hid_data_all.txt', 'r') as f:
    for line in f:
        line = line.strip()
        if not line:
            continue
        hex_str = line.replace(':', '')
        if hex_str == last_data or len(hex_str) < 16:
            continue
        last_data = hex_str
        try:
            bytes_data = bytes.fromhex(hex_str[:16])
        except:
            continue
        modifier = bytes_data[0]
        keys = bytes_data[2:8]
        if all(k == 0 for k in keys):
            continue
        shift_pressed = modifier & 0x22
        for key in keys:
            if key == 0:
                continue
            if shift_pressed and key in shifted_keymap:
                output += shifted_keymap[key]
            elif key in keymap:
                output += keymap[key]
            elif key == 0x2A:
                output = output[:-1]
            else:
                output += f'[0x{key:02x}]'

match = re.search(r'nullctf\{[^}]+\}', output, re.IGNORECASE)
if match:
    print(match.group(0))
```

Output:
```shell
python3 keyboard.py  
nullctf{4nd_7h47s_h0w_4_k3yl0gg3r_-w0rks}
```
Im not sure why the `-` is there but I remove and it and get the flag!

`nullctf{4nd_7h47s_h0w_4_k3yl0gg3r_w0rks}`