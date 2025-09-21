---
layout: post
title:  "daily re"
date:   2025-09-21 10:22:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /K17-2025-daily-re/
---

**Title:** daily re

**Category:** beginner rev web

**Description:** It's too easy to cheat at Wordle, so I fixed that! What are the words for the 72nd, 73rd, and 74th day?

Enter your answer in the format `K17{<72nd day word>, <73rd day word>, <74th day word>}`. For example if you think the words are 'trace', 'crane', and 'rocks' respectively, enter K17{trace, crane, rocks}.

`daily-re.k17.secso.cc`

I visit the site `https://daily-re.k17.secso.cc` and find javascript in source code which sets a `DAY_NUMBER`, `KEY_NUMBER`, ensures the word is 5 letters, sets up the game, etc. 

Near the bottom I see this part which interacts with `words.wasm`:
```js
var Module = {
    onRuntimeInitialized: function() {
        let extracted_word = Module.ccall('get_word', 'string', ['number', 'string'], [DAY_NUMBER - 1, DAY_KEY]) if (!extracted_word) {
            console.error("Failed to get word!");
        }
        extracted_word = extracted_word.toUpperCase();
        if (document.readyState === "loading") {
            document.addEventListener('DOMContentLoaded', () => {
                runGame(extracted_word)
            });
        } else {
            runGame(extracted_word);
        }
    }
};
```

I download `words.wasm` and run strings on it. It looks like pairs of words and keys:
```shell
8482423c95bf2f4a165f0c08ad27a800  
might  
43dc252ca71e1d9b502bc2c44bd3fb3e  
joust  
eb38ca52bed68ffc6e771cb904a76281  
quaky  
dfd13e0a7f17da5496a22cf4423fbd7a  
unite  
de271fa1d0777a3aea3a926280e42158  
agile  
...
```

I havent worked with web assembly so I spent some time using a script override in the firefox debugger to change the day in the index file. I set a breakpoint on line 181 of the index file `runGame(extracted_word);` and was able to see the correct word for day 17. I could force the day to change but I was seeing an empty string in the debugger as a result of having the incorrect key. This was a dead end aside from helping me understand how it works. 

