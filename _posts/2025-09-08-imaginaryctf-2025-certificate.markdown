---
layout: post
title:  "certificate"
date:   2025-09-08 21:39:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /imaginaryctf-2025-certificate/
---

**Title:** certificate

**Category:** Web

**Creator:** Eth007

**Description:** As a thank you for playing our CTF, we're giving out participation certificates! Each one comes with a custom flag, but I bet you can't get the flag belonging to Eth007!

`https://eth007.me/cert/`

I visit the site. It generates a custom certificate like the description says. You can change the `Participant name`, `Title`, `Date`, and `Design` (Classic or Modern) when generating the certificate. 

The javascript for generating the certificate (an `.svg`) is in the source code. Here are the main relevant lines from that code:
```javascript
function customHash(str) {
    let h = 1337;
    for (let i = 0; i < str.length; i++) {
        h = (h * 31 + str.charCodeAt(i)) ^ (h >>> 7);
        h = h >>> 0; // force unsigned
    }
    return h.toString(16);
}

function makeFlag(name) {
    const clean = name.trim() || "anon";
    const h = customHash(clean);
    return `ictf{${h}}`;
}

function renderPreview() {
    var name = nameInput.value.trim();
    if (name == "Eth007") {
        name = "REDACTED"
    }
    
<desc>${makeFlag(participant)}</desc>
```

If you enter `Eth007` the UI swaps it to `"REDACTED"` but the `makeFlag()` function is available globally. In the browser dev tools console run `makeFlag("Eth007")`.

`ictf{7b4b3965}`