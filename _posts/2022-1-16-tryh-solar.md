---
layout: single
title: Tryhackme - Solar
excerpt: "Solar is a linux machine with medium difficulty level in the exploitation phase and easy in privilege escalation, this machine runs the apache solr 8.11.0 service which is vulnerable to log4shell and also explains what is log4j, how it works, how to exploit log4shell step by step and ways to mitigate this vulnerability."
date: 2022-01-16
classes: wide
header:
  teaser: /assets/images/img-solar/portada.png
  teaser_home_page: true
  icon: /assets/images/
categories:
  - CTF
  - Pentest
tags:
  - Tryhackme
  - Linux
  - CVE
  - RCE
---

Solar is a linux machine with medium difficulty level in the exploitation phase and easy in privilege escalation, this machine runs the apache solr 8.11.0 service which is vulnerable to log4shell and also explains what is log4j, how it works, how to exploit log4shell step by step and ways to mitigate this vulnerability.

<p align = "center">
<img src = "/assets/images/img-solar/portada.png">
</p>

The first thing we are going to do is to create a file with the machine name, and inside of that file with ***mkt*** we are going to create to following directories (the mkt function remember that I have it defined in the ***~/.zshr*** to create those directories.). 

<p align = "center">
<img src = "/assets/images/img-solar/captura1.png">
</p>

## Recognition

First we send an icmp trace to see if we have a connection on the victim machine, and with the ttl i know this is a linux machine, remember that linux machines have ttl 64 and windows machines have ttl 128. 

<p align = "center">
<img src = "/assets/images/img-solar/captura2.png">
</p>

## Scanning

Basically i save it in the grepable format is that i have a function defined in the ~/.zshrc called extractports that indicating the name of the file shows me the ports in a more elegant way and copies the ports it to clipboard.

<p align = "center">
<img src = "/assets/images/">
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
