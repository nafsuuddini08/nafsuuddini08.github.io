---
layout: single
title: Reverse shell using netcat
excerpt: "We are going to learn how we can reverse shell in every OS, and some netcat commands that can help us when we are pentesting or scanning our environment."
date: 2021-11-14
classes: wide
header:
  teaser: /assets/images/img-netcat/netcat.jpg
  teaser_home_page: true
  icon: /assets/images/img-netcat/logo.png
categories:
  - Networking
  - Pentest
tags:
  - netcat
  - Linux
  - reverse shell
---

<p align = "center">
<img src = "/assets/images/img-netcat/netcat.jpg">
</p>

We are going to learn how we can reverse shell in every OS, and some netcat commands that can help us when we are pentesting or scanning our environment.

We are going to learn:

+ What is netcat
+ Basics netcat commands
+ Web server in netcat
+ What is reverse shell
+ Create a reverse shell in netcat
+ Reverse shell scripts

## What is netcat?

Netcat ***(The Network Swiss Army knife)*** it is a command line tool that reads and writes data over network connections using TCP, allows us to open TCP and UDP ports for listening and allows us to scan ports similar to nmap but with some limitations. The main use of this tool is in reverse shell.

## Basics netcat commands

On linux systems netcat is installed by default and if it is not installed we use the following command:

```
sudo apt install netcat
```
In some Windows versions it is not installed by default, so we have to install the file called nc.exe.

With the ***-help*** parameter we can see the possibilities that netcat offers us and if we want to know more information about each parameter is to use the command ***man*** to see the netcat documentation. 

<p align = "center">
<img src = "/assets/images/img-netcat/captura1.png">
</p>

Scan ports. we can specify port ranges to report if any of those port ranges are open. it doesn't do a full scan like nmap, but it reports back to the console if the port we specified is open or not.

<p align = "center">
<img src = "/assets/images/img-netcat/captura2.png">
</p>

With the ***-u*** parameter we specify that we want to scan udp ports.

<p align = "center">
<img src = "/assets/images/img-netcat/captura3.png">
</p>
