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

The first thing we are going to do is to create a file with the machine name, and inside of that file with ***mkt*** we are going to create to following directories (the mkt function remember that I have it defined in the ***~/.zshr*** to create those directories.). 

<p align = "center">
<img src = "/assets/images/img-secret/captura1.png">
</p>

## recognition

First we send an icmp trace to see if we have a connection on the victim machine, and with the ttl i know this is a linux machine, remember that linux machines have ttl 64 and windows machines have ttl 128. In my machine I have defined a python script that through the ttl reports me if it is a windows or linux machine in a more elegant way, in the case if we don't remember which ttl belongs to which OS.

<p align = "center">
<img src = "/assets/images/img-secret/captura2.png">
</p>

wichsystem script:

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

## Scanning - Ports recognition

I am going to perform a tcp syn scan by adding the ***min-rate*** parameter to make the scan go as fast as possible, and the evidence of the scan I will save it in grepable format.

```
# Nmap 7.92 scan initiated Thu Dec 30 22:42:19 2021 as: nmap -p- -sS --min-rate 5000 --open -vvv -n -Pn -oG allports 10.10.11.120
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.120 ()   Status: Up
Host: 10.10.11.120 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 3000/open/tcp//ppp/// Ignored State: closed (65532)
# Nmap done at Thu Dec 30 22:42:30 2021 -- 1 IP address (1 host up) scanned in 11.72 seconds
```
Basically i save it in the grepable format is that i have a function defined in the ***~/.zshrc*** called ***extractports*** that indicating the name of the file shows me the ports in a more elegant way and copies the ports it to clipboard.

