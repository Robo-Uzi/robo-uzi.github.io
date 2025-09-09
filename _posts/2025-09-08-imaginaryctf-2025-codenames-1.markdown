---
layout: post
title:  "codenames-1"
date:   2025-09-08 21:10:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /imaginaryctf-2025-codenames-1/
---

**Title:** codenames-1

**Category:** Web

**Creator:** Eth007

**Description:** I hear that multilingual codenames is all the rage these days. Flag is in `/flag.txt`.

`http://codenames-1.chal.imaginaryctf.org/` (bot does not work on this instance, look at codenames-2 for working bot).

I get the source code for the site. Looking at the code, I run `cat * | grep flag` to check where the flag is stored:
```shell
cat * | grep flag
RUN echo ictf{testing_flag_1} > /flag.txt  
ENV FLAG_2 ictf{testing_flag_2}
```
Looks like the second flag will be stored in an environment variable (for codenames-2 challenge).

I go to the site and register an account. You can choose a language when creating a game on the site. 

I intercept the request with burpsuite and replace `language=de` with `language=/flag`. After forwarding the request and using the resulting game code, I get the flag at:

`http://codenames-1.chal.imaginaryctf.org/game/J9L727`

`ictf{common_os_path_join_L_b19d35ca}`