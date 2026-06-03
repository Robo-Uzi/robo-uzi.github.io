---
layout: post
title:  "Misc Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THEM-CTF-2026-part2-misc/
---
* TOC
{:toc}

## Some Play's Libretto

**Category**: Misc

**Author**: smartfella

**Description**: Convert to leetspeak before submitting flag. Example: `THEM?!CTF{50m3_pl41n73x7_6035_h3r3}`

I get the challenge file `score.txt`:
```shell
cat score.txt | head  
Nby Ulcnbgyncw Nluayxs iz nby Vulx'm Mywlyn.  
  
Ligyi, u guh iz fynnylm uhx bcxxyh nlonbm.  
Dofcyn, u eyyjyl iz hogylcwuf gsmnylcym.  
  
  
					Uwn C: Nby Auohnfyn.  
  
					Mwyhy C: Nby Jlifiaoy.
```

I have to rotate by 6 on cyberchef to get plaintext: [cyberchef link](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,6)&input=TmJ5IFVsY25iZ3luY3cgTmx1YXl4cyBpeiBuYnkgVnVseCdtIE15d2x5bi4KCkxpZ3lpLCB1IGd1aCBpeiBmeW5ueWxtIHVoeCBiY3h4eWggbmxvbmJtLgpEb2ZjeW4sIHUgZXl5anlsIGl6IGhvZ3lsY3d1ZiBnc21ueWxjeW0uCgoKICAgICAgICAgICAgICAgICAgICBVd24gQzogTmJ5IEF1b2huZnluLgoKICAgICAgICAgICAgICAgICAgICBNd3loeSBDOiBOYnkgSmxpZmlhb3kuCgpbWWhueWwgTGlneWkgdWh4IERvZmN5bl0KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGJ1ampzIG1xeXluIHF1bGcgamxpb3ggdmx1cHkgYmloeW1uIGxpbXkgdWh4IG5ieSBtb2cgaXogdSBnY2FibnMgZmlwY2hhIHZpZnggYWlpeCBheWhuZnkgemZpcXlsIHVoeCB1IGhpdmZ5IGZpcHlmcyBsY3diIG1vaGhzIGVjaGF4aWchCk5iaW8gdWxuIG5ieSBtb2cgaXogbmJzbXlmeiB1aHggbmJ5IG1vZyBpeiB1IGFpZnh5aCBGaWx4IHVoeCB1IGpmb2chCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgQ0MuCkN6IGhpbiwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUlJDQy4KCiAgICAgICAgICAgICAgICAgICAgTXd5aHkgQ0M6IE5ieSBZaG5sdWh3eS4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGhpdmZ5IGZpcHlmcyB3YnVsZ2NoYSBxdWxnIHZpZnggbGN3YiBlY2hheGlnIHVoeCBuYnkgbW9nIGl6IHUgYWlmeHloIGZpcGNoYSBhaWl4IG1xeXluIGJ1ampzIGpmb2cgdWh4IHUgenVjbCBiaWh5bW4gbW9oaHMgRmlseCEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBDQ0MuCkN6IGhpbiwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUlJDQy4KCiAgICAgICAgICAgICAgICAgICAgTXd5aHkgQ0NDOiBOYnkgQWx5eW5jaGEuCgpEb2ZjeW46Ck5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBhaWZ4eWggbGN3YiB6dWNsIHZsdXB5IGhpdmZ5IG1xeXluIGxpbXkgdWh4IG5ieSBtb2cgaXogdSBiaWh5bW4gbW9oaHMgZ2NhYm5zIHF1bGcgd2J1bGdjaGEgZWNoYXhpZyB1aHggdSB1aGF5ZiEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBDUC4KQ3ogaGluLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUkNDLgoKICAgICAgICAgICAgICAgICAgICBNd3loeSBDUDogTmJ5IFh5d2Z1bHVuY2loLgoKRG9mY3luOgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgYWlpeCB6dWNsIGJ1ampzIGF5aG5meSBtcXl5biB3YnVsZ2NoYSBCeXVweWggdWh4IG5ieSBtb2cgaXogdSBhaWZ4eWggZ2NhYm5zIGpsaW94IHF1bGcgbW9oaHMgRWNoYSB1aHggdSBsY3diIGZpcGNoYSBiaWh5bW4gZWNoYXhpZyEKTmJpbyB1bG4gbmJ5IG1vZyBpeiBuYnNteWZ6IHVoeCBuYnkgbW9nIGl6IHUgdmx1cHkgYnVqamNoeW1tIHVoeCB1IGpmb2chCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUC4KQ3ogaGluLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUkNDLgoKICAgICAgICAgICAgICAgICAgICBNd3loeSBQOiBOYnkgV2loenltbWNpaC4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IG1xeXluIHp1Y2wgaGl2Znkgdmx1cHkgcXVsZyBhaWl4IGJ1ampjaHltbSB1aHggbmJ5IG1vZyBpeiB1IGJ1ampzIGdjYWJucyBiaWh5bW4gZmlwY2hhIHdidWxnY2hhIGpmb2cgdWh4IG5ieSBtb2cgaXogdSBhaWZ4eWggdmlmeCB1aGF5ZiB1aHggdSBGaWx4IQoKTGlneWk6CklqeWggc2lvbCBnY2h4IQoKRG9mY3luOgpVZyBDIHVtIGFpaXggdW0gbmJ5eT8KQ3ogbWksIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFBDLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFBDOiBOYnkgVW1teWxuY2loLgoKRG9mY3luOgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgdmx1cHkgZ2NhYm5zIGFpaXggZmlweWZzIHZpZnggbGN3YiBsaW15IHVoeCBuYnkgbW9nIGl6IHUgcXVsZyB6dWNsIG1xeXluIGJ1ampzIGZpcGNoYSB6ZmlxeWwgdWh4IHUgYXlobmZ5IGpsaW94IG1vaGhzIGFpZnh5aCBFY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IG5ic215ZnogdWh4IG5ieSBtb2cgaXogdSBoaXZmeSBieWxpIHVoeCB1IEZpbHghCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUENDLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFBDQzogTmJ5IEpsaXdmdWd1bmNpaC4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IG1vaGhzIGZpcGNoYSB2aWZ4IHdidWxnY2hhIGpsaW94IGdjYWJucyBFY2hhIHVoeCBuYnkgbW9nIGl6IHUgbGN3YiBhaWl4IGFpZnh5aCBheWhuZnkgYmloeW1uIGJ5bGkgdWh4IHUgZmlweWZzIGJ1ampzIHZsdXB5IHp1Y2wgZGlzIQoKTGlneWk6CklqeWggc2lvbCBnY2h4IQoKRG9mY3luOgpVZyBDIHVtIGFpaXggdW0gbmJ5eT8KQ3ogbWksIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFBDQ0MuCkN6IGhpbiwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUlJDQy4KCiAgICAgICAgICAgICAgICAgICAgTXd5aHkgUENDQzogTmJ5IE9ocHljZmNoYS4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IG1vaGhzIGJ1ampzIGxjd2IgYWlmeHloIGdjYWJucyBheWhuZnkgZGlzIHVoeCBuYnkgbW9nIGl6IHUgcXVsZyB2bHVweSBoaXZmeSBtcXl5biBhaWl4IGJ5bGkgdWh4IG5ieSBtb2cgaXogdSB3YnVsZ2NoYSBmaXB5ZnMgRmlseCB1aHggdSB1aGF5ZiEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBDUi4KQ3ogaGluLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUkNDLgoKICAgICAgICAgICAgICAgICAgICBNd3loeSBDUjogTmJ5IE1vZ2dpaGNoYS4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGFpaXggdmlmeCBnY2FibnMgd2J1bGdjaGEgenVjbCBhaWZ4eWggdWhheWYgdWh4IG5ieSBtb2cgaXogdSBxdWxnIHZsdXB5IGZpcGNoYSBiaWh5bW4gbGN3YiBqZm9nIHVoeCB1IGxpbXkhCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUi4KQ3ogaGluLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUkNDLgoKICAgICAgICAgICAgICAgICAgICBNd3loeSBSOiBOYnkgV2loemxpaG51bmNpaC4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGFpZnh5aCB2aWZ4IG1vaGhzIGJpaHltbiBmaXB5ZnMgZ2NhYm5zIEJ5dXB5aCB1aHggbmJ5IG1vZyBpeiB1IGpsaW94IHp1Y2wgYnVqanMgcXVsZyBheWhuZnkgemZpcXlsIHVoeCBuYnkgbW9nIGl6IHUgbGN3YiBhaWl4IHdidWxnY2hhIG1xeXluIGRpcyB1aHggdSB2bHVweSBFY2hhIQoKTGlneWk6CklqeWggc2lvbCBnY2h4IQoKRG9mY3luOgpVZyBDIHVtIGFpaXggdW0gbmJ5eT8KQ3ogbWksIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJDLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJDOiBOYnkgWHlmY3Z5bHVuY2loLgoKRG9mY3luOgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgbW9oaHMgZ2NhYm5zIGFpZnh5aCBiaWh5bW4gbGN3YiBmaXBjaGEgYnlsaSB1aHggbmJ5IG1vZyBpeiB1IHdidWxnY2hhIGhpdmZ5IHF1bGcgYXlobmZ5IGpsaW94IEZpbHggdWh4IG5ieSBtb2cgaXogdSB2aWZ4IHp1Y2wgYnVqamNoeW1tIHVoeCB1IGpmb2chCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUkNDLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJDQzogTmJ5IEp1b215LgoKRG9mY3luOgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiB1IHdidWxnY2hhIGpsaW94IGdjYWJucyBidWpqcyBsY3diIHpmaXF5bCEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSQ0NDLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJDQ0M6IE5ieSBYeWp1bG5vbHkuCgpEb2ZjeW46Ck5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBmaXB5ZnMgbW9oaHMgYnVqanMgZmlwY2hhIGhpdmZ5IGJpaHltbiBqZm9nIHVoeCBuYnkgbW9nIGl6IHUgd2J1bGdjaGEgdmlmeCBxdWxnIHp1Y2wgbGN3YiBFY2hhIHVoeCBuYnkgbW9nIGl6IHUgdmx1cHkgZ2NhYm5zIGJ5bGkgdWh4IHUgYWlpeCBkaXMhCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUkNQLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJDUDogTmJ5IEp1bW11YXkuCgpEb2ZjeW46Ck5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSB6dWNsIGF5aG5meSB3YnVsZ2NoYSBqbGlveCBmaXBjaGEgdmx1cHkgYnVqamNoeW1tIHVoeCBuYnkgbW9nIGl6IHUgYWlpeCBtcXl5biBsY3diIHZpZnggaGl2ZnkgRWNoYSB1aHggbmJ5IG1vZyBpeiB1IHF1bGcgbW9oaHMgZmlweWZzIGdjYWJucyBlY2hheGlnIHVoeCB1IGJpaHltbiB6ZmlxeWwhCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUlAuCkN6IGhpbiwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUlJDQy4KCiAgICAgICAgICAgICAgICAgICAgTXd5aHkgUlA6IE5ieSBXbGltbWNoYS4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IHp1Y2wgYXlobmZ5IGJpaHltbiBmaXBjaGEgdmlmeCBmaXB5ZnMgdWhheWYgdWh4IG5ieSBtb2cgaXogdSB2bHVweSBnY2FibnMgamxpb3ggaGl2ZnkgcXVsZyBqZm9nIHVoeCBuYnkgbW9nIGl6IHUgYnVqanMgbW9oaHMgYWlmeHloIEZpbHggdWh4IHUgbXF5eW4gd2J1bGdjaGEgemZpcXlsIQpOYmlvIHVsbiBuYnkgbW9nIGl6IG5ic215ZnogdWh4IG5ieSBtb2cgaXogdSBsY3diIGxpbXkgdWh4IHUgYnlsaSEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUEMuCkN6IGhpbiwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUlJDQy4KCiAgICAgICAgICAgICAgICAgICAgTXd5aHkgUlBDOiBOYnkgWHltd3lobi4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IG1vaGhzIGdjYWJucyBheWhuZnkgYmloeW1uIG1xeXluIGxjd2IgQnl1cHloIHVoeCBuYnkgbW9nIGl6IHUgd2J1bGdjaGEgenVjbCBqbGlveCBxdWxnIGhpdmZ5IEZpbHggdWh4IHUgdmx1cHkgYnVqanMgdmlmeCBlY2hheGlnIQpOYmlvIHVsbiBuYnkgbW9nIGl6IG5ic215ZnogdWh4IG5ieSBtb2cgaXogdSBhaWl4IGZpcHlmcyBieWxpIHVoeCB1IHpmaXF5bCEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUENDLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJQQ0M6IE5ieSBNY2Z5aHd5LgoKRG9mY3luOgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiB1IGdjYWJucyBmaXBjaGEgYnVqanMgaGl2ZnkgbW9oaHMgRWNoYSEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUENDQy4KQ3ogaGluLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUkNDLgoKICAgICAgICAgICAgICAgICAgICBNd3loeSBSUENDQzogTmJ5IFVsbGNwdWYuCgpEb2ZjeW46Ck5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBtcXl5biBoaXZmeSBhaWl4IHZsdXB5IHdidWxnY2hhIGZpcGNoYSBCeXVweWggdWh4IG5ieSBtb2cgaXogdSBmaXB5ZnMgYnVqanMgbGN3YiBhaWZ4eWggYXlobmZ5IEVjaGEgdWh4IG5ieSBtb2cgaXogdSB6dWNsIHF1bGcgZ2NhYm5zIHZpZnggamZvZyB1aHggdSBqbGlveCBtb2hocyB6ZmlxeWwhCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUkNSLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJDUjogTmJ5IFVxdWV5aGNoYS4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGpsaW94IGxjd2IgenVjbCBiaWh5bW4gcXVsZyB2bHVweSBqZm9nIHVoeCBuYnkgbW9nIGl6IHUgdmlmeCBidWpqcyBmaXBjaGEgYWlmeHloIG1xeXluIGJ5bGkgdWh4IG5ieSBtb2cgaXogdSBmaXB5ZnMgaGl2ZnkgRmlseCB1aHggdSB1aGF5ZiEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUi4KQ3ogaGluLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUkNDLgoKICAgICAgICAgICAgICAgICAgICBNd3loeSBSUjogTmJ5IEx5d2VpaGNoYS4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGZpcHlmcyBhaWl4IHZsdXB5IGdjYWJucyBheWhuZnkgcXVsZyBidWpqY2h5bW0gdWh4IG5ieSBtb2cgaXogdSB6dWNsIGpsaW94IGJ1ampzIG1vaGhzIG1xeXluIHVoYXlmIHVoeCB1IGxjd2Igd2J1bGdjaGEgaGl2ZnkgRmlseCEKTmJpbyB1bG4gbmJ5IG1vZyBpeiBuYnNteWZ6IHVoeCBuYnkgbW9nIGl6IHUgdmlmeCBmaXBjaGEgZGlzIHVoeCB1IGpmb2chCgpMaWd5aToKSWp5aCBzaW9sIGdjaHghCgpEb2ZjeW46ClVnIEMgdW0gYWlpeCB1bSBuYnl5PwpDeiBtaSwgZnluIG9tIGpsaXd5eXggbmkgbXd5aHkgUlJDLgpDeiBoaW4sIGZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ0MuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJSQzogTmJ5IFdvZmdjaHVuY2loLgoKRG9mY3luOgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgdmx1cHkgd2J1bGdjaGEgZ2NhYm5zIGJpaHltbiBmaXB5ZnMgamxpb3ggZGlzIHVoeCBuYnkgbW9nIGl6IHUgYXlobmZ5IG1vaGhzIHZpZnggaGl2ZnkgbXF5eW4gYnlsaSB1aHggdSBhaWZ4eWggZmlwY2hhIGJ1ampzIHp1Y2wgdWhheWYhCk5iaW8gdWxuIG5ieSBtb2cgaXogbmJzbXlmeiB1aHggbmJ5IG1vZyBpeiB1IHF1bGcgYWlpeCB6ZmlxeWwgdWh4IHUgRWNoYSEKCkxpZ3lpOgpJanloIHNpb2wgZ2NoeCEKCkRvZmN5bjoKVWcgQyB1bSBhaWl4IHVtIG5ieXk/CkN6IG1pLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUkNDQy4KQ3ogaGluLCBmeW4gb20gamxpd3l5eCBuaSBtd3loeSBSUkNDLgoKICAgICAgICAgICAgICAgICAgICBNd3loeSBSUkNDOiBOYnkgT2hxaWxuYnMuCgpEb2ZjeW46CkZ5biBvbSBqbGl3eXl4IG5pIG13eWh5IFJSQ1AuCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJSQ0NDOiBOYnkgTHlweWZ1bmNpaC4KCkRvZmN5bjoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IHZsdXB5IG1xeXluIHZpZnggYWlpeCBxdWxnIGFpZnh5aCBlY2hheGlnIHVoeCBuYnkgbW9nIGl6IHUgaGl2ZnkgZ2NhYm5zIGZpcHlmcyB3YnVsZ2NoYSBheWhuZnkgbGlteSB1aHggdSBmaXBjaGEgYnVqamNoeW1tIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSB3YnVsZ2NoYSBheWhuZnkgZmlwY2hhIHZsdXB5IGJ1ampzIGpsaW94IGVjaGF4aWcgdWh4IG5ieSBtb2cgaXogdSBtb2hocyB2aWZ4IGdjYWJucyBsY3diIHF1bGcgZGlzIHVoeCBuYnkgbW9nIGl6IHUgYWlpeCBtcXl5biBiaWh5bW4gZmlweWZzIGJ5bGkgdWh4IHUgenVjbCBFY2hhIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBtb2hocyB3YnVsZ2NoYSBidWpqcyB2aWZ4IGhpdmZ5IHF1bGcgYnlsaSB1aHggbmJ5IG1vZyBpeiB1IGF5aG5meSBqbGlveCB6dWNsIGFpZnh5aCBiaWh5bW4gbGlteSB1aHggbmJ5IG1vZyBpeiB1IGxjd2IgZmlwY2hhIGFpaXggZGlzIHVoeCB1IGdjYWJucyB2bHVweSB1aGF5ZiEKTmJpbyB1bG4gbmJ5IG1vZyBpeiBuYnNteWZ6IHVoeCBuYnkgbW9nIGl6IHUgZmlweWZzIGVjaGF4aWcgdWh4IHUgRmlseCEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiB1IG1xeXluIGFpaXggYnVqanMgbGN3YiBtb2hocyBCeXVweWghCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IHdidWxnY2hhIGhpdmZ5IGFpaXggbXF5eW4gdmlmeCB6dWNsIHVoYXlmIHVoeCBuYnkgbW9nIGl6IHUgbGN3YiBheWhuZnkgYWlmeHloIGpsaW94IGJ1ampzIGJ5bGkgdWh4IHUgZmlwY2hhIHZsdXB5IGZpcHlmcyBqZm9nIQpOYmlvIHVsbiBuYnkgbW9nIGl6IG5ic215ZnogdWh4IG5ieSBtb2cgaXogdSBnY2FibnMgYmloeW1uIGxpbXkgdWh4IHUgQnl1cHloIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBmaXB5ZnMgZmlwY2hhIGFpZnh5aCBqbGlveCBtb2hocyBheWhuZnkgamZvZyB1aHggbmJ5IG1vZyBpeiB1IGJ1ampzIHZpZnggcXVsZyBsY3diIHdidWxnY2hhIGxpbXkgdWh4IG5ieSBtb2cgaXogdSBhaWl4IHp1Y2wgaGl2ZnkgYnlsaSB1aHggdSBlY2hheGlnIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBoaXZmeSBqbGlveCBxdWxnIGZpcGNoYSBtcXl5biBiaWh5bW4gemZpcXlsIHVoeCBuYnkgbW9nIGl6IHUgYXlobmZ5IGFpaXggdmlmeCBhaWZ4eWggbW9oaHMgdWhheWYgdWh4IHUgdmx1cHkgd2J1bGdjaGEgRWNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiBuYnNteWZ6IHVoeCBuYnkgbW9nIGl6IHUgbGN3YiBqZm9nIHVoeCB1IGVjaGF4aWchCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGJpaHltbiBtcXl5biB3YnVsZ2NoYSBxdWxnIHp1Y2wgbGN3YiBkaXMgdWh4IG5ieSBtb2cgaXogdSBqbGlveCB2bHVweSBhaWl4IGJ1ampzIGF5aG5meSBCeXVweWggdWh4IHUgaGl2ZnkgbW9oaHMgZmlwY2hhIGpmb2chCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IHZpZnggenVjbCB2bHVweSBiaWh5bW4gZmlweWZzIGxjd2IgbGlteSB1aHggbmJ5IG1vZyBpeiB1IGpsaW94IHdidWxnY2hhIHF1bGcgbXF5eW4gZ2NhYm5zIHVoYXlmIHVoeCBuYnkgbW9nIGl6IHUgYnVqanMgYWlmeHloIGF5aG5meSBmaXBjaGEgemZpcXlsIHVoeCB1IGFpaXggaGl2ZnkgRWNoYSEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiB1IGJpaHltbiBnY2FibnMgYWlpeCBtb2hocyB3YnVsZ2NoYSB6ZmlxeWwhCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGpsaW94IHF1bGcgbXF5eW4gZ2NhYm5zIGFpaXggYWlmeHloIGpmb2cgdWh4IG5ieSBtb2cgaXogdSBsY3diIGJpaHltbiBheWhuZnkgd2J1bGdjaGEgZmlweWZzIGRpcyB1aHggdSBieWxpIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBiaWh5bW4gbGN3YiBidWpqcyB3YnVsZ2NoYSB2bHVweSBoaXZmeSBieWxpIHVoeCBuYnkgbW9nIGl6IHUgdmlmeCBqbGlveCBtb2hocyBmaXBjaGEgZmlweWZzIGxpbXkgdWh4IG5ieSBtb2cgaXogdSBheWhuZnkgZGlzIHVoeCB1IHVoYXlmIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSB2aWZ4IG1xeXluIGFpZnh5aCBnY2FibnMgcXVsZyBhaWl4IGJ1ampjaHltbSB1aHggbmJ5IG1vZyBpeiB1IGZpcHlmcyBsY3diIGpsaW94IHp1Y2wgYnVqanMgZGlzIHVoeCBuYnkgbW9nIGl6IHUgYXlobmZ5IGZpcGNoYSBoaXZmeSB3YnVsZ2NoYSBCeXVweWggdWh4IHUgYmloeW1uIG1vaGhzIEZpbHghCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGhpdmZ5IHdidWxnY2hhIHF1bGcgenVjbCB2aWZ4IGdjYWJucyBieWxpIHVoeCBuYnkgbW9nIGl6IHUgbGN3YiBhaWZ4eWggdmx1cHkgbXF5eW4gbW9oaHMgdWhheWYgdWh4IHUgamxpb3ggYWlpeCBidWpqcyBiaWh5bW4gbGlteSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiBuYnNteWZ6IHVoeCBuYnkgbW9nIGl6IHUgZmlwY2hhIGF5aG5meSBidWpqY2h5bW0gdWh4IHUgemZpcXlsIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBmaXBjaGEgYnVqanMgaGl2ZnkgbXF5eW4gYWlpeCBiaWh5bW4gbGlteSB1aHggbmJ5IG1vZyBpeiB1IGpsaW94IGxjd2IgZ2NhYm5zIG1vaGhzIHZsdXB5IGJ5bGkgdWh4IHUgZGlzIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBoaXZmeSBnY2FibnMgenVjbCBmaXBjaGEgd2J1bGdjaGEgYnVqanMgemZpcXlsIHVoeCBuYnkgbW9nIGl6IHUgcXVsZyBiaWh5bW4gamxpb3ggdmx1cHkgbGN3YiBFY2hhIHVoeCBuYnkgbW9nIGl6IHUgdmlmeCBheWhuZnkgZmlweWZzIHVoYXlmIHVoeCB1IGFpaXggbW9oaHMgbGlteSEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgbW9oaHMgYXlobmZ5IGJpaHltbiBmaXB5ZnMgZ2NhYm5zIHZpZnggdWhheWYgdWh4IG5ieSBtb2cgaXogdSB6dWNsIGFpaXggaGl2ZnkgYnVqanMgd2J1bGdjaGEgYnlsaSB1aHggbmJ5IG1vZyBpeiB1IG1xeXluIGFpZnh5aCBmaXBjaGEgemZpcXlsIHVoeCB1IHZsdXB5IHF1bGcgbGlteSEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgZmlweWZzIGZpcGNoYSB6dWNsIGJpaHltbiBsY3diIG1xeXluIHpmaXF5bCB1aHggbmJ5IG1vZyBpeiB1IHF1bGcgbW9oaHMgd2J1bGdjaGEgaGl2ZnkgYWlmeHloIHVoYXlmIHVoeCB1IGpsaW94IGJ1ampzIHZsdXB5IGdjYWJucyBlY2hheGlnIQpOYmlvIHVsbiBuYnkgbW9nIGl6IG5ic215ZnogdWh4IG5ieSBtb2cgaXogdSB2aWZ4IGFpaXggYXlobmZ5IGxpbXkgdWh4IHUgRmlseCEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiB1IGZpcHlmcyBtcXl5biB2aWZ4IHdidWxnY2hhIGJpaHltbiBidWpqY2h5bW0hCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IHF1bGcgYnVqanMgYWlpeCBiaWh5bW4gbXF5eW4gbW9oaHMgYnVqamNoeW1tIHVoeCBuYnkgbW9nIGl6IHUgamxpb3ggYWlmeHloIHZsdXB5IGZpcHlmcyBmaXBjaGEgdWhheWYgdWh4IHUgbGN3YiBFY2hhIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSBoaXZmeSBtcXl5biB6dWNsIHF1bGcgamxpb3ggZ2NhYm5zIGJ1ampjaHltbSB1aHggbmJ5IG1vZyBpeiB1IHZpZnggd2J1bGdjaGEgdmx1cHkgYnVqanMgbW9oaHMgdWhheWYgdWh4IG5ieSBtb2cgaXogdSBmaXBjaGEgYWlpeCBlY2hheGlnIHVoeCB1IEZpbHghCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gdSB3YnVsZ2NoYSBxdWxnIG1vaGhzIGhpdmZ5IHp1Y2wgYnlsaSEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgenVjbCBxdWxnIGJ1ampzIGdjYWJucyBmaXB5ZnMgYXlobmZ5IHpmaXF5bCB1aHggbmJ5IG1vZyBpeiB1IG1vaGhzIGhpdmZ5IHZsdXB5IHdidWxnY2hhIGZpcGNoYSBlY2hheGlnIHVoeCB1IGFpaXggYmloeW1uIGxjd2IgYWlmeHloIEJ5dXB5aCEKTmJpbyB1bG4gbmJ5IG1vZyBpeiBuYnNteWZ6IHVoeCBuYnkgbW9nIGl6IHUgdmlmeCBFY2hhIHVoeCB1IGpmb2chCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGJpaHltbiBidWpqcyB2aWZ4IHZsdXB5IGF5aG5meSB6dWNsIHVoYXlmIHVoeCBuYnkgbW9nIGl6IHUgZmlweWZzIHF1bGcgbW9oaHMgZmlwY2hhIGFpaXggRmlseCB1aHggdSBtcXl5biBqbGlveCBoaXZmeSBFY2hhIQpNanl1ZSBuYnMgZ2NoeCEKCk5iaW8gdWxuIGhpbmJjaGEhCk5iaW8gdWxuIG5ieSBtb2cgaXogdSB6dWNsIGhpdmZ5IGJpaHltbiBmaXBjaGEgd2J1bGdjaGEgZmlweWZzIHpmaXF5bCB1aHggbmJ5IG1vZyBpeiB1IGF5aG5meSB2bHVweSBtb2hocyBnY2FibnMgbXF5eW4gYnVqamNoeW1tIHVoeCB1IEZpbHghCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGJpaHltbiBtb2hocyB2aWZ4IGZpcHlmcyBidWpqcyBtcXl5biBqZm9nIHVoeCBuYnkgbW9nIGl6IHUgcXVsZyBoaXZmeSBmaXBjaGEgenVjbCBhaWZ4eWggbGlteSB1aHggdSB2bHVweSBnY2FibnMgamxpb3ggYnlsaSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiBuYnNteWZ6IHVoeCBuYnkgbW9nIGl6IHUgd2J1bGdjaGEgRWNoYSB1aHggdSBkaXMhCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGFpZnh5aCBnY2FibnMgdmlmeCBmaXB5ZnMgZmlwY2hhIGFpaXggemZpcXlsIHVoeCBuYnkgbW9nIGl6IHUgYnVqanMgbGN3YiBheWhuZnkgenVjbCB2bHVweSBFY2hhIHVoeCBuYnkgbW9nIGl6IHUgcXVsZyBqbGlveCBidWpqY2h5bW0gdWh4IHUgamZvZyEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgYWlmeHloIGhpdmZ5IGxjd2Igdmx1cHkgenVjbCBmaXB5ZnMgamZvZyB1aHggbmJ5IG1vZyBpeiB1IG1xeXluIGJ1ampzIHF1bGcgamxpb3ggYmloeW1uIGxpbXkgdWh4IHUgd2J1bGdjaGEgZ2NhYm5zIG1vaGhzIGZpcGNoYSBCeXVweWghCk5iaW8gdWxuIG5ieSBtb2cgaXogbmJzbXlmeiB1aHggbmJ5IG1vZyBpeiB1IHZpZnggRWNoYSB1aHggdSB6ZmlxeWwhCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IGFpZnh5aCBoaXZmeSBmaXB5ZnMgZmlwY2hhIGFpaXggdmlmeCBCeXVweWggdWh4IG5ieSBtb2cgaXogdSBqbGlveCB2bHVweSBtcXl5biBnY2FibnMgenVjbCBidWpqY2h5bW0gdWh4IHUgd2J1bGdjaGEgbGN3YiBiaWh5bW4gYnVqanMgbGlteSEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgYWlpeCBmaXBjaGEgYnVqanMgYWlmeHloIGxjd2IgZmlweWZzIHVoYXlmIHVoeCBuYnkgbW9nIGl6IHUgbXF5eW4gbW9oaHMgaGl2ZnkgenVjbCBqbGlveCBieWxpIHVoeCBuYnkgbW9nIGl6IHUgdmlmeCB2bHVweSBlY2hheGlnIHVoeCB1IEVjaGEhCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IHZpZnggYWlmeHloIHp1Y2wgdmx1cHkgbW9oaHMgYnVqanMgZGlzIHVoeCBuYnkgbW9nIGl6IHUgYmloeW1uIGZpcGNoYSBnY2FibnMgZmlweWZzIG1xeXluIEZpbHggdWh4IHUgdWhheWYhCk1qeXVlIG5icyBnY2h4IQoKTmJpbyB1bG4gaGluYmNoYSEKTmJpbyB1bG4gbmJ5IG1vZyBpeiB1IG1vaGhzIHZsdXB5IGF5aG5meSBsY3diIGdjYWJucyBoaXZmeSBsaW15IHVoeCBuYnkgbW9nIGl6IHUgd2J1bGdjaGEgZmlweWZzIHF1bGcgenVjbCBidWpqcyB1aGF5ZiB1aHggbmJ5IG1vZyBpeiB1IGFpaXggYmloeW1uIGZpcGNoYSBhaWZ4eWggQnl1cHloIHVoeCB1IHZpZnggRmlseCEKTWp5dWUgbmJzIGdjaHghCgpOYmlvIHVsbiBoaW5iY2hhIQpOYmlvIHVsbiBuYnkgbW9nIGl6IHUgdmx1cHkgZmlwY2hhIGxjd2IgbW9oaHMgenVjbCBhaWl4IGJ1ampjaHltbSB1aHggbmJ5IG1vZyBpeiB1IGF5aG5meSBhaWZ4eWggd2J1bGdjaGEgYnVqanMgdmlmeCBGaWx4IHVoeCBuYnkgbW9nIGl6IHUgamxpb3ggYmloeW1uIGVjaGF4aWcgdWh4IHUgbGlteSEKTWp5dWUgbmJzIGdjaHghCgogICAgICAgICAgICAgICAgICAgIE13eWh5IFJSQ1A6IE5ieSBZcnlvaG4uCgpbWXJ5b2huXQo)

