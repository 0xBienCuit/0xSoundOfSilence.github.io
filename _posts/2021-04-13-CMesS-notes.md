---
Author: 0xSoundOfSilence
date: "2021-04-13"
keywords:
- Markdown
- Example
lang: en
subject: Markdown
title: Try Hack Me - CMesS
---

# Write-Up CMesS

## Target information

* Name: cmess.thm
* IP: 10.10.23.53 (export ip=10.10.23.53)

## Enumeration
### nmap /port discovery

So we start off by rustscanning our target:

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d9:b6:52:d3:93:9a:38:50:b4:23:3b:fd:21:0c:05:1f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCvfxduhH7oHBPaAYuN66Mf6eL6AJVYqiFAh6Z0gBpD08k+pzxZDtbA3cdniBw3+DHe/uKizsF0vcAqoy8jHEXOOdsOmJEqYXjLJSayzjnPwFcuaVaKOjrlmWIKv6zwurudO9kJjylYksl0F/mRT6ou1+UtE2K7lDDiy4H3CkBZALJvA0q1CNc53sokAUsf5eEh8/t8oL+QWyVhtcbIcRcqUDZ68UcsTd7K7Q1+GbxNa3wftE0xKZ+63nZCVz7AFEfYF++glFsHj5VH2vF+dJMTkV0jB9hpouKPGYmxJK3DjHbHk5jN9KERahvqQhVTYSy2noh9CBuCYv7fE2DsuDIF
|   256 21:c3:6e:31:8b:85:22:8a:6d:72:86:8f:ae:64:66:2b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGOVQ0bHJHx9Dpyf9yscggpEywarn6ZXqgKs1UidXeQqyC765WpF63FHmeFP10e8Vd3HTdT3d/T8Nk3Ojt8mbds=
|   256 5b:b9:75:78:05:d7:ec:43:30:96:17:ff:c6:a8:6c:ed (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFUGmaB6zNbqDfDaG52mR3Ku2wYe1jZX/x57d94nxxkC
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: Gila CMS
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 3 disallowed entries 
|_/src/ /themes/ /lib/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

It's looks like our target only has 2 ports open: http & ssh.

Looking at the result of nmap, robot's.txt seems to be present. Let's see what the contents are:

```bash
──(root💀kali)-[~/ctf/CMesS]
└─# curl http://$ip/robots.txt                                                                                   2 ⚙
User-agent: *
Disallow: /src/
Disallow: /themes/
Disallow: /lib/
```


### Enumerating HTTP

Heading over to the website we just stumble on a basic unfinished website:

![](https://github.com/MrZeeQa/hugoblog/tree/master/images/webenum.png)

Let's enumerate the directories.

```bash
===============================================================
2021/04/13 16:55:50 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 276]
/.hta.php             (Status: 403) [Size: 276]
/.hta.html            (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/.htaccess.php        (Status: 403) [Size: 276]
/.htpasswd.html       (Status: 403) [Size: 276]
/.htaccess.html       (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.htpasswd.php        (Status: 403) [Size: 276]
/0                    (Status: 200) [Size: 3857]
/01                   (Status: 200) [Size: 4086]
/01.php               (Status: 200) [Size: 4086]
/01.html              (Status: 200) [Size: 4086]
/1.php                (Status: 200) [Size: 4086]
/1.html               (Status: 200) [Size: 4086]
/1                    (Status: 200) [Size: 4086]
/1x1                  (Status: 200) [Size: 4086]
/1x1.php              (Status: 200) [Size: 4086]
/1x1.html             (Status: 200) [Size: 4086]
/About                (Status: 200) [Size: 3343]
/about                (Status: 200) [Size: 3357]
/admin                (Status: 200) [Size: 1582]
/api                  (Status: 200) [Size: 0]   
/assets               (Status: 301) [Size: 322] [--> http://10.10.23.53/assets/?url=assets]
/author               (Status: 200) [Size: 3596]                                           
/blog                 (Status: 200) [Size: 3857]                                           
/category             (Status: 200) [Size: 3868]                                           
/cm                   (Status: 500) [Size: 0]                                              
/feed                 (Status: 200) [Size: 735]                                            
/fm                   (Status: 200) [Size: 0]                                              
/index                (Status: 200) [Size: 3857]                                           
/Index                (Status: 200) [Size: 3857]                                           
/lib                  (Status: 301) [Size: 316] [--> http://10.10.23.53/lib/?url=lib]      
/log                  (Status: 301) [Size: 316] [--> http://10.10.23.53/log/?url=log]      
/login                (Status: 200) [Size: 1582]                                           
/robots.txt           (Status: 200) [Size: 65]                                             
/Search               (Status: 200) [Size: 3857]                                           
/search               (Status: 200) [Size: 3857]                                           
/server-status        (Status: 403) [Size: 276]                                            
/sites                (Status: 301) [Size: 320] [--> http://10.10.23.53/sites/?url=sites]  
/src                  (Status: 301) [Size: 316] [--> http://10.10.23.53/src/?url=src]      
/tag                  (Status: 200) [Size: 3880]                                           
/tags                 (Status: 200) [Size: 3143]                                           
/themes               (Status: 301) [Size: 322] [--> http://10.10.23.53/themes/?url=themes]
/tmp                  (Status: 301) [Size: 316] [--> http://10.10.23.53/tmp/?url=tmp]      
                                                                                           
===============================================================
2021/04/13 16:59:41 Finished
===============================================================
```



