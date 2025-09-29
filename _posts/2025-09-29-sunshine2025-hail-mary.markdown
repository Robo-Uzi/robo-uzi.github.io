---
layout: post
title:  "Hail Mary"
date:   2025-09-29 13:14:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-hail-mary/
---

**Title:** Hail Mary

**Category:** scripting

**Author:** tsuto

**Description:** Dr. Ryland Grace is in a tough spot! He has been tasked with performing an experiment on some "taumoeba" (Amoebas collected in the Tau Ceti system). He needs to try to breed them to meet certain biological parameters to ensure their survival when they return to Earth. See if you can help guide the experiments to find the optimal genetic code needed and you will be rewarded!

`nc chal.sunshinectf.games 25201`

When I connect I see this:
```shell
nc chal.sunshinectf.games 25201  
  
Welcome to the NASA Evolutionary Biology Lab! We are running tests on genetically modified taumoeba to try to optimize their survivability rating in alien atmospheres. You have 100 generations to attempt to reach an average of 95.0% that survive. Submit populations of 100 gene samples, each 10 floats (0-1) in JSON format.  
            
Example: {"samples":[[0,0,0,0,0,0,0,0,0,0],[0,0,0,0,0,0,0,0,0,0]....]}  
            
Good luck! The survival of Earth depends on your science!  
  
^C
```
So the server expects 100 gene samples (each 10 floats between 0–1) and simulates their survival. The goal is to reach an average survival of > 95%.

I run this script to get the flag:
```python
#!/usr/bin/env python3
import socket, json, random, time, re

HOST, PORT = "chal.sunshinectf.games", 25201
POP_SIZE, GENOME_LEN = 100, 10
MUT_STD = 0.08
ELITE_COUNT = 5

# random population
pop = [[random.random() for _ in range(GENOME_LEN)] for _ in range(POP_SIZE)]

def mutate(genome):
    return [min(1.0, max(0.0, x + random.gauss(0, MUT_STD))) for x in genome]

with socket.create_connection((HOST, PORT), timeout=5) as s:
    s.settimeout(5)
    print(s.recv(4096).decode())

    for gen in range(1, 101):
        # send population
        s.sendall(json.dumps({"samples": pop}).encode() + b"\n")

        # read full response
        resp = b""
        while True:
            try:
                chunk = s.recv(4096)
                if not chunk: break
                resp += chunk
                if b"sun{" in chunk: break
            except socket.timeout:
                break

        text = resp.decode(errors="ignore")
        print(f"[Gen {gen}] Response received")

        # extract average survival
        m = re.search(r'"average"\s*:\s*([0-9.]+)', text)
        avg = float(m.group(1)) if m else None
        print("Average survival:", avg)

        # extract flag
        f = re.search(r"sun\{.*?\}", text)
        if f:
            print("\nFlag found:", f.group(0))
            break

        # parse JSON for per sample scores
        try:
            j = json.loads(text)
            scores = j.get("scores", [sum(g) for g in pop])
            if len(scores) != len(pop):
                scores = [sum(g) for g in pop]
        except:
            scores = [sum(g) for g in pop]

        # select top ELITE_COUNT using server scores
        elite_idx = sorted(range(len(scores)), key=lambda i: scores[i], reverse=True)[:ELITE_COUNT]
        top = [pop[i] for i in elite_idx]

        # create new population
        new_pop = top[:]
        while len(new_pop) < POP_SIZE:
            parent = random.choice(top)
            new_pop.append(mutate(parent))
        pop = new_pop

        time.sleep(0.1)
```
First the script creates 100 random gene samples (each a float between 0–1). The mutate function slightly tweaks each gene using a Gaussian mutation [https://www.geeksforgeeks.org/python/random-gauss-function-in-python/](https://www.geeksforgeeks.org/python/random-gauss-function-in-python/).

It sends the current population to the server, reads back the average survival and per sample scores, and stops reading early if I get the flag. 

It keeps the top 5 genomes from each generation which gradually increases the surival percentage. Each new population consists of: the best genomes, and mutated copies of the best genomes.

Generation 48 survives! Output:
```shell
python3 solve5.py  
  
Welcome to the NASA Evolutionary Biology Lab! We are running tests on genetically modified taumoeba to try to optimize their survivability rating in alien atmospheres. You have 100 generations to attempt to reach an average of 95.0% that survive. Submit populations of 100 gene samples, each 10 floats (0-1) in JSON format.  
            
Example: {"samples":[[0,0,0,0,0,0,0,0,0,0],[0,0,0,0,0,0,0,0,0,0]....]}  
            
Good luck! The survival of Earth depends on your science!  
  
  
[Gen 1] Response received  
Average survival: 0.5774086719166959  
[Gen 2] Response received  
Average survival: 0.7960111395752114  
[Gen 3] Response received  
Average survival: 0.8797699522230292

...

[Gen 46] Response received  
Average survival: 0.9485546409725782  
[Gen 47] Response received  
Average survival: 0.9494842046611254  
[Gen 48] Response received  
Average survival: None  
  
Flag found: sun{wh4t_4_gr34t_pr0j3ct}
```

`sun{wh4t_4_gr34t_pr0j3ct}`