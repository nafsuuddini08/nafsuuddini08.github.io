---
layout: single
title: THM - SteelMountain
excerpt: "Steelmountain is windows machine inspired by the mr.robot serie, first we have a little osint challenge we need to indentify the person on the image on the website..." 
date: 2022-08-02
classes: wide
header:
  teaser: /assets/images/img-steelMountain/portada.jpeg
  teaser_home_page: true
  icon: /assets/images/img-steelMountain/
categories:
  - CTF 
tags:
  - Tryhackme
  - Windows
  - Powershell
  - CVE
---

Steelmountain is windows machine inspired by the mr.robot serie, first we have a little osint challenge we need to indentify the person on the image on the website, then we exploit the CVE-2014-6287  to gain access to the target machine and finally we utilise powershell for privESC enumeration to gain access as a admin.

<p align = "center">
<img src = "/assets/images/img-steelMountain/portada.jpeg">
</p>

First we going to create a directory with the name of the target machine and inside of that directory with ***mkt*** we going to create the following directories to organize the content.

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

mkt is a function that i have defined on my ***~/.zshrc***, the function is the following:

```
mkt(){
	mkdir {nmap,content,exploits,scripts}
  }
```

And if we send one icmp trace on the target machine we receive a connection, and remember that the linux machine have 64 TTL and windows have 128 TTL and sometimes this values can decrease one digit or more and this because of traceroute. we can check this by using the flag ***-R*** on the ping command, in the case of windows this flag doesn't apply.

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura2.png">
</p>

## Scanning

This is the nmap scan result with the following ports that i have discovered with a previous scan. We can see the following ports with versions of the services and the nmap scan reports us that the smb is not signed, so this can be useful to enumerate hosts or the target machine with the smb protocol.

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura3.png">
</p>

We can use crackmapexec using the smb protocol to know what version of windows is using, this can be useful when we need to search for certain exploit or vulnerablity with a specific version of windows.

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura4.png">
</p>

So one of the challenges in this machine is to identify the person of the picture on the website.

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura5.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura7.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura8.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura9.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

<p align = "center">
<img src = "/assets/images/img-steelMountain/captura1.png">
</p>

