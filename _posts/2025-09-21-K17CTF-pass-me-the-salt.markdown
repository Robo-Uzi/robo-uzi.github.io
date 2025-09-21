---
layout: post
title:  "pass me the salt"
date:   2025-09-21 10:25:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /K17-2025-pass-me-the-salt/
---

**Title:** pass me the salt

**Category:** beginner crypto

**Description:** Pretty pleaseeeeee 🥺🥺🥺

`nc challenge.secso.cc 7002`

I get this file:
```shell
from hashlib import sha1
import os
  
logins = {}  
salts = {}  
  
def create_account(login, pwd):  
   if login in logins.keys():  
       return False  
      
   salt = os.urandom(16)  
   salted_pwd = salt + (pwd).encode()  
   passw = sha1(salted_pwd).hexdigest()  
      
   logins[login] = passw  
   salts[login] = salt  
  
   return True  
  
def check_login(login, pwd):  
   if login not in logins:  
       return False  
  
   salt = salts[login]  
   salted_pwd = salt + bytes.fromhex(pwd)  
  
   passw = sha1(salted_pwd).hexdigest()  
   return passw == logins[login]  
  
def change_password(login, new_pass):  
   if login not in logins:  
       return  
      
   print(f"Current password: {logins[login]}")  
  
   logins[login] = new_pass  
  
if __name__ == "__main__":  
   create_account("admin", "admin".encode().hex())  
  
   while True:  
       option = input("1. Create Account\n2. Login\n3. Change Password\n(1, 2, 3)> ")  
       if option == "1":  
           login = input("Login: ")  
           pwd = input("Password: ")  
           if create_account(login, pwd.encode().hex()):  
               print("Account created!")  
           else:  
               print("Could not create account.")  
       elif option == "2":  
           login = input("Login: ")  
           pwd = input("Password: ")  
  
           if not check_login(login, pwd):  
               print("Invalid login or password.")  
               continue  
  
           if login == "admin":  
               if pwd != "admin".encode().hex():  
                   print(f"Congratulations! Here is your flag: {os.getenv("FLAG")}")  
               else:  
                   print("Your flag is in another castle...")  
           else:  
               print(f"Login successful as {login}!")  
       elif option == "3":  
           login = input("Login: ")  
           new_pass = input("New password: ")  
  
           change_password(login, new_pass)  
           print("Password changed!")  
       else:  
           print("Invalid option.")
```

On startup the server creates the admin account with `create_account("admin", "admin".encode().hex())`. That gets stored as `salted_pwd = salt + (pwd).encode()`. That gets stored as `passw = sha1(salted_pwd).hexdigest()`.

On login the server checks `salted_pwd = salt + bytes.fromhex(pwd)` and compares to the stored SHA1. Then `passw = sha1(salted_pwd).hexdigest()`.

So the server created admin with `pwd = "admin".encode().hex()` (stored input is the ASCII bytes of `"61646d696e"`). 

`pwd_input` must be the hex of the ASCII bytes `b"61646d696e"` which is `36313634366436393665`:
```shell
python3  
Python 3.11.2 (main, Apr 28 2025, 14:11:48) [GCC 12.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> print("admin".encode().hex().encode())  
b'61646d696e'  
>>> print("admin".encode().hex().encode().hex())  
36313634366436393665  
>>> exit()
```

```shell
nc challenge.secso.cc 7002  
1. Create Account  
2. Login  
3. Change Password  
(1, 2, 3)> 2  
Login: admin  
Password: 36313634366436393665  
Congratulations! Here is your flag: K17CTF{s4Lt_4nD_p3pper_is_ov3rr4t3d}  
4. Create Account  
5. Login  
6. Change Password  
(1, 2, 3)> ^C
```

`K17CTF{s4Lt_4nD_p3pper_is_ov3rr4t3d}`