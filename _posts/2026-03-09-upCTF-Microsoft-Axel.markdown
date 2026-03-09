---
layout: post
title:  "Microsoft Axel"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-Microsoft-Axel/
---

**Title**: Microsoft Axel

**Category**: web

**Author**: 0xacb & castilho

**Description**: Upload anything to download later, which our latest download everything service.

I get the challenge files:
```shell
unzip microsoft-axel.zip  
Archive:  microsoft-axel.zip  
  inflating: app.py  
  inflating: build_docker.sh  
  inflating: Dockerfile  
  inflating: entrypoint.sh  
  creating: files/  
  inflating: files/welcome.txt  
  inflating: readFlag.c  
  inflating: requirements.txt  
  creating: static/  
  inflating: static/style.css  
  creating: templates/  
  inflating: templates/index.html
```

Contents of `app.py`:
```python
from __future__ import annotations

import os
import subprocess
from pathlib import Path

from flask import Flask, render_template, request, send_file

app = Flask(__name__)

BASE_DIR = Path(__file__).resolve().parent
FILES_DIR = Path(os.environ.get("FILES_DIR", BASE_DIR / "files")).resolve()
AXEL_BIN = os.environ.get("AXEL_BIN", "/usr/local/bin/axel")
AXEL_TIMEOUT = int(os.environ.get("AXEL_TIMEOUT", "120"))


def run_axel_download(url: str) -> tuple[bool, str]:
    FILES_DIR.mkdir(parents=True, exist_ok=True)
    cmd = [AXEL_BIN, url]
    try:
        proc = subprocess.run(
            cmd,
            cwd=FILES_DIR,
            capture_output=True,
            text=True,
            timeout=AXEL_TIMEOUT,
            check=False,
        )
    except subprocess.TimeoutExpired:
        return False, f"axel timed out after {AXEL_TIMEOUT}s"
    except OSError as exc:
        return False, f"failed to execute axel: {exc}"

    output = (proc.stdout or "") + (proc.stderr or "")
    if proc.returncode != 0:
        detail = output.strip().splitlines()[-1] if output.strip() else "unknown error"
        return False, f"axel failed (exit {proc.returncode}): {detail}"

    return True, "Download completed"


def human_size(num_bytes: int) -> str:
    units = ["B", "KB", "MB", "GB", "TB"]
    size = float(num_bytes)
    for unit in units:
        if size < 1024.0 or unit == units[-1]:
            return f"{size:.1f} {unit}" if unit != "B" else f"{int(size)} B"
        size /= 1024.0
    return f"{num_bytes} B"


def list_downloads() -> list[dict[str, str]]:
    FILES_DIR.mkdir(parents=True, exist_ok=True)
    items = []

    for entry in FILES_DIR.iterdir():
        if not entry.is_file() or entry.name.startswith("."):
            continue

        stat = entry.stat()
        items.append(
            {
                "name": entry.name,
                "size": human_size(stat.st_size),
                "modified": entry.stat().st_mtime,
            }
        )

    return sorted(items, key=lambda x: x["modified"], reverse=True)


@app.get("/")
def index():
    files = list_downloads()
    return render_template(
        "index.html",
        files=files,
        file_count=len(files),
        result=None,
        url_value="",
        axel_bin=AXEL_BIN,
    )


@app.post("/fetch")
def fetch():
    url = request.form.get("url", "").strip()
    ok, message = run_axel_download(url)
    files = list_downloads()
    return render_template(
        "index.html",
        files=files,
        file_count=len(files),
        result={"ok": ok, "message": message},
        url_value=url,
        axel_bin=AXEL_BIN,
    )


@app.get("/download/<path:filename>")
def download(filename: str):
    target = FILES_DIR / filename
    return send_file(target, as_attachment=True)


if __name__ == "__main__":
   app.run(host="0.0.0.0", port=5006)
```

Contents of `readFlag.c`:
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    FILE *fptr;
    char flag[256];

    fptr = fopen("/flag.txt", "r");

    if (fptr == NULL) {
        perror("Error opening file /flag.txt");
        return 1;
    }

    if (fgets(flag, sizeof(flag), fptr) != NULL) {
        printf(flag);
    } else {
        printf("File is empty.\n");
    }

    fclose(fptr);
    return 0;
}
```

The `download` route in `app.py` has zero sanitization:
```python
@app.get("/download/<path:filename>")
def download(filename: str):
    target = FILES_DIR / filename
    return send_file(target, as_attachment=True)
```

Through some testing I find path traversal in the `/download` endpoint:
```shell
http://46.225.117.62:30012/download/..%2F..%2F..%2Fflag.txt
```

This downloads `flag.txt` for me. The filename parameter is unsanitized and flask allows slashes in `<path:filename>`, so you can escape the `files` directory and access arbitrary files on the system.

`upCTF{4x3l_0d4y_w1th4_tw1st-w6jvxKs322a0241d}`