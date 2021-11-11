---
layout: single
title: Nmap Guide
excerpt: "we are going to learn how to scan a network or a host using nmap, we are going to see some simple nmap commands and commands that help us to pentest. "
date: 2021-11-09
classes: wide
header:
  teaser: /assets/images/img-nmap/nmap.jpg
  teaser_home_page: true
  icon: /assets/images/img-nmap/
categories:
  - Networking
  - Pentest
tags:
  - Nmap
  - Kali linux
  - Hack the box
---

<p align = "center">
<img src = "/assets/images/img-nmap/nmap.jpg">
</p>

we are going to learn how to scan a network or a host using nmap, we are going to see some simple nmap commands and commands that help us to pentest, and we are going to use some nmap scripts to speed up the scanning process.

## What is nmap?

Nmap is a very powerful tool that allows us to scan networks and hots to detect possible ports that are open or vulnerable, and also this tool will be very helpful if we are learning pentest or if you want become a hacker.

## Nmap simple commands and parameters

Use the command "nmap -help" to see the parameters and options offered by nmap, or you can use the command "man nmap" to see the full documentation.

<p align = "center">
<img src = "/assets/images/img-nmap/captura1.png">
</p>

Using nmap commands:

```
$ nmap [Scan Type(s)] [Options] {target specification}
```

This is a basic scan to find all open ports and running services (this scan is not very good if we want to speed up the process and we will make a lot of noise on the network if we are attackers).

<p align = "center">
<img src = "/assets/images/img-nmap/captura2.png">
</p>

We can do the same scan with the domain name.

<p align = "center">
<img src = "/assets/images/img-nmap/captura3.png">
</p>

If we want to report information about the machine in the scanning process we will use the parameter "-v" (from verbose). it is advisable to use this parameter to not wait for the scan to finish and meanwhile we can speed up the work. 

<p align = "center">
<img src = "/assets/images/img-nmap/captura4.png">
</p>

If we do a triple verbose we will get more information about the machine.

<p align = "center">
<img src = "/assets/images/img-nmap/captura5.png">
</p>

We can scan several hots by puting the ip.

<p align = "center">
<img src = "/assets/images/img-nmap/captura6.png">
</p>

We can perform the same scan of several hosts using the last octet of the ip address.

<p align = "center">
<img src = "/assets/images/img-nmap/captura7.png">
</p>

Escaneo de una subred, nos reporta los puertos abiertos de esa subred y los equipos que están activos en esa subred. (hay que recordar si hacemos estos tipos de escaneo vamos a hacer más ruido en la red)

<p align = "center">
<img src = "/assets/images/img-nmap/captura8.png">
</p>

Scanning with a range of ip addresses.

<p align = "center">
<img src = "/assets/images/img-nmap/captura9.png">
</p>

Scan the list of hosts that are in a file. In my case what i created a file and indicate the ip address, the localhost and the dns of htb.

<p align = "center">
<img src = "/assets/images/img-nmap/captura10.png">
</p>

Now we will use the parameter "-iL (Input from hosts/network list)" and specify the file that we have created, to scan all IP addresses that we specify in the file. 

<p align = "center">
<img src = "/assets/images/img-nmap/captura11.png">
</p>

Scan the OS used by the target machine with the parameter "-O (os detection)". 

<p align = "center">
<img src = "/assets/images/img-nmap/captura12.png">
</p>

With the "-A" parameter it shows more information about the machine's OS. this parameter is interesting because it shows us the ssh-hostkey that is to authenticate to connect in the ssh protocol, in this case this key is public. and below we can see the traceroute that would be the intermediary nodes where they have passed the packets that we have sent.

<p align = "center">
<img src = "/assets/images/img-nmap/captura13.png">
</p>

Scan a host to detect if the machine uses packet filters or firewall. In my case it is not filtered.

<p align = "center">
<img src = "/assets/images/img-nmap/captura14.png">
</p>

To view all active hosts within the subnet, this command allows you to ping all hosts on the subnet.

<p align = "center">
<img src = "/assets/images/img-nmap/captura15.png">
</p>

Print hotspot interfaces and routes to the terminal. You can find out the host interface and route information with nmap by using the "-iflist" option.

<p align = "center">
<img src = "/assets/images/img-nmap/captura16.png">
</p>

## Difference between TCP connect scan and SYN scan

Before using these types of scanning we must know how the TCP protocol works, the TCP protocol (Transmission Control Protocol) for example if we want to access a web site from our computer to the server where the web site is hosted, tcp allows how we must start the communication and how we can access that page. the rules that the tcp protocol uses to start a communication is with "3 wey handshake". first the client sends a request called "syn", the server resive the request and responds by sending the client a packet called "syn ack" and then the client sends another packet called "ack" so that the server receives the message that the client is successfully communicating with the server.

<p align = "center">
<img src = "/assets/images/img-nmap/tcp.png">
</p>

Scan specific ports on the subnet to see if any hosts within the subnet have open ports 80 and 443 via tcp. With the "-sT" parameter we specify that we are going to use the full TCP connection (full open scan), and the magic of this is with the TCP protocol using the "3 way handshake". using the 3 wey handshake nmap can find out if the ports are open or not.

<p align = "center">
<img src = "/assets/images/img-nmap/captura17.png">
</p>




