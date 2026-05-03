---
layout: post
title:  "teamforge (partner)"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-teamforge/
---

**Author**: teamforge

**Category**: Web

**Description**: This is a task from our partner. It takes place on a separate platform. Your task is to register and launch a container. The flag that you find is inserted by us :)

**HackAdvisor Labs** is an educational cybersecurity CTF platform where each challenge is an isolated web application with a real vulnerability that needs to be found and exploited to obtain a flag.

[HackAdvisor Labs — Challenge](https://labs.hackadvisor.io/ru/challenges/teamforge-kubstu-event)

Description on `https://labs.hackadvisor.io`:
```shell
TeamForge is a modern team collaboration platform with role-based access control (Owner, Admin, Member). The platform allows create teams, inviting managers, and projects.

You have a regular account. The platform has an action that will allow the team owners to invite new members with different roles. Your goal is to find a logic flaw that allows you to your escalate your privileges.

Objective: Escalate your privileges from a regular Member to Admin or Owner. The flag is displayed on the admin dashboard, only to users with induced privileges.

Test credentials: user@test.com / password123
```

I go to `https://labs.hackadvisor.io/ru/challenges/teamforge-kubstu-event` and login and start the website. On the site I login with `user@test.com:password123`. 

I explore the site and log the requests in burpsuite. I see requests to `/org/2`, `/org/2/teams`, and `/org/2/projects`. 

I check `/org/1` through `/org/5` (there were 5 shown on the page). All of them other than 2 say: `You are not a member of this organization`. The same happens for `/org/1/projects` through `/org/5/projects`.

On `https://1262e5bf-ba76-491a-b805-27cef8298e88.labs.hackadvisor.io/org/1/team` I find a new email of someone who was invited: `victoria.chase@acme.com`

I logout and create a user with this email. When I login I see the invitation:

![Alt text](/images/kubstuweb5.png)

I accept the invitation and find the flag in the settings:

![Alt text](/images/kubstuweb6.png)

`KubSTU{21509994fd5a1383bfb6b4c4d85b4cf0}`