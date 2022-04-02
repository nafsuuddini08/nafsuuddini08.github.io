---
layout: single
title: HTB - Secret
excerpt: "Secret is a linux machine with difficulty esay pulling in the exploitation phase when accessing the machine (which for me has not been easy, I will explain this later in the blog) and the escalation of privileges is at medium level of difficulty, this machine is vulnerable to RCE through jwt."
date: 2022-1-05
classes: wide
header:
  teaser: /assets/images/img-secret/portada.png
  teaser_home_page: true
  icon: /assets/images/
categories:
  - CTF
  - Pentest
tags:
  - Hack the box
  - Linux
---

Secret is a linux machine with difficulty esay pulling in the exploitation phase when accessing the machine (which for me has not been easy, I will explain this later in the blog) and the escalation of privileges is at medium level of difficulty, this machine is vulnerable to RCE through jwt.

<p align = "center">
<img src = "/assets/images/img-secret/portada.png">
</p>

Machine rating according to the people.

<p align = "center">
<img src = "/assets/images/img-secret/calificacion.png">
</p>

Machine matrix:

<p align = "center">
<img src = "/assets/images/img-secret/matrix.png">
</p>

First what we are going to do is create a directory with the name of the machine, and then with the command ***mkt*** i am going to create the following directories to organize the files and scripts. 

<p align = "center">
<img src = "/assets/images/img-secret/captura1.png">
</p>

Mkt is a function that i have defined in my ***~/.zshrc***, which is the following:

<p align = "center">
<img src = "/assets/images/img-secret/mkt.png">
</p>

First we send an icmp packet to check if we have connection with the target machine, and trough the TTL we can know what OS have the machine if is linux or windows. Remember that linux machine hava a TTL 64 and windows 128.

<p align = "center">
<img src = "/assets/images/img-secret/captura2.png">
</p>

And if we asking why on the output of the ping command have 63 instead of 64? it's because when we send an icmp packet does not send directly to the target machine, there are some intermediary nodes that pass that packet until arriving the target machine and this make the TTL decrease one unit, so thats why we have 63 instead of 64.

<p align = "center">
<img src = "/assets/images/img-secret/captura3.png">
</p>

Anyway, i have a command defined on my shell called ***wichSystem*** which is a script in python that specifying the IP address of the target machine and through the TTL it will output if the machine is linux or windows. And also this script makes much less noise and traffic than nmap to know the OS of the target machine.

<p align = "center">
<img src = "/assets/images/img-secret/ttl.png">
</p>

wichSystem script:

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
|-oN | Means that we want to save the scan in nmap format to not rescan again, you have more formats to save like nmap, xml, etc.

And here we can the result of the scan, and as we can see there is 3 ports on the target machine.

<p align = "center">
<img src = "/assets/images/img-secret/captura3.png">
</p>

## Scanning - Port Recognition

Once we have discovered possible ports, we will perform another scan to recognize the services and versions that use each of these ports. To order to do that we going to use the following parameters or flags:

|Flags|Description |  
|-----|------------|
|-sCV |Means that we want to use some nmap scripts, in this case to discover the version and services that are running each of those ports. 
|-p   |To specify the ports.           |
|-oN  |Save the scan in nmap format. 

Remember that nmap have bunch of scripts that we can use, nmap scripts ends in .nse extension (nmap script engine).

<p align = "center">
<img src = "/assets/images/img-secret/nse.png">
</p>

Remember that nmap scripts have many categories that we can search for.

<p align = "center">
<img src = "/assets/images/img-secret/categories.png">
</p>

Here we can see the result of the scan and we can the versions of the services that are running on those three ports, and it output that the target machine is an ubuntu but it does not specify anu version of ubuntu. 

<p align = "center">
<img src = "/assets/images/img-secret/captura7.png">
</p>

If we want to know the version of the ubuntu that the target machine is using, what we can do is copy the version of some services like apache or openssh that is using on the target machine and we can search in launchpad to see what version of ubuntu is used that particular version.