Plaintext contents:
```shell
cat ctf.spl | head  
The Arithmetic Tragedy of the Bard's Secret.  
  
Romeo, a man of letters and hidden truths.  
Juliet, a keeper of numerical mysteries.  
  
  
					Act I: The Gauntlet.  
  
					Scene I: The Prologue.
```

```shell
cat ctf.spl | wc -l  
463
```

I get 463 lines of the Shakespeare Programming Language, an esoteric language with source code that looks like William Shakespeare's plays. [https://shakespearelang.com/1.0/](https://shakespearelang.com/1.0/)

I run the debugger:
```shell
shakespeare debug ctf.spl  
Enter Romeo, Juliet  
Romeo set to 0  
Romeo set to 112  
Romeo set to 115  
Taking input character: A  
Setting Juliet to input value 65  
Setting global boolean to False  
Not jumping to Scene II because global boolean is False  
Jumping to Scene XXII  
Jumping to Scene XXIV  
Exeunt all
```

Scenes `I` through `XXI` each do this:
- Juliet sets Romeo to a number using adjectives and nouns established in SPL ("a happy sweet warm proud brave honest rose")
- Romeo says `Open your mind!` which reads one character from my input, and its ASCII value much equal Romeos current number. 
- If correct you move to the next scene

Once I input the full correct string (`shakespeare from temu`), it goes on to scene `XXIII` and outputs the flag which needs mapped to leetspeak. Debugger output:
```shell
shakespeare debug ctf.spl  
Enter Romeo, Juliet  
Romeo set to 0  
Romeo set to 112  
Romeo set to 115  
Taking input character: s  
Setting Juliet to input value 115  
Setting global boolean to True  
Jumping to Scene II  
Romeo set to 0  
Romeo set to 104  
Taking input character: h  
Setting Juliet to input value 104  
Setting global boolean to True  
Jumping to Scene III  
Romeo set to 0  
Romeo set to 97  
Taking input character: a  
Setting Juliet to input value 97  
Setting global boolean to True  
Jumping to Scene IV  
Romeo set to 0  
Romeo set to 104  
Romeo set to 107  
Taking input character: k  
Setting Juliet to input value 107  
Setting global boolean to True  
Jumping to Scene V  
Romeo set to 0  
Romeo set to 101  
Taking input character: e  
Setting Juliet to input value 101  
Setting global boolean to True  
Jumping to Scene VI  
Romeo set to 0  
Romeo set to 112  
Romeo set to 115  
Taking input character: s  
Setting Juliet to input value 115  
Setting global boolean to True  
Jumping to Scene VII  
Romeo set to 0  
Romeo set to 112  
Taking input character: p  
Setting Juliet to input value 112  
Setting global boolean to True  
Jumping to Scene VIII  
Romeo set to 0  
Romeo set to 101  
Taking input character: e  
Setting Juliet to input value 101  
Setting global boolean to True  
Jumping to Scene IX  
Romeo set to 0  
Romeo set to 97  
Taking input character: a  
Setting Juliet to input value 97  
Setting global boolean to True  
Jumping to Scene X  
Romeo set to 0  
Romeo set to 114  
Taking input character: r  
Setting Juliet to input value 114  
Setting global boolean to True  
Jumping to Scene XI  
Romeo set to 0  
Romeo set to 101  
Taking input character: e  
Setting Juliet to input value 101  
Setting global boolean to True  
Jumping to Scene XII  
Romeo set to 0  
Romeo set to 32  
Taking input character:  
Setting Juliet to input value 32  
Setting global boolean to True  
Jumping to Scene XIII  
Romeo set to 0  
Romeo set to 102  
Taking input character: f  
Setting Juliet to input value 102  
Setting global boolean to True  
Jumping to Scene XIV  
Romeo set to 0  
Romeo set to 114  
Taking input character: r  
Setting Juliet to input value 114  
Setting global boolean to True  
Jumping to Scene XV  
Romeo set to 0  
Romeo set to 108  
Romeo set to 111  
Taking input character: o  
Setting Juliet to input value 111  
Setting global boolean to True  
Jumping to Scene XVI  
Romeo set to 0  
Romeo set to 104  
Romeo set to 109  
Taking input character: m  
Setting Juliet to input value 109  
Setting global boolean to True  
Jumping to Scene XVII  
Romeo set to 0  
Romeo set to 32  
Taking input character:  
Setting Juliet to input value 32  
Setting global boolean to True  
Jumping to Scene XVIII  
Romeo set to 0  
Romeo set to 116  
Taking input character: t  
Setting Juliet to input value 116  
Setting global boolean to True  
Jumping to Scene XIX  
Romeo set to 0  
Romeo set to 101  
Taking input character: e  
Setting Juliet to input value 101  
Setting global boolean to True  
Jumping to Scene XX  
Romeo set to 0  
Romeo set to 104  
Romeo set to 109  
Taking input character: m  
Setting Juliet to input value 109  
Setting global boolean to True  
Jumping to Scene XXI  
Romeo set to 0  
Romeo set to 112  
Romeo set to 117  
Taking input character: u  
Setting Juliet to input value 117  
Setting global boolean to True  
Jumping to Scene XXIII  
Romeo set to 0  
Romeo set to 98  
Outputting Romeo  
Outputting character: 'b'  
Romeo set to 0  
Romeo set to 114  
Outputting Romeo  
Outputting character: 'r'  
Romeo set to 0  
Romeo set to 108  
Romeo set to 111  
Outputting Romeo  
Outputting character: 'o'  
Romeo set to 0  
Romeo set to 32  
Outputting Romeo  
Outputting character: ' '  
Romeo set to 0  
Romeo set to 104  
Romeo set to 109  
Outputting Romeo  
Outputting character: 'm'  
Romeo set to 0  
Romeo set to 105  
Outputting Romeo  
Outputting character: 'i'  
Romeo set to 0  
Romeo set to 100  
Romeo set to 103  
Outputting Romeo  
Outputting character: 'g'  
Romeo set to 0  
Romeo set to 104  
Outputting Romeo  
Outputting character: 'h'  
Romeo set to 0  
Romeo set to 116  
Outputting Romeo  
Outputting character: 't'  
Romeo set to 0  
Romeo set to 32  
Outputting Romeo  
Outputting character: ' '  
Romeo set to 0  
Romeo set to 97  
Outputting Romeo  
Outputting character: 'a'  
Romeo set to 0  
Romeo set to 99  
Outputting Romeo  
Outputting character: 'c'  
Romeo set to 0  
Romeo set to 116  
Outputting Romeo  
Outputting character: 't'  
Romeo set to 0  
Romeo set to 112  
Romeo set to 117  
Outputting Romeo  
Outputting character: 'u'  
Romeo set to 0  
Romeo set to 97  
Outputting Romeo  
Outputting character: 'a'  
Romeo set to 0  
Romeo set to 108  
Outputting Romeo  
Outputting character: 'l'  
Romeo set to 0  
Romeo set to 108  
Outputting Romeo  
Outputting character: 'l'  
Romeo set to 0  
Romeo set to 112  
Romeo set to 121  
Outputting Romeo  
Outputting character: 'y'  
Romeo set to 0  
Romeo set to 32  
Outputting Romeo  
Outputting character: ' '  
Romeo set to 0  
Romeo set to 98  
Outputting Romeo  
Outputting character: 'b'  
Romeo set to 0  
Romeo set to 101  
Outputting Romeo  
Outputting character: 'e'  
Romeo set to 0  
Romeo set to 32  
Outputting Romeo  
Outputting character: ' '  
Romeo set to 0  
Romeo set to 112  
Romeo set to 115  
Outputting Romeo  
Outputting character: 's'  
Romeo set to 0  
Romeo set to 104  
Outputting Romeo  
Outputting character: 'h'  
Romeo set to 0  
Romeo set to 97  
Outputting Romeo  
Outputting character: 'a'  
Romeo set to 0  
Romeo set to 104  
Romeo set to 107  
Outputting Romeo  
Outputting character: 'k'  
Romeo set to 0  
Romeo set to 101  
Outputting Romeo  
Outputting character: 'e'  
Romeo set to 0  
Romeo set to 112  
Romeo set to 115  
Outputting Romeo  
Outputting character: 's'  
Romeo set to 0  
Romeo set to 112  
Outputting Romeo  
Outputting character: 'p'  
Romeo set to 0  
Romeo set to 101  
Outputting Romeo  
Outputting character: 'e'  
Romeo set to 0  
Romeo set to 97  
Outputting Romeo  
Outputting character: 'a'  
Romeo set to 0  
Romeo set to 114  
Outputting Romeo  
Outputting character: 'r'  
Romeo set to 0  
Romeo set to 101  
Outputting Romeo  
Outputting character: 'e'  
Exeunt all
```

