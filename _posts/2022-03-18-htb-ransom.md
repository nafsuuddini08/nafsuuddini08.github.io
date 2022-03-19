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

The scan:

```
# Nmap 7.92 scan initiated Thu Mar 17 16:51:23 2022 as: nmap -p- -sS --min-rate 5000 --open -vvv -n -Pn -oG allports 10.10.11.153
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.153 ()   Status: Up
Host: 10.10.11.153 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///        Ignored State: closed (65533)
# Nmap done at Thu Mar 17 16:51:35 2022 -- 1 IP address (1 host up) scanned in 12.01 seconds
```

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

Once the scan is finish we can see the versions of the services, and it output that the target machine is an ubuntu but it does not specify anu version of ubuntu.

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

If we want to know the version of the ubuntu that the target machine is using, what we can do is copy the version that of some services like apache or openssh that is using on the target machine and we can search in launchpad to see what version of ubuntu is used that particular version.

<p align = "center">
<img src = "/assets/images/img-ransom/captura9.png">
</p>

And we see that it is a ubuntu focal, this will not help us much to exploit the machine, but it would be good for us, to know what machine we are attacking.

<p align = "center">
<img src = "/assets/images/img-ransom/captura10.png">
</p>

Before we have seen a port 80 on the scan process, what we can so is using the command ***whatweb*** to do a little recognition, to know if the website is using any cms or some particular frameworks. And we can see that the website is using a old version of jquery that can be vulnerable to xss and prototype pollution attack, and we can see that the website is using laravel, that we will have to keeping that in mind these informations for the exploitation phase.

<p align = "center">
<img src = "/assets/images/img-ransom/captura11.png">
</p>

And if we access to the website and we can see an authentication panel that asks us to put a password. btw, we can use the wappalyzer extension in our browser, which is the same when we have used the whatweb command.

<p align = "center">
<img src = "/assets/images/img-ransom/captura12.png">
</p>

We are going to check with the typical defualt passwords and didn't work.

<p align = "center">
<img src = "/assets/images/img-ransom/captura13.png">
</p>

What we can do is see if the website is vulnerable to sql injections, for this we are going to edit a little bit the html to the website and we are going to modify the password field to be able to see what we are writing.

<p align = "center">
<img src = "/assets/images/img-ransom/captura14.png">
</p>

We try a simple sql injection and we can see that is not vulnerable to sql injections.

<p align = "center">
<img src = "/assets/images/img-ransom/captura15.png">
</p>

Now what we can do is open burpsuite to intercept the request and manipulate them. Remembe that burpsuite is act like proxy between your browser and the web server.

<p align = "center">
<img src = "/assets/images/img-ransom/captura16.png">
</p>

We put any password in the password field and click login on the website to capture the request before sending it to the web server. And we can two cookies and we can see that is using laravel session cookie, another way to to know that the target website is using laravel framework. and on the other side is using cross-site request forgery token (xsrf). 

<p align = "center">
<img src = "/assets/images/img-ransom/captura17.png">
</p>

We see that it is using and api behind the login form and now because it's going into this api this xsrf token it's not to useful. There's also a second thing that happens in a lot laravel forms that's if it not going to api it also normally likes passing in ***&_token*** parameter which is another xsrf thing, but in this case it not having this parameter and also having api in the url mean's we're hitting the api middleware of laravel.

So what we could have done is save this burpsuite request and with sqlmap make several sql injections to check if it is injectable or not, but as we have seen before, the website it's not vulnerable to sql injections. So if we try to send the request it says "invalid password".

<p align = "center">
<img src = "/assets/images/img-ransom/captura18.png">
</p>

So what we can try is to change the request method to send the same data but in  ***POST***, so in order to do that we right click and click where is says ***change request method***. And when sending the request and it will output the status http code 405, so by post we do not see information that can be useful to us. 
<p align = "center">
<img src = "/assets/images/img-ransom/captura19.png">
</p>

What we can do is to change POST to GET, but keeping the same format as post. And we can see that it returns the request in json format, and we get the status code 422, and this happen because the ***content-type*** it doesn't in json format. So what we're going to do is change the content-type in json and put the password field in json format.

<p align = "center">
<img src = "/assets/images/img-ransom/captura20.png">
</p>



<p align = "center">
<img src = "/assets/images/img-ransom/captura21.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

<p align = "center">
<img src = "/assets/images/img-ransom/captura6.png">
</p>

