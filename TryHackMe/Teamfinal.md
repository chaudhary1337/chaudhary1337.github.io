# Teamfinal

[Play](https://tryhackme.com/room/teamcw)

## Recon

As usual, we start off with `nmap` and `gobuster` scans. 

`❯ nmap -sC -sV -A 10.10.82.31`

`❯ gobuster dir -u 10.10.82.31 -w "/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt" -x "html,php,txt"`

While they run, let's check out the webpage itself. Looks like the default page.

![](https://i.imgur.com/YFJ7Hhw.png)

We see two things: that bug and the `team.thm` thing. The bug link is nothing special. Let's add `team.thm` to our host list. `nmap` is finished, and we can kill gobuster scan; since it does not look any useful.

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

Ah. we see a proper page up here. Looking at the source, we see `/assets` and `/images`. I don't like stego challenges, so I'll keep it for the end.

Let's fire up `gobuster` again, on `team.thm` this time. While that runs, I tried looking around in the website, but nothing special. Same goes for trying out `anon` login in `ftp`. Searchsploit did not return anything intersting. 

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

Okay. One by one.

- `-c` flag is for colorful output
- `--hw` stands for `hide words`, probably. It was showing a ton of rows with `977`, so I hardcoded :D
- `-w` wordlist. If you are in kali, you already have it saved.
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

Interesting times. Scripts I can't access, so i'll leave it be. We can run another `gobuster` on `team.thm/scripts`, but that is for later. `robots.txt` is always interesting.

![](https://i.imgur.com/0u3oCyW.png)

Is what is shows. Lmao. Probably the ftp username? Password we can bruteforce. This is another thing we can try.

Pending paths:
- images stego
- gobuster on `/scripts`
- bruteforce `ftp` using `dale` as username.

However, there are tastier things for us right now. Let's go to `dev.team.thm` as given by `wfuzz`. Do not forget to include it in the `/etc/hosts`!

![](https://i.imgur.com/L0mKUUj.png)

Huh. Let's follow that. It redirects us to the url: `http://dev.team.thm/script.php?page=teamshare.php`

![](https://i.imgur.com/wnQJ8Ns.png)


Both of these combined, this looks very much like a `LFI`. Let's try `http://dev.team.thm/script.php?page=../../../../etc/passwd`. Since Apache is around `/var/www/html/`.

![](https://i.imgur.com/AAdqDXG.png)


Works! We see `dale`, `gyles` and some `ftpuser`. Interesting. Our first target is then to get into `dale`.

## Foothold

