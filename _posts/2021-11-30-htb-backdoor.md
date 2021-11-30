---
layout: single
title: HTB - Backdoor
excerpt: "Backdoor is a machine that has linux OS with easy level of difficulty both in terms of intrusion and privilege escalation. on the port 80 runs wordpress which is vulnerable to local file inclusion and also the machine is vulnerable to remote command execution."
date: 2021-11-30
classes: wide
header:
  teaser: /assets/images/img-backdoor/portada.png
  teaser_home_page: true
  icon: /assets/images/img-backdoor/
categories:
  - CTF
  - Pentest
  - web pentesting
tags:
  - Hack the box
  - wordpress 
  - reverse shell
  - lfi 
---

Backdoor is a machine that has linux OS with easy level of difficulty both in terms of intrusion and privilege escalation. on the port 80 runs wordpress which is vulnerable to local file inclusion and also the machine is vulnerable to remote command execution.

<p align = "center">
<img src = "/assets/images/img-backdoor/portada.png">
</p>

Machine matrix:

<p align = "center">
<img src = "/assets/images/img-backdoor/nivel.png">
</p>

First I will create a directory with the name of the machine, and with ***mkt*** I will create the following directories to be able to move better the content of each one of those directories.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura1.png">
</p>

mkt is a function that i have defined in the ***~/.zshrc*** so that I can create these directories without creating them one by one.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura2.png">
</p>

## Recognition

We send one icmp trace to the victim machine, and we can see that we have sent a packet and received that packet back. and through the TTL I know that I am on a linux machine. since linux machines have ttl 64 and windows machines have ttl 128. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura3.png">
</p>

If you asking why when I receive the packet the ttl shows 63 instead of 64? this is because when we send icmp traces to the machine it goes through a series of intermediary nodes and this causes the ttl to decrease by one digit or unit whatever you want to call it. we can see this if we do a traceroute with the ***-R*** parameter.

<p align = "center">
<img src = "/assets/images/img-backdoor/trace.png">
</p>

Anyway i have a tool on my system called ***wichsystem*** that tells you if the machine is linux or windows through the ttl.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura4.png">
</p>

And this is the python script of the function wichsystem.

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

## Ports recognition

Now with nmap we are going to do the port recognition and services that run those ports.

First we are going to scan how many open ports this machine has and export it in grepable format to the allports file.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura6.png">
</p>
 
Basically I export it in grepable format because I have a function define in the ~/.zshrc which is the ***extractports*** function, basically it allows me to visualize the ports in a more elegant way and it copies the ports in the clipboard.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura7.png">
</p>

Now let's do another scan to see the versions of the services running on each of these ports.

```
# Nmap 7.91 scan initiated Tue Nov 30 03:14:40 2021 as: nmap -sCV -p22,80,1337 -oN targeted 10.10.11.125
Nmap scan report for backdoor.htb (10.10.11.125)
Host is up (0.058s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 b4:de:43:38:46:57:db:4c:21:3b:69:f3:db:3c:62:88 (RSA)
|   256 aa:c9:fc:21:0f:3e:f4:ec:6b:35:70:26:22:53:ef:66 (ECDSA)
|_  256 d2:8b:e4:ec:07:61:aa:ca:f8:ec:1c:f8:8c:c1:f6:e1 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: WordPress 5.8.1
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Backdoor &#8211; Real-Life
1337/tcp open  waste?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Nov 30 03:17:36 2021 -- 1 IP address (1 host up) scanned in 175.66 seconds
```

With the command ***whatweb*** we can make a small recognition of the web, to be able to see the services used by the web.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura8.png">
</p>

Now we access the website to see some information that may be useful to us.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura9.png">
</p>

When I click on the home menu it does not let me access because it does not find the following domain. And this means that virtual hosting is being applied.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura10.png">
</p>

To apply the virtual hosting we have to go to the file ***/etc/hosts*** and we put the ip and the domain.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura11.png">
</p>

Now we can see that the domain is now recognized.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura12.png">
</p>

Now when accessing the home page with the domain there is no difference in the web by putting the domain.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura13.png">
</p>