<p align = "center">
<img src = "/assets/images/img-secret/google.png">
</p>

And we see that it is a ubuntu focal, this will not help us much to exploit the machine, but it would be good for us, to know what machine we are attacking.

<p align = "center">
<img src = "/assets/images/img-secret/launchpad.png">
</p>

If we click where is says ***focal***, we can see what version of ubuntu supports focal and we can see which version of ubuntu have focal and in this case we can see that focal is in version 20.4 of ubuntu, and with this we can know that the target machine is usinf ubuntu 20.4 which in this case this version is LTS, this can then be confirmed when we gain access to the machine. 

<p align = "center">
<img src = "/assets/images/img-secret/launchpad2.png">
</p>

Before we have seen that there is an open port 80, so what can do is with the command ***whatweb*** to know if the webserver is using any cms, frameworks that is uses and its version, etc. We are also going to check the port 3000 that uses node.js. And at the moment we don't anything interesting here, if there is some framework that is vulnerable and that we can exploit.  

<p align = "center">
<img src = "/assets/images/img-secret/whatweb.png">
</p>

Visiting the websit we can see that the ***wappalyzer*** reports us that this website use node.js, wappalyzer is an extesion similar to the command whatweb. And i try to access to the website with the port 3000 and it's back me on the same website that has on the port 80, so at the moment nothing interesting on the port 3000.

<p align = "center">
<img src = "/assets/images/img-secret/captura8.png">
</p>

If we scroll down we can see a button to download to source code fo an API and it will download a zip file. With this we can guess that this website is using an API on the backend side and it's probably that is on the port 3000, we'll see later to figure it out.

<p align = "center">
<img src = "/assets/images/img-secret/captura9.png">
</p>

If we look the content of the website, we can see that it tells us that there is a field to register through the API and that the data is represented in json format.

<p align = "center">
<img src = "/assets/images/img-secret/captura11.png">
</p>

If we try to access on that particular path it will tell us not found, let's try to check with burpsuite whats is really going on behind. 

<p align = "center">
<img src = "/assets/images/img-secret/captura12.png">
</p>

In this case i am going togdfghdfgdfgbfvxbfgxt use curl, but you can do the same process with burpsuite. We see that when we send the request with POST it response with "name required", with this we can know that there is a name field that we need to put to register.

<p align = "center">
<img src = "/assets/images/img-secret/captura13.png">
</p>

If we visit the website and will continue scrolling, we see that it tells us that there is a login page on the API for authenticate with a valid user and the way in which the data is represented, which in this case is in json format.  

<p align = "center">
<img src = "/assets/images/img-secret/captura14.png">
</p>

If we try to access on that particular path it will say again that the following page is not found.

<p align = "center">
<img src = "/assets/images/img-secret/captura15.png">
</p>

So let's do the same process what have we done before, with curl i am going to send the same request with POST. So we can see that is output "email is required", so with that we know that there is email field to authenticate as an valid user in that particular API.

<p align = "center">
<img src = "/assets/images/img-secret/captura16.png">
</p>

Back in on the website, if we continue scrolling it shows us a path that is ***/api/priv*** that we dont have a access to.

<p align = "center">
<img src = "/assets/images/img-secret/captura17.png">
</p>

And through curl if i try to send a GET request i get back again the "access denied" message, and try to send the same request with POST and nothing happens.

<p align = "center">
<img src = "/assets/images/img-secret/captura18.png">
</p>

So what we can do now it's try to reguster in that API, so with curl we are going to send a POST request method specifying the URL, and with ***-H*** flag we are going to specify the data format which in this case is json, because remember that on the website we saw that the representation of the data in regiester form and login form is in json. And with the flag ***-d*** we are going to create a user specifying in a json format. And it will output the name of the user in json, so it's seems that the user is created, let's going to check. 

<p align = "center">
<img src = "/assets/images/img-secret/captura19.png">
</p>

So i'am going to send the same request to know if the user is created correctly, and it's says username and user email already exists, so it will created successfully.

