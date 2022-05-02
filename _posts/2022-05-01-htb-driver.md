---
layout: single
title: HTB - Driver
excerpt: "Driver is a windows machine with easy level of difficulty both in exploitation phase and privilage escalation this machine is based to attacking printers on a corporate network, we will going to start to create..."
date: 2022-05-01
classes: wide
header:
  teaser: /assets/images/img-driver/portada.png
  teaser_home_page: true
  icon: /assets/images/
categories:
  - CTF
  - Pentest
tags:
  - Hack the box
  - Windows
  - Web
  - Printer exploitation
  - CVE
---

Driver is a windows machine with easy level of difficulty both in exploitation phase and privilage escalation this machine is based to attacking printers on a corporate network, we will going to start to create and upload a malicious scf file which allows to get user ntlmv2 hash which then we will crack it to gain access to the machine, and we will escalate privilage to exploiting the vulnerability called PrintNightMare.

<p align = "center">
<img src = "/assets/images/img-driver/portada.png">
</p>

Machine matrix:

<p align = "center">
<img src = "/assets/images/img-driver/matrix.png">
</p>

First of all we are going to create a directory with the name of the machine, and with the command “mkt” i am gonna create a following directories.

<p align = "center">
<img src = "/assets/images/img-driver/captura1.png">
</p>

The ***mkt*** command is a function that i defined on the file ***~/.zshrc*** that allows me to create the following directories, if you using bash in your case is the file ***~/.bashrc***.

<p align = "center">
<img src = "/assets/images/img-driver/captura2.png">
</p>

Now we are going to send one icmp trace to see if we get a connection with the target machine, and through the ttl we can know what OS is using the machine, remember that the Windows machine have 128 ttl and the Linux machine have 64 tll. And if you can asking why it’s output me 127 instead of 128? It’s because when we send icmp packets it not send directly with the target machine or server, it will send those packets some intermediate node before sending to the target machine and for this reason the ttl decreases by one digit, this process is also called ***traceroute***. You can try to check it with the flag ***-R*** on the ping command.

<p align = "center">
<img src = "/assets/images/img-driver/captura3.png">
</p>

Anyway in my machine i have defined a script called ***wichsystem*** that specifying the target ip address it Will output us if the machine is Windows or Linux through the ttl.

<p align = "center">
<img src = "/assets/images/img-driver/captura4.png">
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

Now we are going to perform nmap scanning to discover ports and other relevant information to the target machine, for this we are going to use the following parameters or flags:

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

the scan:

<p align = "center">
<img src = "/assets/images/img-driver/captura5.png">
</p>

And basically i save the scan in grapable format because i have defined in zshrc a function called ***extractports***, that specifying the file name shows me the ports and the IP address of the target machine in a much more elegant way and copies the ports to the clipboard. And this can be useful if there is a machine that has many ports enabled and we don't have to write those ports one by one to perform another scan.

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
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

Once we have discovered possible ports, we will perform another scan to recognize the version of the services that use each of these ports. In order to do that we going to use the following parameters or flags:

|Flags|Description |  
|-----|------------|
|-sCV |Means that we want to use some nmap scripts, in this case to discover the version and services that are running each of those ports. 
|-p   |To specify the ports.           |
|-oN  |Save the scan in nmap format. 

The scan:

<p align = "center">
<img src = "/assets/images/img-driver/captura10.png">
</p>

Remember that nmap have bunch of scripts that we can use, nmap scripts end in ***.nse*** extension (nmap script engine).

<p align = "center">
<img src = "/assets/images/img-ransom/locate.png">
</p>

Remember that nmap scripts have many categories that we can search for.

<p align = "center">
<img src = "/assets/images/img-ransom/categories.png">
</p>

Now we can use ***crackmapexec*** using the smb protocol to see what specific version of Windows have the target machine. And we can see that is a windows 10 enterprise version.

<p align = "center">
<img src = "/assets/images/img-driver/captura7.png">
</p>

Let’s check if we can list the shared resources with ***smbclient*** making use of a null session. And we get an access denied, so nothing interesting at the moment.

<p align = "center">
<img src = "/assets/images/img-driver/captura8.png">
</p>

We are going the check with another tool if it’s let us to use the null session, and this case nothing.

<p align = "center">
<img src = "/assets/images/img-driver/captura9.png">
</p>

Before we see that the target machine has the port 80 enabled, so with “whatweb” command we can use it as a “wappalyzer” to see the versions of the framework, web service and the programming language used by the website. hmm firmware update center inresting...

<p align = "center">
<img src = "/assets/images/img-driver/captura12.png">
</p>

So, when we try to access on the website it will ask us for credentials, and I try with some typical default credentials and it’s works. The default credential it was user “admin” and the password “admin”, so in this case the target machine has weak password to access on a private site.

<p align = "center">
<img src = "/assets/images/img-driver/captura13.png">
</p>

So we are in on the website and the wappalyzer reports few things, which is the frameworks and libraries that are using on this particular webpage, so anything interesting here to exploit.

<p align = "center">
<img src = "/assets/images/img-driver/captura14.png">
</p>

On the “Firmware Update” section we can see that we can upload a file, hmm interesting … 

<p align = "center">
<img src = "/assets/images/img-driver/captura15.png">
</p>

So with “searchsploit” tool we try to see if that have any exploit or vulnerability the service called “MFP firmware update center” we can't find anything related to that service.

<p align = "center">
<img src = "/assets/images/img-driver/captura16.png">
</p>

Before on the section "firmware upload" on the webpage it’s says that the file we upload it will be reviewed by a somebody and then uploaded to the page. Thinking a bit here we can try to use a malicious scf file. So what we can try is to upload the scf file to indicate that we are uploading a new firmware for the printer, and what we can do is that malicious file will load as an icon and on the victim side if the user only sees the icon of that file we can obtain the ntlmv2 hash of that user. More info about scf files on the following website.

<p align = "center">
<img src = "/assets/images/img-driver/captura17.png">
</p>

We are going to use this payload and the idea is that icon of the file it will load to our shared resource at the network that is in our attacker’s machine, in my case the share resource will be called “smbFolder”.

<p align = "center">
<img src = "/assets/images/img-driver/captura18.png">
</p>

Now we are going to upload the scf file on the target webpage.

<p align = "center">
<img src = "/assets/images/img-driver/captura19.png">
</p>

While we are uploading the file, we are going to create our smb server with ***impacket*** in our attacker machine, and we are going to specify a shared resource that is called the same name that we have specify on the csf file that is synchronized in the current working directory at the absolute path level, and since the target machine is windows 10 we are going to add support the version 2 of smb.

And once we upload the file, it should report the ntlm hash of the user who checked our file, in this case the user is “tony”.

<p align = "center">
<img src = "/assets/images/img-driver/captura20.png">
</p>

Once we obtain the user hash, we are going try to crack it with hashcat or john,  in this case using the rockyou dictionary, it doesn’t take too long to crack the hash and in this case the user “tony” has a very weak password.

<p align = "center">
<img src = "/assets/images/img-driver/captura21.png">
</p>

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-driver/captura6.png">
</p>