`THEM?!CTF{br0_m16h7_4c7u4lly_b3_5h4k35p34r3}`

___

## Git-Art

**Category**: Misc

**Author**: tegeda

**Description**: i got this repo from a famous painter, he said to search for the flag where it's been buried.

I download the challenge file `Git-art.zip` and unzip it to get the repo:
```shell
unzip Git-art.zip  
Archive:  Git-art.zip  
   creating: Git-art/  
   creating: Git-art/git/  
   creating: Git-art/git/.git/  
   creating: Git-art/git/.git/info/  
  inflating: Git-art/git/.git/info/exclude  
  inflating: Git-art/git/.git/description  
   creating: Git-art/git/.git/hooks/  
  inflating: Git-art/git/.git/hooks/pre-commit.sample  
  inflating: Git-art/git/.git/hooks/pre-push.sample  
  inflating: Git-art/git/.git/hooks/post-update.sample  
  inflating: Git-art/git/.git/hooks/push-to-checkout.sample  
  inflating: Git-art/git/.git/hooks/fsmonitor-watchman.sample  
  inflating: Git-art/git/.git/hooks/sendemail-validate.sample  
  inflating: Git-art/git/.git/hooks/commit-msg.sample  
  inflating: Git-art/git/.git/hooks/pre-applypatch.sample  
  inflating: Git-art/git/.git/hooks/pre-merge-commit.sample  
  inflating: Git-art/git/.git/hooks/pre-receive.sample  
  inflating: Git-art/git/.git/hooks/applypatch-msg.sample  
  inflating: Git-art/git/.git/hooks/prepare-commit-msg.sample  
  inflating: Git-art/git/.git/hooks/pre-rebase.sample  
  inflating: Git-art/git/.git/hooks/update.sample  
   creating: Git-art/git/.git/refs/  
   creating: Git-art/git/.git/refs/heads/  
  inflating: Git-art/git/.git/refs/heads/main  
   creating: Git-art/git/.git/refs/tags/  
   creating: Git-art/git/.git/refs/original/  
  inflating: Git-art/git/.git/HEAD  
   creating: Git-art/git/.git/objects/  
   creating: Git-art/git/.git/objects/pack/  
   creating: Git-art/git/.git/objects/info/  
   creating: Git-art/git/.git/objects/e3/  
  inflating: Git-art/git/.git/objects/e3/999f30f9d48f653015b37d650065cb28902533

... etc
```

