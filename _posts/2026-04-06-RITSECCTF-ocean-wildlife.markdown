---
layout: post
title:  "Ocean Wildlife"
date:   2026-04-06 18:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RITSECCTF-2026-ocean-wildlife/
---

**Category**: forensics

**Author:** @koromoseji

**Description**: You have recieved a message in a bottle, saying something about the strange behavior of sea creatures. I wonder what that could be about?

I get the challenge files:
```shell
file *  
metadata.yaml:         ASCII text, with very long lines (602)  
mystery_message_0.db3: SQLite 3.x database, last written using SQLite version 3037002, file counter 892, database pages 29, cookie 0x5, schema 4, UTF-8, version-valid-for 892
```

Not sure whats going on with this challenge. The flag is just there:
```shell
strings mystery_message_0.db3 | grep RS  
Finished drawing: RS{f0ll0w_th3_5ea_Turtles}
```

After some time I realized the flag is hidden inside drawing commands in the messages table with the `topic_id 6`. To inspect the messages I ran this:
```python
import sqlite3 

conn = sqlite3.connect("mystery_message_0.db3") 
cur = conn.cursor() 

cur.execute("SELECT * FROM messages WHERE topic_id = 6 ORDER BY id") 
rows = cur.fetchall() 

for row in rows: 
	print(row)
```

Output:
```shell
python3 solve.py  
(156, 6, 1774823543910844173, b'\x00\x01\x00\x00F\x00\x00\x00{"cmd": "teleport", "x": 1.044999999999999, "y": 5.845, "theta": 0.0}\x00\x00\x00')  
(159, 6, 1774823543927583963, b'\x00\x01\x00\x00C\x00\x00\x00{"cmd": "pen", "r": 255, "g": 255, "b": 255, "width": 3, "off": 0}\x00\x00')  
(162, 6, 1774823543942809345, b'\x00\x01\x00\x00E\x00\x00\x00{"cmd": "teleport", "x": 1.044999999999999, "y": 6.67, "theta": 0.0}\x00\x00\x00\x00')  
(165, 6, 1774823543959405207, b'\x00\x01\x00\x00C\x00\x00\x00{"cmd": "pen", "r": 255, "g": 255, "b": 255, "width": 3, "off": 1}\x00\x00')  
(168, 6, 1774823543975748771, b'\x00\x01\x00\x00E\x00\x00\x00{"cmd": "teleport", "x": 1.044999999999999, "y": 6.67, "theta": 0.0}\x00\x00\x00\x00')  
(171, 6, 1774823543991634972, b'\x00\x01\x00\x00C\x00\x00\x00{"cmd": "pen", "r": 255, "g": 255, "b": 255, "width": 3, "off": 0}\x00\x00')  
(174, 6, 1774823544007666674, b'\x00\x01\x00\x00E\x00\x00\x00{"cmd": "teleport", "x": 1.429999999999999, "y": 6.67, "theta": 0.0}\x00\x00\x00\x00')  
(177, 6, 1774823544023237749, b'\x00\x01\x00\x00G\x00\x00\x00{"cmd": "teleport", "x": 1.594999999999999, "y": 6.5325, "theta": 0.0}\x00\x00')
... etc etc
```

From the output `cmd` can be:
- `teleport`: moves the pen to `(x, y, theta)` coordinates
- `pen`: pen control with keys like `"r"`, `"g"`, `"b"`, `"width"`, `"off"`

Solve script:
```python
import sqlite3
import json
import matplotlib.pyplot as plt

# connect to db
conn = sqlite3.connect("mystery_message_0.db3")
cur = conn.cursor()

# fetch the 4th column from messages
cur.execute("SELECT * FROM messages WHERE topic_id = 6 ORDER BY id")
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

![Alt text](/images/flag-solved.png)

`RS{f0ll0w_th3_5ea_Turtles}`
