---
layout: post
title:  "Ocean Wildlife Revenge"
date:   2026-04-06 18:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RITSECCTF-2026-ocean-wildlife-revenge/
---

**Category**: forensics

**Author:** @koromoseji

**Description**: See the first challenge

I solved the revenge challenge the same way I solved the first one. I think they just removed the `strings` and `grep` trick.

I get the challenge files:
```shell
file *  
metadata.yaml:         ASCII text, with very long lines (332)  
mystery_message_0.db3: SQLite 3.x database, last written using SQLite version 3037002, file counter 981, database pages 29, cookie 0x5, schema 4, UTF-8, version-valid-for 981
```

I inspect the messaged but this time it seems the flag is hidden in the messaged where `topic_id` equals `4`:
```python
import sqlite3 

conn = sqlite3.connect("mystery_message_0.db3") 
cur = conn.cursor() 

cur.execute("SELECT * FROM messages WHERE topic_id = 4 ORDER BY id") 
rows = cur.fetchall() 

for row in rows: 
	print(row)
```

Output:
```shell
python3 solve3.py  
(130, 4, 1775257737859532937, b'\x00\x01\x00\x00C\x00\x00\x00{"cmd": "pen", "r": 255, "g": 255, "b": 255, "width": 3, "off": 0}\x00\x00')  
(133, 4, 1775257737875870898, b'\x00\x01\x00\x00F\x00\x00\x00{"cmd": "teleport", "x": 1.6949999999999994, "y": 6.67, "theta": 0.0}\x00\x00\x00')  
(136, 4, 1775257737891840111, b'\x00\x01\x00\x00C\x00\x00\x00{"cmd": "pen", "r": 255, "g": 255, "b": 255, "width": 3, "off": 1}\x00\x00')  
(139, 4, 1775257737908188872, b'\x00\x01\x00\x00F\x00\x00\x00{"cmd": "teleport", "x": 1.6949999999999994, "y": 6.67, "theta": 0.0}\x00\x00\x00')  
(142, 4, 1775257737923936354, b'\x00\x01\x00\x00C\x00\x00\x00{"cmd": "pen", "r": 255, "g": 255, "b": 255, "width": 3, "off": 0}\x00\x00')  
(145, 4, 1775257737940308186, b'\x00\x01\x00\x00E\x00\x00\x00{"cmd": "teleport", "x": 2.079999999999999, "y": 6.67, "theta": 0.0}\x00\x00\x00\x00')
... etc
```

I use the same solve script, just change the `topic_id`:
```python
import sqlite3
import json
import matplotlib.pyplot as plt

# connect to db
conn = sqlite3.connect("mystery_message_0.db3")
cur = conn.cursor()

# fetch the 4th column from messages
cur.execute("SELECT * FROM messages WHERE topic_id = 4 ORDER BY id")
rows = cur.fetchall()

# initialize drawing state
lines = []
pen_down = False
prev_x = None
prev_y = None

for row in rows:
    payload = row[3]  # 4th column contains the message bytes
    payload_str = payload.decode(errors='ignore')
    
    start = payload_str.find('{')
    end = payload_str.rfind('}') + 1
    if start == -1 or end == -1:
        continue

    data = json.loads(payload_str[start:end])

    if data['cmd'] == 'pen':
        pen_down = not bool(data.get('off', 0))  # off=1 means pen lifted
    elif data['cmd'] == 'teleport':
        x = data['x']
        y = data['y']
        if prev_x is not None and prev_y is not None and pen_down:
            lines.append((prev_x, prev_y, x, y))
        prev_x, prev_y = x, y

# plot the lines
plt.figure(figsize=(8, 8))
for x1, y1, x2, y2 in lines:
    plt.plot([x1, x2], [y1, y2], color='black', linewidth=2)

plt.axis('equal')
plt.axis('off')
plt.show()
```

Output image:

![Alt text](/images/flag-solved-2.png)

For some reason it missed the first line for `R`, but you can still read the flag!

`RS{W4tch1ng_r0b0t_turtl3s}`