---
layout: post
title:  "Web Challenges"
date:   2026-08-04 17:35:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BashBash-CTF-web/
---
* TOC
{:toc}

## Secret hidden website

**Category**: web

**Author**: Alyssa

**Description**: We have discovered one of our cybervillain's secret websites located at [https://secret-hidden-website.bushbash.cssa.club](https://secret-hidden-website.bushbash.cssa.club) but we don't seem to be able to access it. Can you have work out the next step?

The given link did not resolve:
```shell
curl https://secret-hidden-website.bushbash.cssa.club/  
curl: (6) Could not resolve host: secret-hidden-website.bushbash.cssa.club
```

I started looking at dns records. This didn't lead anywhere. I found the flag on [https://crt.sh/](https://crt.sh/):

![Alt text](/images/bushbashweb1.png)

`bushbash{h0w-d1d-y0u-f1nd-th1s}`

___

## Old website

**Category**: web

**Author**: Eisverygoodletter

**Description**: We found this website running using one of cybervillain Zoowee Blubberworth's old domain names. He's supposed to be in jail right now so there's really no reason why this server could be up and running. It probably hasn't been updated in a year or so. Can you hack in and have a peek around?

The website is at [http://34.40.133.67:8080](http://34.40.133.67:8080)

I look at the website:
```shell
curl http://34.40.133.67:8080/
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charSet="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="preload" as="script" fetchPriority="low" href="/_next/static/chunks/webpack-5adebf9f62dc3001.js" />
    <script src="/_next/static/chunks/4bd1b696-80bcaf75e1b4285e.js" async=""></script>
    <script src="/_next/static/chunks/517-0ec0e5f25493795c.js" async=""></script>
    <script src="/_next/static/chunks/main-app-a7031ed1fe6ebaad.js" async=""></script>
    <script src="/_next/static/chunks/polyfills-42372ed130431b0a.js" noModule=""></script>
  </head>
  <body>
    <main>
      <h1>Zoowee Blubberworth&#x27;s epic website</h1>
      <p>No content loaded - database offline</p>
    </main>
    <script src="/_next/static/chunks/webpack-5adebf9f62dc3001.js" async=""></script>
    <script>
      (self.__next_f = self.__next_f || []).push([0])
    </script>
    <script>
      self.__next_f.push([1, "1:\"$Sreact.fragment\"\n2:I[5244,[],\"\"]\n3:I[3866,[],\"\"]\n5:I[6213,[],\"OutletBoundary\"]\n7:I[6213,[],\"MetadataBoundary\"]\n9:I[6213,[],\"ViewportBoundary\"]\nb:I[4835,[],\"\"]\n0:{\"P\":null,\"b\":\"nLUkWzLoBFZT61KFqWxQ0\",\"p\":\"\",\"c\":[\"\",\"\"],\"i\":false,\"f\":[[[\"\",{\"children\":[\"__PAGE__\",{}]},\"$undefined\",\"$undefined\",true],[\"\",[\"$\",\"$1\",\"c\",{\"children\":[null,[\"$\",\"html\",null,{\"lang\":\"en\",\"children\":[\"$\",\"body\",null,{\"children\":[\"$\",\"$L2\",null,{\"parallelRouterKey\":\"children\",\"segmentPath\":[\"children\"],\"error\":\"$undefined\",\"errorStyles\":\"$undefined\",\"errorScripts\":\"$undefined\",\"template\":[\"$\",\"$L3\",null,{}],\"templateStyles\":\"$undefined\",\"templateScripts\":\"$undefined\",\"notFound\":[[\"$\",\"title\",null,{\"children\":\"404: This page could not be found.\"}],[\"$\",\"div\",null,{\"style\":{\"fontFamily\":\"system-ui,\\\"Segoe UI\\\",Roboto,Helvetica,Arial,sans-serif,\\\"Apple Color Emoji\\\",\\\"Segoe UI Emoji\\\"\",\"height\":\"100vh\",\"textAlign\":\"center\",\"display\":\"flex\",\"flexDirection\":\"column\",\"alignItems\":\"center\",\"justifyContent\":\"center\"},\"children\":[\"$\",\"div\",null,{\"children\":[[\"$\",\"style\",null,{\"dangerouslySetInnerHTML\":{\"__html\":\"body{color:#000;background:#fff;margin:0}.next-error-h1{border-right:1px solid rgba(0,0,0,.3)}@media (prefers-color-scheme:dark){body{color:#fff;background:#000}.next-error-h1{border-right:1px solid rgba(255,255,255,.3)}}\"}}],[\"$\",\"h1\",null,{\"className\":\"next-error-h1\",\"style\":{\"display\":\"inline-block\",\"margin\":\"0 20px 0 0\",\"padding\":\"0 23px 0 0\",\"fontSize\":24,\"fontWeight\":500,\"verticalAlign\":\"top\",\"lineHeight\":\"49px\"},\"children\":\"404\"}],[\"$\",\"div\",null,{\"style\":{\"display\":\"inline-block\"},\"children\":[\"$\",\"h2\",null,{\"style\":{\"fontSize\":14,\"fontWeight\":400,\"lineHeight\":\"49px\",\"margin\":0},\"children\":\"This page could not be found.\"}]}]]}]}]],\"notFoundStyles\":[]}]}]}]]}],{\"children\":[\"__PAGE__\",[\"$\",\"$1\",\"c\",{\"children\":[\"$L4\",null,[\"$\",\"$L5\",null,{\"children\":\"$L6\"}]]}],{},null]},null],[\"$\",\"$1\",\"h\",{\"children\":[null,[\"$\",\"$1\",\"SV222zpXkDwAIfDkOwdgA\",{\"children\":[[\"$\",\"$L7\",null,{\"children\":\"$L8\"}],[\"$\",\"$L9\",null,{\"children\":\"$La\"}],null]}]]"])
    </script>
    <script>
      self.__next_f.push([1, "}]]],\"m\":\"$undefined\",\"G\":[\"$b\",\"$undefined\"],\"s\":false,\"S\":true}\n"])
    </script>
    <script>
      self.__next_f.push([1, "4:[\"$\",\"main\",null,{\"children\":[[\"$\",\"h1\",null,{\"children\":\"Zoowee Blubberworth's epic website\"}],[\"$\",\"p\",null,{\"children\":\"No content loaded - database offline\"}]]}]\n"])
    </script>
    <script>
      self.__next_f.push([1, "a:[[\"$\",\"meta\",\"0\",{\"name\":\"viewport\",\"content\":\"width=device-width, initial-scale=1\"}]]\n8:[[\"$\",\"meta\",\"0\",{\"charSet\":\"utf-8\"}]]\n"])
    </script>
    <script>
      self.__next_f.push([1, "6:null\n"])
    </script>
  </body>
</html>
```

Theres not much on the page. It just says:
```shell
Zoowee Blubberworth's epic website

No content loaded - database offline
```

I noticed they are running [Next.js 15.0.4](https://www.wappalyzer.com/technologies/javascript-frameworks/next-js/?utm_source=popup&utm_medium=extension&utm_campaign=wappalyzer) which is vulnerable to [react2shell](https://react2shell.com/) (CVE-2025-55182). I intercepted a normal request with burpsuite and changed the method from `GET` to `POST`. I also need to add some headers to the request. Once my request was ready I ran `ls`:

![Alt text](/images/bushbashweb2.png)

I get the output from my command in the `x-action-redirect` header. I read the file `zoowee_message.txt` to get the flag:

![Alt text](/images/bushbashweb3.png)

`bushbash{youWillNeverCatchMe!IDugATunnelOut}`
