---
layout: single
title: HTB - LogForge
excerpt: "LogForge is a linux machine with a medium level of difficulty both in the exploitation phase and the privileges escalation, in this machine we take advantage of the vulnerability of the apache tomcat service to have access to the manager panel and we will also be exploiting a very critical vulnerability that has just been released recently which is the log4shell."
date: 2022-01-14
classes: wide
header:
  teaser: /assets/images/img-logforge/portada.png
  teaser_home_page: true
  icon: /assets/images/img-logforge
categories:
  - CTF
  - Pentest
tags:
  - Hack the box
  - Linux
  - CVE
  - RCE
---

LogForge is a linux machine with a medium level of difficulty both in the exploitation phase and the escalation of privileges, in this machine we take advantage of the vulnerability of the apache tomcat service to have access to the manager panel and we will also be exploiting a very critical vulnerability that has just been released recently which is the log4shell.

<p align = "center">
<img src = "/assets/images/img-logforge/portada.png">
</p>

Machine rating according to the people.

<p align = "center">
<img src = "/assets/images/img-horizontall/calificacion.png">
</p>

Machine matrix:

<p align = "center">
<img src = "/assets/images/img-logforge/matrix.png">
</p>

The first thing we are going to do is to create a file with the machine name, and inside of that file with ***mkt*** we are going to create to following directories (the mkt function remember that I have it defined in the ***~/.zshr*** to create those directories.). 

<p align = "center">
<img src = "/assets/images/img-logforge/captura1.png">
</p>

## Recognition

First we send an icmp trace to see if we have a connection on the victim machine, and with the ttl i know this is a linux machine, remember that linux machines have ttl 64 and windows machines have ttl 128. 

<p align = "center">
<img src = "/assets/images/img-logforge/captura2.png">
</p>

In my machine I have defined a python script that through the ttl reports me if it is a windows or linux machine in a more elegant way, in the case if we don’t remember which ttl belongs to which OS.

<p align = "center">
<img src = "/assets/images/img-logforge/captura3.png">
</p>

Wichsystem script:

```python
#!/usr/bin/python3
#coding: utf-8
 
import re, sys, subprocess
 
# python3 wichSystem.py YOURIP 
 
if len(sys.argv) != 2:
    print("\n[!] Uso: python3 " + sys.argv[0] + " <direccion-ip>\n")
    sys.exit(1)
 
def get_ttl(ip_address):
 
    proc = subprocess.Popen(["/usr/bin/ping -c 1 %s" % ip_address, ""], stdout=subprocess.PIPE, shell=True)
    (out,err) = proc.communicate()
 
    out = out.split()
    out = out[12].decode('utf-8')
 
    ttl_value = re.findall(r"\d{1,3}", out)[0]
 
    return ttl_value
 
def get_os(ttl):
 
    ttl = int(ttl)
 
    if ttl >= 0 and ttl <= 64:
        return "Linux"
    elif ttl >= 65 and ttl <= 128:
        return "Windows"
    else:
        return "Not Found"
 
if __name__ == '__main__':
 
    ip_address = sys.argv[1]
 
    ttl = get_ttl(ip_address)
 
    os_name = get_os(ttl)
    print("\n%s (ttl -> %s): %s\n" % (ip_address, ttl, os_name))
```

And if you asking why the ttl in the image shows 63 but not 64?, this happens because the packet that we send in the hackthebox machine does not go directly to the machine but tha packet goes through a series of intermediary nodes. and if we do a trace route when we ping the machine ***(-R)*** we can see those intermediate nodes.

<p align = "center">
<img src = "/assets/images/img-logforge/captura4.png">
</p>

## Scanning - Ports recognition

i am going to perform a tcp syn scan by adding the ***min-rate*** parameter to make the scan go as fast as possible, and the evidence of the scan I will save it in grepable format in the allports file.