<p align = "center">
<img src = "/assets/images/img-secret/captura3.png">
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
# Nmap 7.92 scan initiated Thu Dec 30 22:44:07 2021 as: nmap -sCV -p22,80,3000 -oN targeted 10.10.11.120
Nmap scan report for 10.10.11.120
Host is up (0.051s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 97:af:61:44:10:89:b9:53:f0:80:3f:d7:19:b1:e2:9c (RSA)
|   256 95:ed:65:8d:cd:08:2b:55:dd:17:51:31:1e:3e:18:12 (ECDSA)
|_  256 33:7b:c1:71:d3:33:0f:92:4e:83:5a:1f:52:02:93:5e (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-title: DUMB Docs
|_http-server-header: nginx/1.18.0 (Ubuntu)
3000/tcp open  http    Node.js (Express middleware)
|_http-title: DUMB Docs
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 30 22:44:22 2021 -- 1 IP address (1 host up) scanned in 15.32 seconds
```
As there is a web server on port 80 with ***whatweb*** we do a small recognition as if it were wappalyzere extension, to know the version of the web service, cms, etc.

<p align = "center">
<img src = "/assets/images/img-secret/captura4.png">
</p>

We access the web and the wappalyzer reports the following information about the web.

<p align = "center">
<img src = "/assets/images/img-secret/webw.png">
</p>

in the web it indicates us a route to register on the website, I have tried to access to that route but it does not allow me to accede, it can be that we need credentials of user admin.

<p align = "center">
<img src = "/assets/images/img-secret/register.png">
</p>

We also have a path to log in, but we can't access it.

<p align = "center">
<img src = "/assets/images/img-secret/login.png">
</p>

And in this path it tells us that only admins can access to see if there are other admin users and i imagine that there is something else in this path, at the moment we don't have any valid credentials so we can't do anything.

<p align = "center">
<img src = "/assets/images/img-secret/priv.png">
</p>

And we go further down in the documentation section we can see something interesting, I am already thinking of some attacks that we can perform.

<p align = "center">
<img src = "/assets/images/img-secret/jwt.png">
</p>

Well we have an option to download the repository of this project.

<p align = "center">
<img src = "/assets/images/img-secret/download.png">
</p>

And these are the files contained in the repo, I'm seeing some interesting files and directories.

<p align = "center">
<img src = "/assets/images/img-secret/captura11.png">
</p>

But before looking at the content of each directory in that repo i go to see if there is something in the code of the website, and i don't see anything useful.

<p align = "center">
<img src = "/assets/images/img-secret/captura6.png">
</p>

I am going to proceed with the fuzzing process to see if there is any more route on the website and i don't see anything interesting, only the api but i can't access because it's says access denied and i don't have the credentials at the moment.

<p align = "center">
<img src = "/assets/images/img-secret/captura8.png">
</p>

In the ***/etc/hots*** file i have applied a virtual hosting to see if there is any difference in the website with the domain, but nothing.

<p align = "center">
<img src = "/assets/images/img-secret/captura10.png">
</p>

Since the ssh port is open on the victim machine i wanted to check if the user ***anonymous*** is enabled but in this case it is not enabled.

<p align = "center">
<img src = "/assets/images/img-secret/captura7.png">
</p>

Going back to the repository that i downloaded earlier, before we saw that there was the ***.git*** folder, what i have done is first look at the logs of the commits that have been made in that repository and we can see the user who has made the commits and his email, and we can see that there is a commit that says ***removed .env for security reasons*** let's see what changes were made in that commit.

<p align = "center">
<img src = "/assets/images/img-secret/captura12.png">
</p>

And with the command ***git diff*** we can see that there is a ***secret token***, hmm interesting i can now imagine how to use this token that was deleted from that code.

<p align = "center">
<img src = "/assets/images/img-secret/captura13.png">
</p>

By reading the following article we can do the following: [The article](https://en.internetwache.org/dont-publicly-expose-git-or-how-we-downloaded-your-websites-sourcecode-an-analysis-of-alexas-1m-28-07-2015/)

Reading that article, what we can do instead of seeing the changes of the repo one by one, we can dump the old versions of this git repository, for this there is a tool called ***GitTools*** that basically allows us to do this that i just mentioned and more.

<p align = "center">
<img src = "/assets/images/img-secret/github.png">
</p>

Ok, first we go to where we have this tool installed (in my case in / opt) and we are going to use the following command: ***/ opt / GitTools / Extractor / extractor.sh local-web / dump (specify the name of the folder)***. 

And as we can see, we already have the previous versions of that repository (and we know it from the hashes of the commits that have been assigned) and now we can view the content of those directories more easily. 

<p align = "center">
<img src = "/assets/images/img-secret/captura14.png">
</p>

And by accessing one of the directories that we have just extracted, we can see that it is the same structure of files and folders that we have seen previously. 

<p align = "center">
<img src = "/assets/images/img-secret/captura15.png">
</p>

by accessing the route directory we can find the following javascript files (which is the source code of the API that was mentioned on the website):

<p align = "center">
<img src = "/assets/images/img-secret/route.png">
</p> 

auth.js file content:

```js
const router = require('express').Router();
const User = require('../model/user');
const bcrypt = require('bcryptjs')
const jwt = require('jsonwebtoken')
const { registerValidation, loginValidation} = require('../validations')

router.post('/register', async (req, res) => {

    // validation
    const { error } = registerValidation(req.body)
    if (error) return res.status(400).send(error.details[0].message);

    // check if user exists
    const emailExist = await User.findOne({email:req.body.email})
    if (emailExist) return res.status(400).send('Email already Exist')

    // check if user name exist 
    const unameexist = await User.findOne({ name: req.body.name })
    if (unameexist) return res.status(400).send('Name already Exist')

    //hash the password
    const salt = await bcrypt.genSalt(10);
    const hashPaswrod = await bcrypt.hash(req.body.password, salt)


    //create a user 
    const user = new User({
        name: req.body.name,
        email: req.body.email,
        password:hashPaswrod
    });

    try{
        const saveduser = await user.save();
        res.send({ user: user.name})
    
    }
    catch(err){
        console.log(err)
    }

});


// login 

router.post('/login', async  (req , res) => {

    const { error } = loginValidation(req.body)
    if (error) return res.status(400).send(error.details[0].message);

    // check if email is okay 
    const user = await User.findOne({ email: req.body.email })
    if (!user) return res.status(400).send('Email is wrong');

    // check password 
    const validPass = await bcrypt.compare(req.body.password, user.password)
    if (!validPass) return res.status(400).send('Password is wrong');


    // create jwt 
    const token = jwt.sign({ _id: user.id, name: user.name , email: user.email}, process.env.TOKEN_SECRET )
    res.header('auth-token', token).send(token);

})

router.use(function (req, res, next) {
    res.json({
        message: {

            message: "404 page not found",
            desc: "page you are looking for is not found. "
        }
    })
});

module.exports = router
```
forgot.js file content:

```js
st router = require('express').Router();
const verifytoken = require('./verifytoken')
const User = require('../model/user');

router.get('/priv', verifytoken, (req, res) => {
    // res.send(req.user)

    const userinfo = { name: req.user }

    const name = userinfo.name.name;

    if (name == 'theadmin') {
        res.json({
            role: {
                role: "you are admin",
                desc: "{path to the binary}"
            }
        })
    }
    else {
        res.json({
            role: {
                role: "not enough privilages",
                desc: userinfo.name.name
            }
        })
    }

})


module.exports = router
```
private.js file content:

```js
onst router = require('express').Router();
const verifytoken = require('./verifytoken')
const User = require('../model/user');

router.get('/priv', verifytoken, (req, res) => {
   // res.send(req.user)

    const userinfo = { name: req.user }

    const name = userinfo.name.name;
    
    if (name == 'theadmin'){
        res.json({
            role:{

                role:"you are admin", 
                desc : "{flag will be here}"
            }
        })
    }
    else{
        res.json({
            role: {
                role: "you are normal user",
                desc: userinfo.name.name
            }
        })
    }

})

router.use(function (req, res, next) {
    res.json({
        message: {

            message: "404 page not found",
            desc: "page you are looking for is not found. "
        }
    })
});


module.exports = router
```

verifytoken.js file content:

```js
const jwt = require("jsonwebtoken");

module.exports = function (req, res, next) {
    const token = req.header("auth-token");
    if (!token) return res.status(401).send("Access Denied");

    try {
        const verified = jwt.verify(token, process.env.TOKEN_SECRET);
        req.user = verified;
        next();
    } catch (err) {
        res.status(400).send("Invalid Token");
    }
};
```
Another file that i found interesting is to validate users when they register, validations.js file contet:

```js
const Joi = require('@hapi/joi')


// register validation 

const registerValidation = data =>{
    const schema = {
        name: Joi.string().min(6).required(),
        email: Joi.string().min(6).required().email(),
        password: Joi.string().min(6).required()
    };

    return Joi.validate(data, schema)
}

// login validation

const loginValidation = data => {
    const schema2 = {
        email: Joi.string().min(6).required().email(),
        password: Joi.string().min(6).required()
    };

    return Joi.validate(data, schema2)
}


module.exports.registerValidation = registerValidation
module.exports.loginValidation = loginValidation
```

Upcoming more info about this machine....

