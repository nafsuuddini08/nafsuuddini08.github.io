---
layout: single
title: Ready - Hack The Box
excerpt: "Ready was a pretty straighforward box to get an initial shell on: We identify that's it running a vulnerable instance of Gitlab and we use an exploit against version 11.4.7 to land a shell. Once inside, we quickly figure out we're in a container and by looking at the docker compose file we can see the container is running in privileged mode. We then mount the host filesystem within the container then we can access the flag or add our SSH keys to the host root user home directory."
date: 2021-05-15
classes: wide
header:
  teaser: /assets/images/htb-writeup-ready/ready_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - linux
  - gitlab
  - cve
  - docker
  - privileged container
---

![](/assets/images/htb-writeup-ready/ready_logo.png)

Ready was a pretty straighforward box to get an initial shell on: We identify that's it running a vulnerable instance of Gitlab and we use an exploit against version 11.4.7 to land a shell. Once inside, we quickly figure out we're in a container and by looking at the docker compose file we can see the container is running in privileged mode. We then mount the host filesystem within the container then we can access the flag or add our SSH keys to the host root user home directory.

## Portscan

```
sudo nmap -T4 -sC -sV -oA scan -p- 10.129.149.31
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-09 22:41 EDT
Nmap scan report for 10.129.149.31
Host is up (0.015s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
5080/tcp open  http    nginx
| http-robots.txt: 53 disallowed entries (15 shown)
| / /autocomplete/users /search /api /admin /profile 
| /dashboard /projects/new /groups/new /groups/*/edit /users /help 
|_/s/ /snippets/new /snippets/*/edit
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://10.129.149.31:5080/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
```

## Gitlab

