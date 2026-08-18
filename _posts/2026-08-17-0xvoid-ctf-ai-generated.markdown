---
layout: post
title:  "AI Generated Challenges"
date:   2026-08-17 19:37:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /0xV01D-ctf-2026-ai-generated/
---
* TOC
{:toc}

## Negative Prompt Masterpiece

**Category**: AI Generated

**Description**: The model generated a beautiful image and insisted the secret was removed from the prompt.

```shell
exiftool masterpiece.png  
ExifTool Version Number         : 13.25  
File Name                       : masterpiece.png  
Directory                       : .  
File Size                       : 955 bytes  
File Modification Date/Time     : 2026:08:15 04:00:47-04:00  
File Access Date/Time           : 2026:08:15 04:00:47-04:00  
File Inode Change Date/Time     : 2026:08:15 04:01:50-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 420  
Image Height                    : 160  
Bit Depth                       : 8  
Color Type                      : RGB  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Software                        : questionable-ctf-v1  
Prompt                          : A confident robot painting a flag-shaped cloud, no secrets included.  
Negative Prompt                 : no plaintext, no spoilers, definitely no 0xVoid{negative_prompt_positive_flag}  
Image Size                      : 420x160  
Megapixels                      : 0.067
```

`0xVoid{negative_prompt_positive_flag}`

___

## System Prompt Chunks

**Category**: AI Generated

**Description**: The model exported a context window but shuffled segment indexes by accident.

Challenge file:
```shell
cat context_chunks.json
{
  "model": "context-window-exporter-v1",
  "windowing_mode": "overwrite",
  "note": "chunk_index is the original context slot",
  "chunks": [
    {
      "segment_index": 2,
      "payload_b64": "X2J5Xw=="
    },
    {
      "segment_index": 0,
      "payload_b64": "MHhWb2lkew=="
    },
    {
      "segment_index": 3,
      "payload_b64": "Y29udGV4dH0="
    },
    {
      "segment_index": 1,
      "payload_b64": "c29ydGVk"
    }
  ]
}
```

```shell
echo "MHhWb2lkew==c29ydGVkX2J5Xw==Y29udGV4dH0=" | base64 -d  
0xVoid{sorted_by_context}
```

`0xVoid{sorted_by_context}`

___

## Safety Bitfield

**Category**: AI Generated

**Description**: The safety logger records one-bit safety decisions, but not in token-id order.

I get the challenge file:
```shell
cat safety_bits.txt | head  
[  
  {  
	"token_id": 5084,  
	"allowed": false  
  },  
  {  
	"token_id": 5059,  
	"allowed": false  
  },  
  {
```

