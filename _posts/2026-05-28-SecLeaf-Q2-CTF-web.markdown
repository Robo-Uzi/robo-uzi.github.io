---
layout: post
title:  "Web Challenges"
date:   2026-05-28 13:58:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /SecLeaf-Q2-CTF-2026-web/
---
* TOC
{:toc}

## Hidden_panel

**Category**: web

**Description**: We discovered a partially exposed internal web portal during reconnaissance. Developers claimed sensitive endpoints were “properly hidden.” Can you discover what was left behind? Link: [https://s3.secleaf.tech/](https://s3.secleaf.tech/) Flag format: `SecLeaf{}`

```shell
curl https://s3.secleaf.tech/  
<h1>SecureCorp Internal Portal</h1>  
<p>Unauthorized access prohibited.</p>  
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'a004cb9749bc26b5',t:'MTc3OTU0NzAxMQ=='};var a=document.createElement('script');a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.add EventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script>
```

```shell
curl https://s3.secleaf.tech/robots.txt  
# As a condition of accessing this website, you agree to abide by the following  
# content signals:  
  
# (a)  If a Content-Signal = yes, you may collect content for the corresponding  
#      use.  
# (b)  If a Content-Signal = no, you may not collect content for the  
#      corresponding use.  
# (c)  If the website operator does not include a Content-Signal for a  
#      corresponding use, the website operator neither grants nor restricts  
#      permission via Content-Signal with respect to the corresponding use.  
  
# The content signals and their meanings are:  
  
# search:   building a search index and providing search results (e.g., returning  
#           hyperlinks and short excerpts from your website's contents). Search does not  
#           include providing AI-generated search summaries.  
# ai-input: inputting content into one or more AI models (e.g., retrieval  
#           augmented generation, grounding, or other real-time taking of content for  
#           generative AI search answers).  
# ai-train: training or fine-tuning AI models.  
  
# ANY RESTRICTIONS EXPRESSED VIA CONTENT SIGNALS ARE EXPRESS RESERVATIONS OF  
# RIGHTS UNDER ARTICLE 4 OF THE EUROPEAN UNION DIRECTIVE 2019/790 ON COPYRIGHT  
# AND RELATED RIGHTS IN THE DIGITAL SINGLE MARKET.  
  
# BEGIN Cloudflare Managed content  
  
User-agent: *  
Content-Signal: search=yes,ai-train=no  
Allow: /  
  
User-agent: Amazonbot  
Disallow: /  
  
User-agent: Applebot-Extended  
Disallow: /  
  
User-agent: Bytespider  
Disallow: /  
  
User-agent: CCBot  
Disallow: /  
  
User-agent: ClaudeBot  
Disallow: /  
  
User-agent: CloudflareBrowserRenderingCrawler  
Disallow: /  
  
User-agent: Google-Extended  
Disallow: /  
  
User-agent: GPTBot  
Disallow: /  
  
User-agent: meta-externalagent  
Disallow: /  
  
# END Cloudflare Managed Content  
  
User-agent: *  
  
Disallow: /admin/  
Disallow: /backup/  
Disallow: /panel-final/  
  
# temporary developer note:  
# SecLeaf{r0b0ts_sh0uldnt_t4lk}
```

`SecLeaf{r0b0ts_sh0uldnt_t4lk}`

___

## Backup_leak

**Category**: web

**Description**: During a routine security review, we discovered this internal notes portal. Developers claim no sensitive information was ever exposed publicly. Can you verify that claim? Link: [https://backup-leak.secleaf.tech/](https://backup-leak.secleaf.tech/) Flag format: `SecLeaf{}`

On `https://backup-leak.secleaf.tech/` it says:
```shell
# Secure Notes Portal

Nothing sensitive is stored here.
```

Wappalyzer tells me the site runs php. After a while I correctly guess the filename `index.php.bak`. I downloaded the file from `https://backup-leak.secleaf.tech/index.php.bak` and viewed it:
```shell
cat index.php.bak  
<?php  
  
$flag = "SecLeaf{backup_files_are_dangerous}";  
  
echo "<h1>Development Backup</h1>";  
  
?>
```

`SecLeaf{backup_files_are_dangerous}`
