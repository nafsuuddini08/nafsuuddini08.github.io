---
layout: single
title: HTB - Ransom
excerpt: "Ransom is Linux machine with a medium level defficulty both in exploitain, user own, privilage escalation phase, this involves vulnerabilities such as type juggling that helps us gain access to the web page, and we will also have an encrypted zip file that we must..."
date: 2022-03-18
classes: wide
header:
  teaser: /assets/images/img-ransom/portada.png
  teaser_home_page: true
  icon: /assets/images/
categories:
  - CTF
  - Pentest
tags:
  - Hack the box
  - Linux
  - Web
  - PHP
  - Api
---

Ransom is Linux machine with a medium level defficulty both in exploitain, user own, privilage escalation phase, this involves vulnerabilities such as type juggling that helps u  s gain access to the web page, and we will also have an encrypted zip file that we must access through a plaintext attack and for privilege escalation we must review some code files of the web page that will help us access as the root user.

<p align = "center">
<img src = "/assets/images/img-ransom/portada.png">
</p>

Machine matrix

<p align = "center">
<img src = "/assets/images/img-ransom/matrix.png">
</p>

Fisrt thing that we are going to do is created a directory with the name of the target machine and inside of that directory with ***mkt*** command i am going to create the following directories, to organize the content (mkt is a function that i have defined in my zshrc to create the following directories). 

<p align = "center">
<img src = "/assets/images/img-ransom/captura1.png">
</p>

Once we have connected to the htb vpn and turned on the target machine, we will check if we have connectivity with the machine by sending one ICMP trace. And we see that we have sent a package and we received it back and with this we already know that we have connectivity, trough the TTL we can know if the machine is windows or linux, remember that the linux machines usually has ttl 64 and the windows machine has ttl 128.

<p align = "center">
<img src = "/assets/images/img-ransom/captura2.png">
</p>

And if you asking, why the ttl reports 63 instead of 64? This is because the packet that we send has to go through certain intermediate nodes before reaching the destination and this term is known as traceroute. If we use the ***-R*** flag on the ping command we can see those "nodes".

<p align = "center">
<img src = "/assets/images/img-ransom/captura3.png">
</p>

Anyway, i have s script defined on my machine called ***wichSystem***, and simply specifying the IP address of the machine through the ttl will tell us if it is a linux or windows machine.

<p align = "center">
<img src = "/assets/images/img-ransom/captura4.png">
</p>

wichSystem script:

```python
#!/usr/bin/python3
#coding: utf-8

import re, sys, subprocess

# python3 wichSystem.py 10.10.10.188

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

We are going to perform nmap scanning to discover ports and other relevant information to the target machine, for this we are going to use the following parameters or flags:

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

And basically i save the scan in grapable format because i have defined in zshrc a function called ***extractports***, that specifying the file name shows me the ports and the IP address of the target machine in a much more elegant way and copies the ports to the clipboard. And this can be useful if there is a machine that has many ports enbled and we do not have to write those ports one by one.

<p align = "center">
<img src = "/assets/images/img-ransom/captura5.png">
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