I end up running this in the console as quickly as I can after refreshing the page:
```javascript
(async function findDailyWords() {
  const candidateKeys = [
"8482423c95bf2f4a165f0c08ad27a800",
"43dc252ca71e1d9b502bc2c44bd3fb3e",
"eb38ca52bed68ffc6e771cb904a76281",
"dfd13e0a7f17da5496a22cf4423fbd7a",
"de271fa1d0777a3aea3a926280e42158",
"980920de164c7f279b2dc8e4ccd6fbf9",
"5dfd61c8ecae413bc851075388721e37",
"2bd5e35921f7682406c59e06e0bd5a73",
"a7d955cc7bbe8de8bb70f772da29e262",
"508eb8634faba2eaaacda7194ff6ad10",
"072f9d124ccaa8426d453b179123087c",
"651a7234ff4100ed989795879163a5f6",
"9b37ddb0f7d92de5c73ecf4b1e0ab88b",
"17aa8695a3d0cdac566de5e3f2e9d594",
"afbbe4b3dd89ce9ff319649bfe90ab79",
"337cab92d07605b1d1fc37d311b525a5",
"70e7726a9b8d5e65a6dbb19cf784ad5a",
"b3f61e6d07992d25a42114c7d50f5f69",
"67cb213109a49a3b9d3ca167c051dad9",
"37e55328fb18231f159ebb1d12b2c6a2",
"68bf7179ac3257087966787f004543d5",
"fc6521b9a1194a19de15ccd7e0975390",
"743fd28187df87b8e26e6c37ae50be1a",
"c80f40e3c5250b243bd1ceceab2edf03",
"8522b961b71c5b6bbde930cd62bf36cb",
"13dcc16b15548672e0d308c1c2c67116",
"d0229677b1c1734fd39aa96e895979c1",
"8cc8f9f16e05dfeddad7aa7b847dbd4f",
"102c63a00d6309b1a62f3552898bf5dc",
"654ce3959aadb4aa0cea8ab1f09719d7",
"9b50e0517ec8a9fe31a6712cb239fc69",
"a8b87cb7ca6c4cce8b6de91e55ff9c3e",
"6a1cc835e586a668e5de940eeb1fe364",
"83bee0d3dce885bf1c51214c32f2f90a",
"45b7698a8e92ce91b506d64fecba314a",
"f288bc4ee4dabb9ece63c60f9b9cb081",
"e612ca2c10c3a2790551a3153b725c71",
"10ebb501bc596fc0d68b77adc07885e5",
"a0ae1102ed936a10f3c0a08a4eeb1d06",
"d790b21bc90d2923cc3da0b07835f0a5",
"0b52b4ae038426e142173b550f6ff2d2",
"80e68391801c921249e41756b42b3694",
"01ccf3f1a516114b47e78b17c41d3068",
"03bc4a9cf8c632bbeef7bbeae2b80cb2",
"a1f6e432046a4d576491fb5977c04466",
"a774822c8ece8c62548df1e0fd338beb",
"559886a46d4fc0334365207ffc10d366",
"dff54dc554b539b8a434bb5313e0a3ac",
"3e0965bc9c65bc987e30db9e0e803505",
"8b8f3419f95f982974bc3ea05897f089",
"193436981ab2cd0c1c5fed476d1d2af8",
"ef6b75ace2f1217dcc33a42911db7a6b",
"e605d6861c75e572778ffb951cf231f7",
"b7732452844d88e885de9bcccb5518b6",
"6b8f71356e819af1a2065cb2b2927490",
"e177fbd897d6043ac26550b45690f304",
"042df3ad75c3a98ffaba61723a87a7d1",
"0a96f95856f4aa17f91da3ec76be0b8b",
"1f7d1fcdbf614b306caacc615ed44c33",
"e68def0e199d4312a21bb63d2fe0b01d",
"a15dd9f89f1ce0521d8d40924864f97f",
"1e83cd89d9be464fda5eb4a265fa665c",
"cec5461ee8647e58534a7ddb9d7dd620",
"c32213f4f34d1529824936ceb3863beb",
"5df4440ca30ced69e48da69ac48b2eb2",
"3d273ba1e7a4d2060921a0b828879720",
"4d24345d562516215d8e5dad1638c27b",
"48c649bc269ac8bb054bd6624ec47525",
"354810314ddeaa675974e2fdaeb558d3",
"3ed04769ba8857297b3d5019bdfde899",
"b0f88f4c8ef877ba8150f5d92d756c2c",
"dc99b25df638f6bc0eb9cdd44811f2dd",
"1ed5047bb0ada6ded5ab08fdb84a8875",
"8748afe7a3214eaafc7380bf19a680f8",
"793ef7fbf6dff901a6606407cd5f0c8b",
"fc520bf146aeb77fdea602e500e9845d",
"ac1d6c3443cd2f849a9d18b851937839",
"6db5e56cec4d173744d5b458c0608c98",
"32a9af7f43ac5a8037a047b5e07a4d5d",
"0c24f57036a5fe8518ef366f6265f9a4",
"5aedfb770b7f3f2477051b22e281f548",
"e9273a3bb44af43be051a5539f4fb66b",
"2fb6b2b917bbaa44cc74b28f2e2916f8",
"f13d833cbfa67a79802051f44f48cd02",
"0b17a8680c388a7caa02a4e296a34833",
"8a810bfed10e6d76b5c18d633f0dfde1",
"6a3d227302a8f3ec34011b7f72845f1f",
"b69f00361af11e7ae4ce9cf9a12dd6d0",
"cfd92e74f1db5f11afab3835199d0d76",
"b72f533a3a90b1b7cb7cc84420252318",
"95ea1906cdc6fa71404e0dd1d5eb5975",
"3c05527bcbb9f936cc2acd0666ef3e0a",
"6affd7f61fab8ddb3f18b4f499a665f1",
"03a85ea2dac0e104d8d1012514c8c667",
"ac5cf869be0f98af893f3e57864f2449",
"e022eaa87c84c719d445d465bc1d8d6e",
"e25903f490405fee417c76721a89bc39",
];

  function waitForModule() {
    return new Promise(resolve => {
      if (window.Module && Module.calledRun) return resolve();
      let done = false;
      if (window.Module) {
        const prev = Module.onRuntimeInitialized;
        Module.onRuntimeInitialized = function() {
          if (typeof prev === 'function') prev();
          done = true;
          resolve();
        };
      }
      const poll = setInterval(()=> {
        if (window.Module && Module.calledRun) {
          clearInterval(poll);
          if (!done) { done = true; resolve(); }
        }
      }, 100);
    });
  }

  await waitForModule();
  console.log("Starting...");

  const days = [71, 72, 73];

  for (const dayIdx of days) {
    console.log("Searching keys for day", dayIdx, "(day", dayIdx+1, ")");
    let found = false;
    for (let i=0; i < candidateKeys.length; i++) {
      const key = candidateKeys[i];
      try {
        const res = Module.ccall('get_word', 'string', ['number','string'], [dayIdx, key]);
        if (res && String(res).trim()) {
          console.log("Found for day", dayIdx+1, "key:", key, " -> word:", String(res).toUpperCase());
          found = true;
          break;
        }
      } catch (e) {
        // handle errors
        console.error("ccall threw for key", key, e);
      }
      if (i % 20 === 0) await new Promise(r => setTimeout(r, 20));
    }
    if (!found) console.warn("No key produced word", dayIdx+1);
  }
  console.log("done");
})();
```

`K17{LIMBO,URBAN,FIBER}`