```
# Nmap 7.92 scan initiated Wed Jan 12 16:36:01 2022 as: nmap -p- -sS --min-rate 5000 --open -vvv -n -Pn -oG allports 10.10.11.138
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.138 ()   Status: Up
Host: 10.10.11.138 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
# Nmap done at Wed Jan 12 16:36:13 2022 -- 1 IP address (1 host up) scanned in 11.62 seconds
```
Basically i save it in the grepable format is that i have a function defined in the ~/.zshrc called extractports that indicating the name of the file shows me the ports in a more elegant way and copies the ports it to clipboard.

<p align = "center">
<img src = "/assets/images/img-logforge/captura5.png">
</p>

Extractports script:

```bash
#!/bin/bash

function extractPorts(){
        ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
        ip_address="$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1)"
        echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
        echo -e "\t[*] IP Address: $ip_address"  >> extractPorts.tmp
        echo -e "\t[*] Open ports: $ports\n"  >> extractPorts.tmp
        echo $ports | tr -d '\n' | xclip -sel clip
        echo -e "[*] Ports copied to clipboard\n"  >> extractPorts.tmp
        cat extractPorts.tmp; rm extractPorts.tmp
}
```

And with the ports discovered we are going to perform another scan to know the versions of the services that run those ports with some recognition scripts (-sCV), and i will save the evidence of the scan in nmap format (it is advisable to save the scans in a file to avoid re-scanning).

```
# Nmap 7.92 scan initiated Wed Jan 12 16:37:32 2022 as: nmap -sCV -p22,80 -oN targeted 10.10.11.138
Nmap scan report for 10.10.11.138
Host is up (0.055s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Ultimate Hacking Championship
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jan 12 16:37:43 2022 -- 1 IP address (1 host up) scanned in 11.34 seconds
```

As there is a web server on port 80 with whatweb we do a small recognition as if it were wappalyzere extension, to know the version of the web service, cms, etc.

<p align = "center">
<img src = "/assets/images/img-logforge/captura6.png">
</p>

We access on the website with the domain and the wappalyzer reports the following information about the web. Ok, so seeing that there is an apache server and also that the website uses java, it means that the website is probably using the apache tomcat service.

<p align = "center">
<img src = "/assets/images/img-logforge/captura7.png">
</p>

But first of all let's check if there is anything interesting in the code of this website, and we don't see anything useful.

<p align = "center">
<img src = "/assets/images/img-logforge/captura8.png">
</p>

We are going to do a little fuzzing with nmap but in this case we do not find anything, well nothing happens as we already know that apache tomcat is running we can check sevaral routes.

<p align = "center">
<img src = "/assets/images/img-logforge/captura9.png">
</p>

And this case if we are go to the page error 404 we can see that it's shows as the version that is being using the apache tomcat service, but something even more interesting is that if we right click and select in "inspect", in the network section and reload tha website we see that is reports different versions of apache and it is probably because there is a apache reverse proxy configured in the website.

<p align = "center">
<img src = "/assets/images/img-logforge/captura10.png">
</p>

So if there is an apache tomcat there is for sure a path that is the ***/manager/html*** which is where is the apache tomcat admin panel for deploy issues of our web app. and we see it exists but it tells us ***forbidden*** that we do not have access to this route.

<p align = "center">
<img src = "/assets/images/img-logforge/captura12.png">
</p>

And as we know the version of the apache tomcat service let's look for the possible vulnerabilities that exist.

<p align = "center">
<img src = "/assets/images/img-logforge/captura13.png">
</p>

Well and we see that there is a vulnerability that is the path traversal through the reverse proxy, that by putting the character ***/..;/*** some resources can be accessed that normally cannot be accessed through the reverse proxy. So let's try it.

<p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p>

Then what we are going to do is to put a resource that does not exist and we will add that character that indicates us in the article, and in this case what i want is to access the path ***/manager/html*** that did not allow us to access. And now instead of the 403 error (forbidden) now it's appear a popup to authenticate.

<p align = "center">
<img src = "/assets/images/img-logforge/captura15.png">
</p>


<!--<p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p><p align = "center">
<img src = "/assets/images/img-logforge/captura14.png">
</p>-->

Not finish