<p align = "center">
<img src = "/assets/images/img-secret/captura20.png">
</p>

Before we saw some user credentials on the website in the example json body, so in this we are going to send the same request as will done before and without changing the email and password let's check is that user exists, and as you can see it outputs that this user is already exits.

<p align = "center">
<img src = "/assets/images/img-secret/captura21.png">
</p>

And sending the same request let's check if there is an admin user, and we can see that there is a admin user. 

<p align = "center">
<img src = "/assets/images/img-secret/captura22.png">
</p>

So on the website we see that if some user is login successfully on the API it will create a jwt for that user.

<p align = "center">
<img src = "/assets/images/img-secret/captura_nose.png">
</p>

Now, let's try to loggin with the user that we created and if it's login seccessfully we are going to receive a jwt. and as we can see i login successfully and it will output with the jwt token, and the it will create a new jwt if we login again.

<p align = "center">
<img src = "/assets/images/img-secret/captura23.png">
</p>

Now i'am going to try to login with the credentials that was shown on the web page, and it outputs that the password was wrong and we know that the followig email address exists, but we don't know the password.

<p align = "center">
<img src = "/assets/images/img-secret/captura24.png">
</p>

So with gobuester let's try to fuzzing the web page if we can see another path that is interesting to us, and we don't see anything interesting except the API path that we already saw.

<p align = "center">
<img src = "/assets/images/img-secret/captura25.png">
</p>

Now if we go on the website jwt.io and we paste the jwt that is created us with the user that we have created previously, it will show the data of that jwt on the decoded section. And we can see the username of that jwt which in my case is "test".

<p align = "center">
<img src = "/assets/images/img-secret/captura26.png">
</p>

For example, let's change the username field in my case i'am going to put the username theadmin and we can see that the jwt it will change.

<p align = "center">
<img src = "/assets/images/img-secret/captura27.png">
</p>

We can test with this jwt to see if we can authenticate with the admin user, but it still won't work since we need the correct signature to authenticate (secret token). we can test the jwt by removing the bit-secret field.

<p align = "center">
<img src = "/assets/images/img-secret/captura28.png">
</p>

So if we go back in the documentation on the website, on the private access route that we can't access as we saw before, we can send a request with the header ***auth-token*** that inside we will put the jwt in this case for user admin and if it works successfully it should output us in a json format on the role field that we are admin user, but at the moment will dont't have the jwt from the admin user.

<p align = "center">
<img src = "/assets/images/img-secret/priv.png">
</p>

And in the case that if we use our jwt with the user that we create it will output the following on json:

<p align = "center">
<img src = "/assets/images/img-secret/normal.png">
</p>

Now i'am going to send a GET request with the flag "-H" specifying the ***auth-token*** header with the token thet we generate in "jwt.io" for testing reasons to see if its's works, and it's output "invalid token" and it was obvious that it did not work because we need the secret token for the signature of the jwt of the admin user. 

<p align = "center">
<img src = "/assets/images/img-secret/captura30.png">
</p>

Let's try to send the same request with our jwt of the user that we create on the "/api/priv" route. And as you can see it will output in json format that we are normal user, we can pipe it to the command ***jq*** to see the data on the json structure (remember that we can do this same process in brupsuite). 

<p align = "center">
<img src = "/assets/images/img-secret/captura31.png">
</p>

In this case with ***wfuzz*** we are going to fuuzzing the ***/api*** route specifying the following dictionary to see if there are more routes in that specific route, remember that at the end of the url we need put the word "Fuzz", and this word it is an integrated word of wfuzz and what allows is to replace that word with the words in the dictionary. And this case it will output responses with 93 characters.  

<p align = "center">
<img src = "/assets/images/img-secret/captura32.png">
</p>

In this case we are going to specify in wfuzz that we don't responses with 93 characters and we will going to do that specifying with ***--hh*** means "hide characters". And as we can see that it reports a route called logs that we have not yet accessed. 

<p align = "center">
<img src = "/assets/images/img-secret/captura33.png">
</p>