Visible history is not helpful:
```shell
$git log  
commit ffa9de241623634375c14dbb61681517d78fb46f (HEAD -> main)  
Author: Ruben Cantasano <t3j3da@intesasanpaolo.com>  
Date:   Sun May 10 21:51:26 2026 +0200  
  
	Initial commit - The canvas is empty.  

git show ffa9de241623634375c14dbb61681517d78fb46f  
commit ffa9de241623634375c14dbb61681517d78fb46f (HEAD -> main)  
Author: Ruben Cantasano <t3j3da@intesasanpaolo.com>  
Date:   Sun May 10 21:51:26 2026 +0200  
  
	Initial commit - The canvas is empty.  
  
diff --git a/README.md b/README.md  
new file mode 100644  
index 0000000..1734870  
--- /dev/null  
+++ b/README.md  
@@ -0,0 +1,3 @@  
+# Git Art Challenge  
+  
+Find the lost painting. The artist left no traces on any branch.
```

The `.git/lost-found/commit/` directory contains 2 commits which both share the same tree containing `data.txt`. 

`data.txt` is the painting. It contains commits spanning 2025-2029. They map to cells on a GitHub contribution graph. Each week is a column (left > right) and each day is a row (Sun > Sat, top > bottom).

I render the grid by running this python:
```python
from datetime import date, timedelta

dates_raw = """2025-01-05
2025-01-12
2025-01-19
2025-01-20
2025-01-21
2025-01-22
2025-01-23
2025-01-24
2025-01-25
2025-01-26
2025-02-02
2025-02-16
2025-02-17
2025-02-18
2025-02-19
2025-02-20
2025-02-21
2025-02-22
2025-02-26
2025-03-05
2025-03-12
2025-03-16
2025-03-17
2025-03-18
2025-03-19
2025-03-20
2025-03-21
2025-03-22
2025-03-30
2025-03-31
2025-04-01
2025-04-02
2025-04-03
2025-04-04
2025-04-05
2025-04-06
2025-04-09
2025-04-12
2025-04-13
2025-04-16
2025-04-19
2025-04-20
2025-04-23
2025-04-26
2025-05-11
2025-05-12
2025-05-13
2025-05-14
2025-05-15
2025-05-16
2025-05-17
2025-05-18
2025-05-19
2025-05-26
2025-05-27
2025-06-01
2025-06-02
2025-06-08
2025-06-09
2025-06-10
2025-06-11
2025-06-12
2025-06-13
2025-06-14
2025-06-29
2025-07-06
2025-07-08
2025-07-09
2025-07-10
2025-07-12
2025-07-13
2025-07-15
2025-07-20
2025-07-21
2025-07-22
2025-08-17
2025-08-18
2025-08-19
2025-08-20
2025-08-21
2025-08-23
2025-09-15
2025-09-16
2025-09-17
2025-09-18
2025-09-19
2025-09-21
2025-09-22
2025-09-26
2025-09-27
2025-09-28
2025-10-04
2025-10-05
2025-10-11
2025-10-12
2025-10-18
2025-10-26
2025-11-02
2025-11-09
2025-11-10
2025-11-11
2025-11-12
2025-11-13
2025-11-14
2025-11-15
2025-11-16
2025-11-23
2025-12-09
2025-12-10
2025-12-11
2025-12-12
2025-12-13
2025-12-16
2025-12-23
2025-12-24
2025-12-25
2025-12-26
2025-12-30
2026-01-06
2026-01-22
2026-01-23
2026-01-28
2026-02-03
2026-02-11
2026-02-19
2026-02-20
2026-03-01
2026-03-08
2026-03-15
2026-03-16
2026-03-17
2026-03-18
2026-03-19
2026-03-20
2026-03-21
2026-03-22
2026-03-29
2026-04-12
2026-04-13
2026-04-14
2026-04-15
2026-04-16
2026-04-17
2026-04-18
2026-04-22
2026-04-29
2026-05-06
2026-05-10
2026-05-11
2026-05-12
2026-05-13
2026-05-14
2026-05-15
2026-05-16
2026-05-27
2026-05-28
2026-05-29
2026-05-30
2026-06-02
2026-06-05
2026-06-09
2026-06-12
2026-06-17
2026-06-18
2026-06-19
2026-06-20
2026-07-05
2026-07-12
2026-07-19
2026-07-20
2026-07-21
2026-07-22
2026-07-23
2026-07-24
2026-07-25
2026-07-26
2026-08-02
2026-08-22
2026-08-29
2026-09-05
2026-09-12
2026-09-19
2026-09-29
2026-10-03
2026-10-06
2026-10-10
2026-10-13
2026-10-14
2026-10-15
2026-10-16
2026-10-17
2026-10-20
2026-10-24
2026-10-27
2026-10-31
2026-11-08
2026-11-09
2026-11-10
2026-11-11
2026-11-14
2026-11-15
2026-11-18
2026-11-21
2026-11-22
2026-11-25
2026-11-28
2026-11-29
2026-12-02
2026-12-05
2026-12-06
2026-12-09
2026-12-10
2026-12-11
2026-12-12
2026-12-26
2027-01-02
2027-01-09
2027-01-16
2027-01-23
2027-02-03
2027-02-04
2027-02-05
2027-02-06
2027-02-09
2027-02-12
2027-02-16
2027-02-19
2027-02-24
2027-02-25
2027-02-26
2027-02-27
2027-03-20
2027-03-27
2027-04-03
2027-04-10
2027-04-17
2027-04-28
2027-04-29
2027-04-30
2027-05-01
2027-05-04
2027-05-08
2027-05-11
2027-05-15
2027-05-19
2027-05-20
2027-05-21
2027-05-22
2027-06-06
2027-06-07
2027-06-08
2027-06-09
2027-06-10
2027-06-11
2027-06-12
2027-06-13
2027-06-19
2027-06-20
2027-06-26
2027-06-27
2027-07-03
2027-07-04
2027-07-05
2027-07-06
2027-07-07
2027-07-08
2027-07-09
2027-07-10
2027-07-18
2027-07-19
2027-07-20
2027-07-21
2027-07-22
2027-07-23
2027-07-24
2027-07-25
2027-07-31
2027-08-01
2027-08-07
2027-08-08
2027-08-14
2027-08-15
2027-08-16
2027-08-17
2027-08-18
2027-08-19
2027-08-20
2027-08-21
2027-08-29
2027-08-30
2027-08-31
2027-09-01
2027-09-02
2027-09-03
2027-09-04
2027-09-05
2027-09-11
2027-09-12
2027-09-18
2027-09-19
2027-09-20
2027-09-24
2027-09-25
2027-09-27
2027-09-28
2027-09-29
2027-09-30
2027-10-01
2027-10-16
2027-10-23
2027-10-30
2027-11-06
2027-11-13
2027-11-24
2027-11-25
2027-11-26
2027-11-27
2027-11-30
2027-12-03
2027-12-07
2027-12-10
2027-12-15
2027-12-16
2027-12-17
2027-12-18
2028-01-02
2028-01-03
2028-01-04
2028-01-05
2028-01-08
2028-01-09
2028-01-12
2028-01-15
2028-01-16
2028-01-19
2028-01-22
2028-01-23
2028-01-26
2028-01-29
2028-01-30
2028-02-02
2028-02-03
2028-02-04
2028-02-05
2028-02-14
2028-02-15
2028-02-16
2028-02-17
2028-02-18
2028-02-20
2028-02-21
2028-02-25
2028-02-26
2028-02-27
2028-03-04
2028-03-05
2028-03-11
2028-03-12
2028-03-18
2028-03-28
2028-04-01
2028-04-04
2028-04-08
2028-04-11
2028-04-12
2028-04-13
2028-04-14
2028-04-15
2028-04-18
2028-04-22
2028-04-25
2028-04-29
2028-05-09
2028-05-13
2028-05-16
2028-05-20
2028-05-23
2028-05-24
2028-05-25
2028-05-26
2028-05-27
2028-05-30
2028-06-03
2028-06-06
2028-06-10
2028-06-24
2028-07-01
2028-07-08
2028-07-15
2028-07-22
2028-08-02
2028-08-03
2028-08-04
2028-08-05
2028-08-08
2028-08-11
2028-08-15
2028-08-18
2028-08-23
2028-08-24
2028-08-25
2028-08-26
2028-09-12
2028-09-13
2028-09-14
2028-09-15
2028-09-16
2028-09-19
2028-09-22
2028-09-26
2028-09-29
2028-10-04
2028-10-05
2028-10-07
2028-10-22
2028-10-29
2028-11-05
2028-11-06
2028-11-07
2028-11-08
2028-11-09
2028-11-10
2028-11-11
2028-11-12
2028-11-19
2028-12-06
2028-12-07
2028-12-15
2028-12-23
2028-12-29
2029-01-03
2029-01-04"""

commit_dates = set()
for line in dates_raw.strip().split('\n'):
    d = date.fromisoformat(line.strip())
    commit_dates.add(d)

start = min(commit_dates)
while start.weekday() != 6:
    start -= timedelta(days=1)

grid = {}
cur = start
col = 0
while cur <= max(commit_dates) + timedelta(days=6):
    for row in range(7):
        d = cur + timedelta(days=row)
        grid[(col, row)] = d in commit_dates
    col += 1
    cur += timedelta(weeks=1)

total_cols = col

# print in chunks of 60 columns to be able to read the letters
chunk = 60
for start_col in range(0, total_cols, chunk):
    end_col = min(start_col + chunk, total_cols)
    print(f"\n--- cols {start_col}-{end_col-1} ---")
    for row in range(7):
        line = ''
        for c in range(start_col, end_col):
            if grid.get((c, row), False):
                line += '█'
            else:
                line += '░'
        print(line)
```