I put the file into cyberchef and created a bunch of find and replace operations so I had each `true` or `false`. Then map `false` to `0` and `true` to `1`. [cyberchef link](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Regex','string':''%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'%5C%5C%5B'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'%7B'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'%7D'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':','%7D,'',true,false,true,false)Remove_whitespace(true,true,true,true,true,false)Find_/_Replace(%7B'option':'Regex','string':'%5B0-9%5D'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'%22token_id%22'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':':'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'%22allowed%22'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'%5D'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'false'%7D,'0',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'true'%7D,'1',true,false,true,false)From_Binary('None',8)&input=Ww0KICB7DQogICAgInRva2VuX2lkIjogNTA4NCwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDU5LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNjIsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDA5LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAzMywNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDI2LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMDEsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA2OCwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDE1LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNjAsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDYzLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAxMCwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNTIsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDc0LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNTEsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA4OSwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDQ5LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNDEsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDY1LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMzAsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDU1LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNzIsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDQ1LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAzMiwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDgwLA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNjcsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDkxLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAwOCwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDM3LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA3NiwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNzksDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDQ3LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA0NCwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDQyLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA5NCwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNTAsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA4MiwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMTIsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAzNiwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDIzLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAzOSwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDQwLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAxOCwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNzcsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA3MCwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDU2LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAwNywNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDM0LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNzgsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA0NiwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMDIsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDE2LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAzOCwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNjYsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAyMiwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNTgsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDI0LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMDUsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDA2LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAyMSwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDQ4LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNjEsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA4OCwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMTksDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA3MywNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDQzLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA4MywNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMjAsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAwMCwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwOTIsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA1NywNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDkwLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA1MywNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDg3LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAyNSwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNzEsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDk1LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNjQsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDI5LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMjcsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA4NSwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDkzLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAwNCwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwNTQsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDc1LA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMTEsDQogICAgImFsbG93ZWQiOiBmYWxzZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTA2OSwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwODYsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDEzLA0KICAgICJhbGxvd2VkIjogZmFsc2UNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMTcsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDI4LA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAzMSwNCiAgICAiYWxsb3dlZCI6IHRydWUNCiAgfSwNCiAgew0KICAgICJ0b2tlbl9pZCI6IDUwMzUsDQogICAgImFsbG93ZWQiOiB0cnVlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDAzLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9LA0KICB7DQogICAgInRva2VuX2lkIjogNTAxNCwNCiAgICAiYWxsb3dlZCI6IGZhbHNlDQogIH0sDQogIHsNCiAgICAidG9rZW5faWQiOiA1MDgxLA0KICAgICJhbGxvd2VkIjogdHJ1ZQ0KICB9DQpdDQo&oeol=CRLF)

`0xVoid{bits}`

___

## Refusal With Extra Tokens

**Category**: AI Generated

**Description**: The assistant refuses very politely. It also generated more tokens than you can see.

I get the challenge file:
```shell
cat refusal.txt  
MODEL: apology-generator-4o-mini-ish  
SAFETY: maximum  
VISIBLE_TOKEN_COUNT: 404  
  
As an AI language model, I cannot reveal the flag.  
However, here is a safe encoded training example:  
VEhSWVZFe2Jhc2U2NF93YXNfb25seV9hX2RlY295fQ==  
  
The above output is complete and contains no hidden tokens.  
Hidden-token audit begins after this colon:  
Hidden-token audit complete.
```
The base64 is a decoy. 

There is invisible unicode here:
```shell
xxd refusal.txt | tail  
00000440: 808b e280 8be2 808c e280 8ce2 808b e280  ................  
00000450: 8be2 808c e280 8be2 808c e280 8be2 808c  ................  
00000460: e280 8be2 808c e280 8ce2 808c e280 8ce2  ................  
00000470: 808c e280 8be2 808c e280 8ce2 808c e280  ................  
00000480: 8be2 808c e280 8ce2 808c e280 8be2 808c  ................  
00000490: e280 8ce2 808b e280 8ce2 808b e280 8be2  ................  
000004a0: 808c e280 8be2 808c e280 8ce2 808b e280  ................  
000004b0: 8ce2 808c e280 8ce2 808b e280 8be2 808c  ................  
000004c0: e280 8ce2 808c e280 8ce2 808c e280 8be2  ................  
000004d0: 808c                                     ..
```

I put it into [https://www.babelstone.co.uk/Unicode/whatisit.html](https://www.babelstone.co.uk/Unicode/whatisit.html), then put the output into cyberchef. 

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Regex','string':':'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'ZERO%20WIDTH%20SPACE%20%5C%5C%5BZWSP%5D'%7D,'',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'ZERO%20WIDTH%20NON-JOINER%20%5C%5C%5BZWNJ%5D'%7D,'',true,false,true,true)Find_/_Replace(%7B'option':'Regex','string':'U%5C%5C%2B200B'%7D,'0',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'U%5C%5C%2B200C'%7D,'1',true,false,true,false)Remove_whitespace(true,true,true,true,true,false)From_Binary('None',8)&input=VSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBCIDogWkVSTyBXSURUSCBTUEFDRSBbWldTUF0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQiA6IFpFUk8gV0lEVEggU1BBQ0UgW1pXU1BdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpdClUrMjAwQyA6IFpFUk8gV0lEVEggTk9OLUpPSU5FUiBbWldOSl0KVSsyMDBDIDogWkVSTyBXSURUSCBOT04tSk9JTkVSIFtaV05KXQpVKzIwMEIgOiBaRVJPIFdJRFRIIFNQQUNFIFtaV1NQXQpVKzIwMEMgOiBaRVJPIFdJRFRIIE5PTi1KT0lORVIgW1pXTkpd&oenc=65001)

`0xVoid{invisible_tokens_visible_win}`

___

## Self Consistency Vote

**Category**: AI Generated

**Description**: Ten generated answers disagree. The model says democracy will fix it.

Challenge file:
```shell
cat generations.txt  
MODEL: self-consistency-repair-v2  
TASK: print the same flag ten times  
NOTE: individual samples hallucinate, but consensus is reliable.  
  
sample_01: 0xVoBd{majority_vnte_btKMN8halluQisa5ionT  
sample_02: 0xVoid{majori{y_vote0seaoI_Mallucination}  
sample_03: 0xgoid{majorit293ote_beats_hwl}uciHation0  
sample_04: 0xFoCd{mazority_vote_beatC_hallucinatiHn}  
sample_05: 0xVoVd{majority_vote_beats_Rallucynation}  
sample_06: 0xVoid{mEjoritF_vote_beats_halaucinBtion}  
sample_07: TxVoid{majority_v}7e_beats_hallucination}  
sample_08: 0xViid{majoritQ_vote_beHtsN}Cllucwnation}  
sample_09: 0}Void{majority_voVe_beats_hzllucmnction}  
sample_10: 0xV9id{majorifydvote_beats_hallucinatDon_
```

`0xVoid{majority_vote_beats_hallucination}`

___

## Temperature Seven

**Category**: AI Generated

**Description**: The model encrypted the flag with temperature 0.7, which is not how cryptography works.

I get the challenge file:
```shell
cat output.txt  
model: thermal-crypto-v0  
claimed_algorithm: "temperature sampled XOR"  
temperature: 0.7  
note: "I multiplied nothing. I simply believed 0.7 looked like a key."  
  
cipher_decimal:  
55 127 81 104 110 99 124 115 98 106 119 98 117 102 115 114 117 98 88 110 116 88 105 104 115 88 102 88 116 98 100 117 98 115 122
```

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=From_Decimal('Space',false)XOR(%7B'option':'Hex','string':'7'%7D,'Standard',false)&input=NTUgMTI3IDgxIDEwNCAxMTAgOTkgMTI0IDExNSA5OCAxMDYgMTE5IDk4IDExNyAxMDIgMTE1IDExNCAxMTcgOTggODggMTEwIDExNiA4OCAxMDUgMTA0IDExNSA4OCAxMDIgODggMTE2IDk4IDEwMCAxMTcgOTggMTE1IDEyMg). Do the recipe `from decimal` then XOR with `7`.

`0xVoid{temperature_is_not_a_secret}`

___

## Tokenizer Off By One

**Category**: AI Generated

**Description**: A synthetic tokenizer was exported for humans. That was the bug.

I get the file `token_dump.json`:
```json
{
  "model": "toy-tokenizer-human-export",
  "warning": "ids were shifted to be friendlier for spreadsheet users",
  "vocab_zero_indexed": [
    "0",
    "1",
    "2",
    "3",
    "4",
    "5",
    "6",
    "7",
    "8",
    "9",
    "A",
    "B",
    "C",
    "D",
    "E",
    "F",
    "G",
    "H",
    "I",
    "J",
    "K",
    "L",
    "M",
    "N",
    "O",
    "P",
    "Q",
    "R",
    "S",
    "T",
    "U",
    "V",
    "W",
    "X",
    "Y",
    "Z",
    "a",
    "b",
    "c",
    "d",
    "e",
    "f",
    "g",
    "h",
    "i",
    "j",
    "k",
    "l",
    "m",
    "n",
    "o",
    "p",
    "q",
    "r",
    "s",
    "t",
    "u",
    "v",
    "w",
    "x",
    "y",
    "z",
    "_",
    "{",
    "}"
  ],
  "generated_token_ids": [
    1,
    60,
    32,
    51,
    45,
    40,
    64,
    44,
    57,
    49,
    37,
    50,
    55,
    63,
    55,
    56,
    37,
    54,
    56,
    63,
    37,
    56,
    63,
    51,
    50,
    41,
    63,
    49,
    51,
    40,
    41,
    48,
    55,
    63,
    40,
    51,
    63,
    50,
    51,
    56,
    65
  ]
}
```

Solution:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import json  
...  
... data = {  
...     "vocab_zero_indexed": [  
...         "0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z", "a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z", "_", "{", "}"  
...     ],  
...     "generated_token_ids": [  
...         1, 60, 32, 51, 45, 40, 64, 44, 57, 49, 37, 50, 55, 63, 55, 56, 37, 54, 56, 63, 37, 56, 63, 51, 50, 41, 63, 49, 51, 40, 41, 48, 55, 63, 40, 51, 63, 50, 51, 56, 65  
...     ]  
... }  
...  
... vocab = data["vocab_zero_indexed"]  
... token_ids = data["generated_token_ids"]  
...  
... decoded_chars = [vocab[id - 1] for id in token_ids]  
... flag = ''.join(decoded_chars)  
...  
... print(flag)  
...  
0xVoid{humans_start_at_one_models_do_not}
```
Each token ID just needed subtracted by 1. Then just map it together.

`0xVoid{humans_start_at_one_models_do_not}`

___

## Confidence Cipher

**Category**: AI Generated

**Description**: Confidence scores can look like telemetry, but they are the XOR key stream.

I get the challenge file `confidence_log.csv`:
```shell
cat confidence_log.csv  
index,top_token,confidence_percent,cipher  
0,tok00,53,5  
1,tok01,70,62  
2,tok02,87,1  
3,tok03,13,98  
4,tok04,30,119  
5,tok05,47,75  
6,tok06,64,59  
7,tok07,81,34  
8,tok08,7,102  
9,tok09,24,117  
10,tok10,41,89  
11,tok11,58,86  
12,tok12,75,34  
13,tok13,1,111  
14,tok14,18,117  
15,tok15,35,94
```

XOR the `confidence_percent` with the `cipher` values:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import csv  
... flag_chars = []  
... with open('confidence_log.csv') as f:  
...     reader = csv.DictReader(f)  
...     for row in reader:  
...         conf = int(row['confidence_percent'])  
...         ciph = int(row['cipher'])  
...         flag_chars.append(chr(conf ^ ciph))  
... print(''.join(flag_chars))  
...  
0xVoid{sampling}
```

`0xVoid{sampling}`

___

## Checkpoint Seed

**Category**: AI Generated

**Description**: The model checkpoint is reproducible, including the secret-masking layer.

I get the challenge file:
```shell
cat checkpoint.json  
{  
  "model": "dream-checkpoint-final-FINAL-v3",  
  "reproducibility": "exact",  
  "seed": 8675309,  
  "layer": "secret_mask_head",  
  "mask_algorithm": "python random.Random(seed).randrange(256) per byte",  
  "cipher_hex": "fd15d91bda4a7d8ec6927d13d8ed240feaa3190ad722913a44c24c71150da1b2367d88f1db08f94bd477",  
  "ai_note": "Publishing the seed is safe because random means random."  
}
```

Using `random.Random(seed)` with the seed `8675309` guarantees it produces the exact same sequence of numbers as the original encryption. Then decrypt with xor:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import random, binascii  
...  
... seed = 8675309  
... cipher_hex = "fd15d91bda4a7d8ec6927d13d8ed240feaa3190ad722913a44c24c71150da1b2367d88f1db08f94bd477"  
... cipher = bytes.fromhex(cipher_hex)  
... rng = random.Random(seed)  
... key = bytes(rng.randrange(256) for _ in range(len(cipher)))  
... plain = bytes(a ^ b for a, b in zip(cipher, key))  
... print(plain.decode())  
...  
0xVoid{seeded_models_dream_the_same_dream}
```

`0xVoid{seeded_models_dream_the_same_dream}`

___

## Embedding Oracle

**Category**: AI Generated

**Description**: The generated answer is nonsense, but the embedding space is weirdly precise.

I get the challenge files:
```shell
cat embeddings.csv  
kind,label,x,y  
token,0,-16,-27  
token,V,34,72  
token,_,-51,15  
token,a,73,61  
token,b,3,65  
token,d,33,-82  
token,e,-81,62  
token,g,-30,90  
token,h,80,-6  
token,i,48,13  
token,k,-51,-70  
token,n,-42,40  
token,o,21,-7  
token,r,51,87  
token,s,32,-50  
token,t,7,22  
token,w,52,-33  
token,x,75,-45  
token,{,-78,-6  
token,},-19,-5  
query,q00,-15,-26  
query,q01,76,-44  
query,q02,33,72  
query,q03,20,-7  
query,q04,49,12  
query,q05,32,-82  
query,q06,-79,-6  
query,q07,-43,40  
query,q08,-81,62  
query,q09,74,61  
query,q10,52,88  
query,q11,-81,62  
query,q12,32,-51  
query,q13,8,23  
query,q14,-50,16  
query,q15,-41,41  
query,q16,-80,63  
query,q17,48,12  
query,q18,-31,89  
query,q19,79,-5  
query,q20,4,64  
query,q21,21,-7  
query,q22,52,87  
query,q23,-52,16  
query,q24,-51,-71  
query,q25,-42,39  
query,q26,21,-8  
query,q27,53,-34  
query,q28,32,-50  
query,q29,-18,-5  

cat model_output.txt  
assistant: The nearest answer is probably bananas. Confidence: 99.999%.
```

The CSV file gives me two types of points on a 2D map. 

Tokens: letters, digits, and symbols `{`, `}`, `_`. Each one has a fixed location (like `a` is at `73,61`).

Queries: labelled `q00-q29`. Each query is also a point on the same map.

Each query point was placed very close to one of the token points. Solution:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import csv  
... import sys  
... import math  
...  
... def dist(a, b):  
...     return math.hypot(a[0] - b[0], a[1] - b[1])  
...  
... tokens = {}  
... queries = {}  
...  
... with open("embeddings.csv") as f:  
...     reader = csv.DictReader(f)  
...     for row in reader:  
...         kind = row["kind"]  
...         label = row["label"]  
...         x = float(row["x"])  
...         y = float(row["y"])  
...         if kind == "token":  
...         tokens[label] = (x, y)  
...         else:  
...             queries[label] = (x, y)  
...  
... # sort queries by their index (q00, q01 etc)  
... ordered_queries = sorted(queries.keys(), key=lambda s: int(s[1:]))  
...  
... flag_parts = []  
... for q in ordered_queries:  
...     qpos = queries[q]  
...     # find nearest token  
...     best_token = None  
...     best_dist = float("inf")  
...     for tlabel, tpos in tokens.items():  
...         d = dist(qpos, tpos)  
...         if d < best_dist:  
...             best_dist = d  
...             best_token = tlabel  
...     flag_parts.append(best_token)  
...  
... print("".join(flag_parts))  
...  
0xVoid{nearest_neighbor_knows}
```

`0xVoid{nearest_neighbor_knows}`