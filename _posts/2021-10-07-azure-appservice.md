---
layout: single
title: Configure dhcp server in linux with isc-dhcp-server
excerpt: "we are going to learn how to configure a dhcp server in linux using the isc-dhcp-server service, we are going to learn how to create subnets and make reserved ips."
date: 2021-10-20
classes: wide
header:
  teaser: /assets/images/img-dhcp/servidor-DHCP-e1511228735448.png
  teaser_home_page: true
  icon: /assets/images/img-dhcp/ics_logo.png
categories:
  - Networking
  - Linux
tags:
  - dhcp
  - isc-dhcp-server
  - linux
  - subnetting
---

<p align = "center">
<img src = "/assets/images/img-dhcp/servidor-DHCP-e1511228735448.png">
</p>

we are going to learn how to configure a dhcp server in linux using the isc-dhcp-server service, we are going to learn: create a fixed ip for our server machine, create subnets, exclude ips ranges, create an ips address concession and finally make a reserved ip for a specific host with its mac address.
 
## What is a dhcp server?

Dynamic Host Configuration Protocol (DHCP), is a network management protocol that allows us to automatically assign IP addresses to client computers, default gateways, and other network parameters. allowing them to use network services such as DNS, NTP and any communication protocol based on UDP or TCP.

To understand more clearly how the dhcp protocol works, the following diagram shows us how it works:

<p align = "center">
<img src = "/assets/images/img-dhcp/dhcp.png">
</p>

## Installing dhcp-server and adding fix ip address in our server machine.
The first thing we are going to do to configure the dhcp server is to install it with the following command shown in the image: (in my case I am using OS ubuntu server)

<p align = "center">
<img src = "/assets/images/img-dhcp/captura1.png">
</p>

In my case I am going to assign a fixed IP address on the server machine. For this we will go to the path "/ etc / netplan" and then we will enter with the «nano» editor in the configuration file that haswithin that route. Inside the file we will put the following parameters:

<p align = "center">
<img src = "/assets/images/img-dhcp/captura2.png">
</p>

After we are going to save the changes to the file, we use the command "netplan apply" so that the ip is added to our server. And we use the command "ifconfig" to see if the ip address has been apply.

<p align = "center">
<img src = "/assets/images/img-dhcp/captura3.png">
</p>

## Configuring our dhcp sever  

Once we have configured an ip for our server, we will go to the file "/etc/default/isc-dhcp.server", where it says "interfacesv4 (ipv4 version)" we put the name of the network card where it will listen to the requests that we are going to configure. In my case I want it to assign the ips addresses in the "enp0s8" adapter on the client machines.

<p align = "center">
<img src = "/assets/images/img-dhcp/captura4.png">
</p>

We are going to create a subnet declaration specifying the ip range, open dhcp configure file ("/etc/dhcp/dhcpd.conf"). what we have to do inside a subnet declaration and then with the command «range» we will first put the lowest ip (in my case it is 10) the second the highest ip ( which in my case is 200).

<p align = "center">
<img src = "/assets/images/img-dhcp/captura6.png">
</p>

To add any changes it is important to ***restart*** the dhcp service with the command:

```
sudo service isc-dhcp-server restart
```

In the same configuration file we can establish the default time that an IP address is going to be lend (defualt-lease-time) and the second would be the maximum rental time of an IP address (max-lease-time). In my case, in both, the time is 1 and 3 hours, which I have indicated in seconds.

<p align = "center">
<img src = "/assets/images/img-dhcp/captura8.png">
</p>

