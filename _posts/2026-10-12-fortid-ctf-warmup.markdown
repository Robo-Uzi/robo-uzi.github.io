---
layout: post
title:  "Warmup Challenges"
date:   2026-10-12 12:40:00 -0400
author: uzi
tags: [CTF]
permalink: /fortid-ctf-2026-warmup/
---
* TOC
{:toc}

## Baby Web

**Author**: dsantl

**Category**: web

**Description**: The flag is only one click away... if you can catch it! 🏃

I get the challenge file `index.html`:
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Catch Me If You Can</title>
  <style>
    * {
      box-sizing: border-box;
    }

    body {
      min-height: 100vh;
      margin: 0;
      overflow: hidden;
      display: grid;
      place-items: center;
      padding: 24px;
      color: #f8fafc;
      background:
        radial-gradient(circle at 20% 20%, #312e81 0, transparent 32%),
        radial-gradient(circle at 80% 80%, #164e63 0, transparent 32%),
        #0f172a;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    }

    main {
      width: min(560px, 100%);
      padding: 48px 36px;
      text-align: center;
      background: rgb(15 23 42 / 72%);
      border: 1px solid #475569;
      border-radius: 20px;
      box-shadow: 0 24px 70px rgb(0 0 0 / 35%);
      backdrop-filter: blur(10px);
    }

    h1 {
      margin: 0 0 12px;
      font-size: clamp(2rem, 7vw, 3.5rem);
    }

    p {
      margin: 0;
      color: #cbd5e1;
      line-height: 1.6;
    }

    #runaway {
      position: fixed;
      left: 50%;
      top: 70%;
      z-index: 2;
      transform: translate(-50%, -50%);
      padding: 14px 24px;
      color: #082f49;
      background: #67e8f9;
      border: 0;
      border-radius: 999px;
      box-shadow: 0 8px 24px rgb(103 232 249 / 30%);
      font: 700 1rem/1 system-ui, sans-serif;
      cursor: pointer;
      transition: left 120ms ease, top 120ms ease, transform 120ms ease;
    }

    #runaway:hover {
      background: #a5f3fc;
    }

    #flag {
      min-height: 1.6em;
      margin-top: 28px;
      color: #67e8f9;
      font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
      font-weight: 700;
      overflow-wrap: anywhere;
    }

    .success {
      animation: pop 350ms ease-out;
    }

    @keyframes pop {
      50% { transform: scale(1.06); }
    }
  </style>
</head>
<body>
  <main>
    <h1>Catch me if you can!</h1>
    <p>The flag is one click away.<br>You only have to catch the button.</p>
    <div id="flag" aria-live="polite"></div>
  </main>

  <button id="runaway" type="button">Show me the flag</button>

  <script>
    const button = document.querySelector("#runaway");
    const output = document.querySelector("#flag");

    function runAway() {
      const padding = 16;
      const maxX = Math.max(padding, window.innerWidth - button.offsetWidth - padding);
      const maxY = Math.max(padding, window.innerHeight - button.offsetHeight - padding);

      button.style.transform = "none";
      button.style.left = `${padding + Math.random() * (maxX - padding)}px`;
      button.style.top = `${padding + Math.random() * (maxY - padding)}px`;
    }

    button.addEventListener("mouseenter", runAway);

    button.addEventListener("click", () => {
      const scrambled = [
        "=0XNx81Nx81auFDa38Vdwk3",
        "X1QzXkJHNo91N0g2NfdDMu9V",
        "Nx8lYzc3eElEdy9mR"
      ].join("");

      output.textContent = atob([...scrambled].reverse().join(""));
      output.classList.add("success");
      button.textContent = "You caught me!";
      button.removeEventListener("mouseenter", runAway);
      button.disabled = true;
    });
  </script>
</body>
</html>
```

I opened the file in my browser. The page never allows you to click the button. It gets moved when you try to hover your mouse over it. The string gets reversed and base64 decoded:
```shell
echo '=0XNx81Nx81auFDa38Vdwk3X1QzXkJHNo91N0g2NfdDMu9VNx8lYzc3eElEdy9mR' | rev | base64 -d  
FortID{w3b_15_n07_7h47_h4rd_45_y0u_7h1nk_17_15}
```

`FortID{w3b_15_n07_7h47_h4rd_45_y0u_7h1nk_17_15}`

___

## Baby Rev

**Author**: ipaljak

**Category**: rev

**Description**: I've been texting a lot recently, but can't seem to strike a conversation with this person. They keep asking about some kind of flag...

I get the challenge file `chall.pyc`:
```shell
file chall.pyc  
chall.pyc: Byte-compiled Python module for CPython 3.10 (magic: 3439), timestamp-based, .py timestamp: Wed Aug  5 21:34:28 2026 UTC, .py size: 816 bytes
```

I decompiled the file on [https://www.lddgo.net/en/string/pyc-compile-decompile](https://www.lddgo.net/en/string/pyc-compile-decompile):
```python
# Visit https://www.lddgo.net/en/string/pyc-compile-decompile for more information
# Version : Python 3.10

import string
from hashlib import sha256
from t9text import t9

def nope():
    print('nope! ಠ_ಠ')
    exit(0)

user_input = input('pls text me the flag :P\n> ')
if len(user_input) != 81:
    nope()
if not user_input.startswith('FortID{') or user_input.endswith('}'):
    nope()
words = user_input[len('FortID{'):-len('}')]
for w in words.split('_'):
    if not all((lambda .0: [ c in string.ascii_lowercase for c in .0 ])(w)) and all((lambda .0: [ c in string.digits for c in .0 ])(w)):
        nope()
words = '-'.join(words.split('_'))
if t9.get_touchtone(words)[::-1] != '33696-7383-3824-4-9465664238-36-32347-878267-8766-348-55487-74-0133-24566':
    nope()
if sha256(user_input.encode('utf-8')).hexdigest() != '3576c495f89236515fbbfbfb519dceb9899122e92460d418d2ce37a1ff357e0c':
    nope()
print('*\\0/* I knew you could do it! ^5')
```

I had to look up this `t9text` library. On [https://pypi.org/project/t9text/](https://pypi.org/project/t9text/) they explain what it does:
```shell
t9text is meant for two main purposes:

- Given a string of alphabetic text (plus -, *, #), t9text can tell a user which numeric touchtone phone buttons correspond to it, e.g.:

from t9text import t9
daily_show = '1-210-WH-CRIME'  # The Daily Show's phone number for Trump to confess his crimes to :-)

buttons = t9.get_touchtone(daily_show)
print(buttons)  # prints "1-210-94-27463"

- Given a string of 10 or fewer digits, display all possible T9 "words", e.g.:

from t9text import t9
digits = "2332" #can also be int such as 2332

words = t9.get_words(digits)
for word in words:
    print(word)  # ["adda", addb", "addc", "adea" ...]
```

I reversed the t9 encoded string to get this:
```shell
66542 3310 47 78455 843 6678 762878 74323 63 8324665649 4 4283 3837 69633
```

I put that into [https://www.dcode.fr/t9-cipher](https://www.dcode.fr/t9-cipher) and I was able to figure out the flag. The only one that couldnt decode was `3310`!

`FortID{nokia_3310_is_still_the_most_robust_piece_of_technology_i_have_ever_owned}`
