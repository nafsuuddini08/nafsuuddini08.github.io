---
layout: single
title: HTB - Pandora
excerpt: "Pandora is a linux machine with easy level of difficulty both in explotation phase and PrivESC, and this machine runs snmp service through UDP..."
date: 2022-05-22
classes: wide
header:
  teaser: /assets/images/img-pandora/portada.png
  teaser_home_page: true
  icon: /assets/images/img-pandora/
categories:
  - CTF 
  - web pentesting
tags:
  - Hack the box
  - CVE  
  - SQLi
  - RCE
  - CMS exploit
---

Pandora is a linux machine with easy level of difficulty both in explotation phase and PrivESC, and this machine runs snmp service through UDP that we will use to enumerate the target machine and some processes that it's running and also this machine runs pandora fms that is vulnerable sqli and RCE that will help us to gain access to the machine and with that we will escalate privileges with PATH hijacking.

<p align = "center">
<img src = "/assets/images/img-pandora/portada.png">
</p>

Machine matrix:

<p align = "center">
<img src = "/assets/images/img-pandora/matrix.png">
</p>

First we will create a directory with the name of the machine, and with ***mkt*** i will create the following directories to be able to move better the content of each one of those directories.

<p align = "center">
<img src = "/assets/images/img-pandora/captura1.png">
</p>

mkt is a function that i have defined in the ***~/.zshrc*** so that I can create these directories without creating them one by one.

```
mkt () {
        mkdir {nmap,content,exploits,scripts}
}
```

## Recognition

We send one icmp trace to the victim machine, and we can see that we have sent a packet and received that packet back. and through the TTL we can know that the target machine is linux. since linux machines have ttl 64 and windows machines have ttl 128. 

<p align = "center">
<img src = "/assets/images/img-pandora/captura2.png">
</p>

If you asking why when we receive the packet the ttl shows 63 instead of 64? this is because when we send icmp packet to the machine it goes through a series of intermediary nodes and this causes the ttl to decrease by one digit, and this process is known a traceroute. We can see this if we use the ***-R*** parameter.

<p align = "center">
<img src = "/assets/images/img-pandora/captura3.png">
</p>

Anyway i have a tool on my system called ***wichsystem*** that tells if the machine is linux or windows through the ttl.

<p align = "center">
<img src = "/assets/images/img-pandora/captura4.png">
</p>

Wichsystem script.

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

## Scanning

Now with nmap we are going to do the scanning process to know what's ports and services are running on the target machine, with the following parameters.

|Flags|Description |  
|-----|----------- |
|-p-  |Means that we want to scan all the ports that exists in tcp and udp which is in total 65,535 ports.|
|-sS  |Means that we want tcp syn scan.           |
|--min-rate 5000 | Means we just want to send packets no slower then 5000 packets per second to discover ports, and with that parameter our scan will be most faster. |
|--open | Means that we want only output the ports with the status open not filtred.
|-vvv | Means that we want to output more information.
|-n | Means we don't want DNS resolution, because sometimes the DNS resolution can take our scan much slower.
|-Pn | Means that we don't to ping to discover ports.
|-oG | Means that we want to save the scan in grapable format to not rescan again, you have more formats to save like nmap, xml, etc.

Basically i export the scan in grepable format because I have a function that i define in the ~/.zshrc which is the ***extractports*** function, basically it allows me to visualize the ports in a more elegant way and it copies the ports in the clipboard, so this is useful when we are scanning a target machine and it has to much ports and we don't need to write one by one to scan those ports.

<p align = "center">
<img src = "/assets/images/img-pandora/captura5.png">
</p>

The ***extractPorts*** script:

```bash
extractPorts () {
        ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
        ip_address="$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1)"
        echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
        echo -e "\t[*] IP Address: $ip_address" >> extractPorts.tmp
        echo -e "\t[*] Open ports: $ports\n" >> extractPorts.tmp
        echo $ports | tr -d '\n' | xclip -sel clip
        echo -e "[*] Ports copied to clipboard\n" >> extractPorts.tmp
        cat extractPorts.tmp
        rm extractPorts.tmp
}
```

