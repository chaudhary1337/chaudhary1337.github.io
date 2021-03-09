# Year of the Dog

Challenge Room

[Play](https://tryhackme.com/room/yearofthedog)

## Recon

```
╭─ ~ ──────────────────────────────────────────── 42s 23:08:25 ─╮
╰─❯ nmap script=default -sV -A -T4 10.10.151.166
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-25 23:07 EST
Nmap scan report for 10.10.151.166
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e4:c9:dd:9b:db:95:9e:fd:19:a9:a6:0d:4c:43:9f:fa (RSA)
|   256 c3:fc:10:d8:78:47:7e:fb:89:cf:81:8b:6e:f1:0a:fd (ECDSA)
|_  256 27:68:ff:ef:c0:68:e2:49:75:59:34:f2:bd:f0:c9:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Canis Queue
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.64 seconds
```

```
❯ searchsploit openssh 7.6p1
------------------------------------------------------ ---------------------------------
 Exploit Title                                        |  Path
------------------------------------------------------ ---------------------------------
OpenSSH 2.3 < 7.7 - Username Enumeration              | linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)        | linux/remote/45210.py
OpenSSH < 7.7 - User Enumeration (2)                  | linux/remote/45939.py
------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

```
❯ gobuster dir -u 10.10.151.166 -w "/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt" -x "html, php, txt"
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.151.166
[+] Threads:        10
[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt,html
[+] Timeout:        10s
===============================================================
2021/02/25 23:07:35 Starting gobuster
===============================================================
/index.php (Status: 200)
/assets (Status: 301)
/config.php (Status: 200)
Progress: 4448 / 220561 (2.02%)
```
![](https://i.imgur.com/ZQCbpJR.png)

The number in the queue looks interesting, with incognito, it shows something else each time, but with normal settings, shows 1. Interesting. I also checked out `config.php` but it showed nothing. the image of the flag may have some clue ... nope. nothing. writeup time :P


apparently its to do with sql shit. ehhh skip for now