Output:
```shell
python3 ../../solve.py  
  
--- cols 0-59 ---  
█████░█░░░█░████░░██░██░░████░░░█░░░░████░█████░░░░░░░░░░░░░  
░░█░░░█░░░█░█░░░░░█████░░░░░█░░░█░░░██░░░░░░█░░░░░░░░░░░░░░░  
░░█░░░█░░░█░█░░░░░█░█░█░░░███░░░█░░░█░░░░░░░█░░░█████░░░█░░░  
░░█░░░█████░████░░█░░░█░░░█░░░░░█░░░█░░░░░░░█░░░█░█░░░░█░█░░  
░░█░░░█░░░█░█░░░░░█░░░█░░░█░░░░░█░░░█░░░░░░░█░░░█░█░░░█░░░█░  
░░█░░░█░░░█░█░░░░░█░░░█░░░░░░░░░░░░░██░░░░░░█░░░█░█░░░█░░░█░  
░░█░░░█░░░█░████░░█░░░█░░░█░░░░░█░░░░████░░░█░░░█░░░░░░░░░░░  
  
--- cols 60-119 ---  
█████░█░░░█░░░░░░░█████░░░░░░░░░░░░░█████░░░░░░░░░░░░░░░░░░░  
░░█░░░█░░░█░░░░░░░░░█░░░░░░░░░░░░░░░█░░░░░░░░░░░░░░░░░░░░░░░  
░░█░░░█░░░█░░██░░░░░█░░░░░░░░░█████░█░░░░░░░░░░░░██░░░░░░░░░  
░░█░░░█████░█░░█░░░░█░░░░░░░░░░░█░░░█████░░░░░░░█░░█░░░░░░░░  
░░█░░░█░░░█░█░░█░░░░█░░░░░░░░░░░█░░░░░░░█░░░░░░░█░░█░░░░░░░░  
░░█░░░█░░░█░████░░░░█░░░░░░░░░░░█░░░░░░░█░░░░░░░████░░░░░░░░  
░░█░░░█░░░█░█░░█░░░░█░░░█████░█████░█████░█████░█░░█░░█████░  
  
--- cols 120-179 ---  
░░░░░░█████░█████░████░░░░░░░░░░░░░░█████░░████░░░░░░░░░░░░░  
░░░░░░█░░░█░█░░░█░█░░██░░░░░░░░░░░░░█░░░░░██░░░░░░░░░░░░░░░░  
░██░░░█░░░█░█░░░█░█░░░█░░░░░░░░██░░░█░░░░░█░░░░░█████░█████░  
█░░█░░█░░░█░█░░░█░█░░░█░░░░░░░█░░█░░█████░█░░░░░░░█░░░░░█░░░  
█░░█░░█░░░█░█░░░█░█░░░█░░░░░░░█░░█░░░░░░█░█░░░░░░░█░░░░░█░░░  
█░░█░░█░░░█░█░░░█░█░░██░░░░░░░████░░░░░░█░██░░░░░░█░░░░░█░░░  
████░░█████░█████░████░░█████░█░░█░░█████░░████░█████░█████░  
  
--- cols 180-209 ---  
░░░░░░░░░░░░░░░░░░█████░░░░░░░  
░░░░░░░░░░░░░░░░░░░░█░░░░░░░░░  
░░░░░░░██░░░███░░░░░█░░░░░░░░░  
░░░░░░█░░█░░█░░█░░░░█░░░█░░░█░  
░░░░░░█░░█░░█░░█░░░░█░░░█░░░█░  
░░░░░░████░░███░░░░░█░░░░█░█░░  
█████░█░░█░░█░░█░░░░█░░░░░█░░░
```

`THEM?!CTF{THaT_iS_a_gOOD_aSCii_arT}`
