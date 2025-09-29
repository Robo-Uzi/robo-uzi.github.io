---
layout: post
title:  "Lunar Auth"
date:   2025-09-29 12:28:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-lunar-auth/
---

**Title:** Lunar Auth

**Category:** web

**Author:** alimuhammadsecured

**Description:** Infiltrate the LunarAuth admin panel and gain access to the super secret FLAG artifact! `https://comet.sunshinectf.games`

I visit the site and see this:
```
We have amazing auth, no robots are allowed — only **admins** may dock. Solve the challenge and gain access to the admin portal.

Robot Protocol

Automated agents are forbidden — human captains only. 🛸

Command Console

Current mission: access admin

May your inspector be sharp and your console kinder than the void. ✦
```
Lots of hints here. I visit `https://comet.sunshinectf.games/robots.txt` and see `/admin` disallowed.

I go to `https://comet.sunshinectf.games/admin` and view the source code. I find this:
```html
<script>
    /*
    To reduce load on our servers from the recent space DDOS-ers we have lowered     login attempts by using Base64 encoded encryption
    ("encryption" 💀) on the client side.
    
    TODO: implement proper encryption.
    */
    const real_username = atob("YWxpbXVoYW1tYWRzZWN1cmVk");
    const real_passwd   = atob("UzNjdXI0X1BAJCR3MFJEIQ==");

    document.addEventListener("DOMContentLoaded", () => {
        const form = document.querySelector("form");

        function handleSubmit(evt) {
        evt.preventDefault();

        const username = form.elements["username"].value;
        const password = form.elements["password"].value;

        if (username === real_username && password === real_passwd) {
            // remove this handler and allow form submission
            form.removeEventListener("submit", handleSubmit);
            form.submit();
        } else {
            alert("[ Invalid credentials ]");
        }
        }

        form.addEventListener("submit", handleSubmit);
    });
</script>
```
Simply decode the base64 encoded creds. Login with `alimuhammadsecured:S3cur4_P@$$w0RD!`

`sun{cl1ent_s1d3_auth_1s_N3V3R_a_g00d_1d3A_983765367890393232}`