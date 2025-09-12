---
layout: post
title:  "gooses-typing-test"
date:   2025-09-11 18:47:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /watctf-2025-gooses-typing-test/
---

**Title:** gooses-typing-test

**Category:** web

**Author:** [virchau13](https://github.com/virchau13)

**Description:** Are you a good typer? The Goose challenges you!

`http://challs.watctf.org:3050/`

I go to the site and it is a typing speed test. On the page, if the expected word is `yes`, the only character you can enter is `y`, then `e`, etc.

I finish the typing test to see the request made when you finish:
```http
POST /doneTest HTTP/1.1
Host: challs.watctf.org:3050
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://challs.watctf.org:3050/
Content-Type: application/json
Content-Length: 10990
Origin: http://challs.watctf.org:3050
DNT: 1
Connection: keep-alive
Priority: u=0

{
  "startPoint":1757448562219,
  "typed":[
    {
      "key":"p",
      "time":1757448562280
    },
    {
      "key":"o",
      "time":1757448562316
    },
...
```

It sends json to the server containing the starting time and the time each key was pressed. Here is the response:
```json
{
  "msg":"Good job on getting 80 wpm! Next, try to aim for 500 wpm to get a special reward!"
}
```

I save the post request json contents to a file so I can easily change it and send it. I make this python script to get the flag:
```python
import json
import requests
import random

url = "http://challs.watctf.org:3050/doneTest"
headers = {"Content-Type": "application/json"}
file_path = "req.txt"

with open(file_path) as f:
    data = json.load(f)

start = data["startPoint"]
for i, k in enumerate(data["typed"]):
    k["time"] = start + i*14 + random.randint(-5, 5)

resp = requests.post(url, headers=headers, json=data)
print(resp.status_code, resp.text)
```

I changed the timing until I got it right:
```shell
python3 solve.py  
400 {"error":"too fast, you're obviously using a script"}

python3 solve.py  
200 {"msg":"Good job on getting 379 wpm! Next, try to aim for 500 wpm to get a special reward!"}

python3 solve.py  
200 {"msg":"Wow... you're really fast at typing (649.0696668109044 wpm)! Here's your reward: watctf{this_works_in_more_places_than_youd_think}"}
```
`watctf{this_works_in_more_places_than_youd_think}`