## Scanning - Ports Recognition

Once we have discovered possible ports, we will perform another scan to recognize the services and versions that use each of these ports. To order to do that we going to use the following parameters or flags:

|Flags|Description |  
|-----|------------|
|-sCV |Means that we want to use some nmap scripts, in this case to discover the version and services that are running each of those ports. 
|-p   |To specify the ports.           |
|-oN  |Save the scan in nmap format. 

Remember that nmap have bunch of scripts that we can use, nmap scripts end in ***.nse*** extension (nmap script engine).

<p align = "center">
<img src = "/assets/images/img-ransom/locate.png">
</p>

Remember that nmap scripts have many categories that we can search for.

<p align = "center">
<img src = "/assets/images/img-ransom/categories.png">
</p>

The scan:

```
# Nmap 7.92 scan initiated Sat May 21 17:47:03 2022 as: nmap -sCV -p22,80 -oN targeted 10.10.11.136
Nmap scan report for Panda.HTB (10.10.11.136)
Host is up (0.055s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
|_  256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Play | Landing
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat May 21 17:47:14 2022 -- 1 IP address (1 host up) scanned in 11.12 seconds
```

Ok so let's access to the website and as you can see the wappalyzer reports some information about the website (services, languges, frameworks, webserver, etc), and we can see a domain called ***panda.htb***.

<p align = "center">
<img src = "/assets/images/img-pandora/captura6.png">
</p>

Let's add this domain on the file ***/etc/hosts*** to apply virtual hosting.

<p align = "center">
<img src = "/assets/images/img-pandora/captura7.png">
</p>

If we access to the webpage with the that domain we can't see any difference on the webpage.

<p align = "center">
<img src = "/assets/images/img-pandora/captura8.png">
</p>

So let's try to fuzz this webpage with gobuester specifying the following file extensions with the ***-x*** flag, and we can't find any interesting routes that we can access.

<p align = "center">
<img src = "/assets/images/img-pandora/captura9.png">
</p>

In this case i try to fuzz if there some subdomains in that particular domain, but i don't anything either.

<p align = "center">
<img src = "/assets/images/img-pandora/captura10.png">
</p>

Looking the source code of the webpage we can't see anything interesting.

<p align = "center">
<img src = "/assets/images/img-pandora/captura11.png">
</p>

If we scan the target machine with udp we can see that there is one port open running ***snmp*** service. The scan can take little bit time, so for this we will going to specify that we want to scan the top most popular ports in UDP to go much faster ***("-top-ports=20")***.

The scan:

```
# Nmap 7.92 scan initiated Sat May 21 17:56:14 2022 as: nmap -sU -top-ports=20 -oN topports 10.10.11.136
Nmap scan report for Panda.HTB (10.10.11.136)
Host is up (0.048s latency).

PORT      STATE  SERVICE
53/udp    closed domain
67/udp    closed dhcps
68/udp    closed dhcpc
69/udp    closed tftp
123/udp   closed ntp
135/udp   closed msrpc
137/udp   closed netbios-ns
138/udp   closed netbios-dgm
139/udp   closed netbios-ssn
161/udp   open   snmp
162/udp   closed snmptrap
445/udp   closed microsoft-ds
500/udp   closed isakmp
514/udp   closed syslog
520/udp   closed route
631/udp   closed ipp
1434/udp  closed ms-sql-m
1900/udp  closed upnp
4500/udp  closed nat-t-ike
49152/udp closed unknown

# Nmap done at Sat May 21 17:56:31 2022 -- 1 IP address (1 host up) scanned in 16.70 seconds
```
So basically snmp (Simple Network Management Protocol) is a protocol used to monitor devices on the network (routers, switches, IoT devices).

<p align = "center">
<img src = "/assets/images/img-pandora/captura5.png">
</p>


<p align = "center">
<img src = "/assets/images/img-pandora/captura5.png">
</p>


<p align = "center">
<img src = "/assets/images/img-pandora/captura5.png">
</p>


<p align = "center">
<img src = "/assets/images/img-pandora/captura5.png">
</p>


