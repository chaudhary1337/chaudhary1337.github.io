# Biohazard

[Play](https://tryhackme.com/room/biohazard)

## 1. Introduction

We start with nmap for port scanning and do some web exploration meanwhile.

### 1.1. Port Scanning
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.111.73
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-22 00:26 EDT
Nmap scan report for 10.10.111.73
Host is up (0.21s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c9:03:aa:aa:ea:a9:f1:f4:09:79:c0:47:41:16:f1:9b (RSA)
|   256 2e:1d:83:11:65:03:b4:78:e9:6d:94:d1:3b:db:f4:d6 (ECDSA)
|_  256 91:3d:e4:4f:ab:aa:e2:9e:44:af:d3:57:86:70:bc:39 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Beginning of the end
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.24 seconds
```

### 1.2. Web Exploration

On the main page, we get two things:
- `/image/` directory
- `/mansionmain` web page

The image directory shows with an interesting file: `zombie`. We'll keep that in mind. Let's go to the mansion page

#### Mansion Room
The `/mansionmain` page source has another link as

```
	<!-- It is in the /diningRoom/ -->
```

Going there, we see 
```
        <!-- SG93IGFib3V0IHRoZSAvdGVhUm9***** -->
```

Is what we see in the source, along with a link to a file, that gives us the emblem flag.

This we can put in the slot.

![Dining room page where we get emblem](https://i.imgur.com/CaCN3hv.png)

We are redirected to this page,

![Dining room emblem page redirect, saying "Nothing happen"](https://i.imgur.com/Io9I4XQ.png)

Looks like one dead end. So, let's go back to the encoded string we saw in the comments in the dining room page. Using [Cyber Chef](http://icyberchef.com/) we get `How about the /teaRoom/`. Let's go there.


#### Tea Room
In the teaRoom, we get 
- `Barry also suggested that Jill should visit the /artRoom/`
- a link to `/teaRoom/master_of_******.html`

The second one shows us a flag! Just to be safe, I checked if I missed some thing in the source code. Nothing there. Onto the art room!

#### Art Room
![The map of interesting directories in the Art Room](https://i.imgur.com/mWupDdm.png)

We see a map, of which we already have covered some. Let's start with the barRoom

#### Bar Room
![Bar Room page](https://i.imgur.com/CeJcJPk.png)

We already have the flag with us. 

![Bar room redirected page with piano](https://i.imgur.com/5kKzeSV.png)

We see the READ link, which gives us another code that needs to be broken. Using our best friend [Cyber Chef](http://icyberchef.com/) we get the music sheet flag!

Putting this in, we get the golden emblem. Now, we put this in the emblem slot in the dining room. We get

`klfvg ks r wimgnd biz mpuiui ulg fiemok tqod. Xii jvmc tbkg ks tempgf tyi_hvgct_jljinf_kvc`


I tried rot13 and CyberChef, but they didn't work. Anohter common way to cipher is Vigenere Cipher, so I used [this tool](https://www.boxentriq.com/code-breaking/vigenere-cipher) to auto-decode without a key.

![Results from vigenere cipher tools](https://i.imgur.com/JLV11bT.png)

We get the key after a lot of guesswork. 

PS: Do try this yourself, because a similar pattern is found later.

Let's move to another room.

#### Dining Room 2F (Second Floor)
We see another code hidden in source-code. This time however, it does not look encoded. Instead, the length and the spaces tell me its a ROT version. ROT13 being the most common one, Let's try it!

Using [ROT13](https://rot13.com/) we get 

```
	<!-- You get the blue gem by pushing the status to the lower floor. The gem is on the diningRoom first floor. Visit ********.html -->       
```
And doing this, we get another flag! 

Let's move to tiger room.


#### Tiger Status Room
![The Tiger Status Room asking for a jewel](https://i.imgur.com/lEZhtJp.png)

We get the blue gem flag, and input it here. Also, if I do not mention about the source code comments or some shady stuff, its because it is not there. 

Anyways, we get
```
crest 1:
{key was here}
Hint 1: Crest 1 has been encoded twice
Hint 2: Crest 1 contanis 14 letters
Note: You need to collect all 4 crests, combine and decode to reavel another path
The combination should be crest 1 + crest 2 + crest 3 + crest 4. Also, the combination is a type of encoded base and you need to decode it
```

We'll come back to it then. 

#### Gallery Room

We see another similar story, the note linked in this page reads the crest 2.

```
crest 2:
{key was here}
Hint 1: Crest 2 has been encoded twice
Hint 2: Crest 2 contanis 18 letters
Note: You need to collect all 4 crests, combine and decode to reavel another path
The combination should be crest 1 + crest 2 + crest 3 + crest 4. Also, the combination is a type of encoded base and you need to decode it
```

Let's keep going. We have study room, armour room and attic. Last two ask for a shield and study asks for a helmet. We start with attic then.

#### Attic
We find crest 4 here

```
crest 4:
{key was here}
Hint 1: Crest 2 has been encoded twice
Hint 2: Crest 2 contanis 17 characters
Note: You need to collect all 4 crests, combine and decode to reavel another path
The combination should be crest 1 + crest 2 + crest 3 + crest 4. Also, the combination is a type of encoded base and you need to decode it
```

#### Armour Room
```
crest 3:
{key was here}
Hint 1: Crest 3 has been encoded three times
Hint 2: Crest 3 contanis 19 letters
Note: You need to collect all 4 crests, combine and decode to reavel another path
The combination should be crest 1 + crest 2 + crest 3 + crest 4. Also, the combination is a type of encoded base and you need to decode it
```

#### Making Sense of the Mess

We have all the 4 crests, although they do not make any sense right now. One we have each of them decoded, we need to put them together and then decode once more.

**Crest1**

CyberChef to the rescue!
![Cyberchef for crest1](https://i.imgur.com/ixVUdKl.png)
We see it has 2 iterations: base64-> base 32.

**Crest2**

Similarly, from base32 to base58.

**Crest3**
![CyberChef for crest3](https://i.imgur.com/CCMs4gg.png)

**Crest4**
![CyberChef for crest4](https://i.imgur.com/IcAqNqB.png)

And we now combine them. 

```
FTP user: hunter, FTP pass: you_cant_hide_forever
```

Hell yes! We are done with this stuff. Now, onto FTP.

### 1.3. FTP

```
┌──(kali㉿kali)-[/tmp]
└─$ ftp 10.10.111.73
Connected to 10.10.111.73.
220 (vsFTPd 3.0.3)
Name (10.10.111.73:kali): hunter
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 1002     1002         4096 Sep 20  2019 .
drwxrwxrwx    2 1002     1002         4096 Sep 20  2019 ..
-rw-r--r--    1 0        0            7994 Sep 19  2019 001-key.jpg
-rw-r--r--    1 0        0            2210 Sep 19  2019 002-key.jpg
-rw-r--r--    1 0        0            2146 Sep 19  2019 003-key.jpg
-rw-r--r--    1 0        0             121 Sep 19  2019 helmet_key.txt.gpg
-rw-r--r--    1 0        0             170 Sep 20  2019 important.txt
226 Directory send OK.

...

ftp> 
221 Goodbye.
```

Get all the files. Ez.

```
┌──(kali㉿kali)-[/tmp]
└─$ cat important.txt         
Jill,

I think the helmet key is inside the text file, but I have no clue on decrypting stuff. Also, I come across a /******_******/ door but it was locked.

From,
Barry
```

#### 1.3.1 Steganography
```
┌──(kali㉿kali)-[/tmp]
└─$ steghide --extract --stegofile 001-key.jpg 
Enter passphrase: 
wrote extracted data to "key-001.txt".

┌──(kali㉿kali)-[/tmp]
└─$ cat key-001.txt  
{key was here}
```

```
┌──(kali㉿kali)-[/tmp]
└─$ binwalk 003-key.jpg -e

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
1930          0x78A           Zip archive data, at least v2.0 to extract, uncompressed size: 14, name: key-003.txt
2124          0x84C           End of Zip archive, footer length: 22

┌──(kali㉿kali)-[/tmp]
└─$ cd _003-key.jpg.extracted 

┌──(kali㉿kali)-[/tmp/_003-key.jpg.extracted]
└─$ ls
78A.zip  key-003.txt

┌──(kali㉿kali)-[/tmp/_003-key.jpg.extracted]
└─$ cat key-003.txt          
{key was here}
```

```
┌──(kali㉿kali)-[/tmp]
└─$ strings 002-key.jpg 
JFIF
{key was here}
...
"*%%*424DD\
...
```

Combine all of them and decode using cyberchef. Then,

```
┌──(kali㉿kali)-[/tmp]
└─$ gpg helmet_key.txt.gpg
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
gpg: AES256.CFB encrypted data
gpg: encrypted with 1 passphrase
```

and we have

```
┌──(kali㉿kali)-[/tmp]
└─$ cat helmet_key.txt
helmet_key{yee_haww}
```

very nice. Back to Web Exploration huh?

### 1.4. Web Exploration - Again

Going to the closet room, we find two more pieces of information:

- SSH password: `*_*****_*****`
- `wpbwbxr wpkzg pltwnhro, txrks_xfqsxrd_bvv_fy_******_***`

Using [this automatic solver](https://www.boxentriq.com/code-breaking/vigenere-cipher) as we saw before, we get: `weasker login password stars members are my ****** ***`


Also, recall that the last time, we couldn't epxlore the study room - because it required the helmet key which we didnt have. Exploring that, we get a `doom.tar.gz` file. Very nice.


```
┌──(kali㉿kali)-[/tmp]
└─$ tar -xzvf doom.tar.gz 
eagle_medal.txt
   
┌──(kali㉿kali)-[/tmp]
└─$ cat eagle_medal.txt 
SSH user: umbrella_guest
```

Ha! Easy!

## 2. Foothold
Ah. Finally. We are done with the chaos.


```
┌──(kali㉿kali)-[/tmp]
└─$ ssh umbrella_guest@10.10.111.73

{bla bla bla nothing interezting}

umbrella_guest@umbrella_corp:~$ 
```

Doing some exploration, we come across:

```
umbrella_guest@umbrella_corp:/home/weasker$ cat weasker_note.txt 
Weaker: Finally, you are here, Jill.
Jill: Weasker! stop it, You are destroying the  mankind.
Weasker: Destroying the mankind? How about creating a 'new' mankind. A world, only the strong can survive.
Jill: This is insane.
Weasker: Let me show you the ultimate lifeform, the {enemy name here}.

({enemy name here} jump out and kill Weasker instantly)
(Jill able to stun the tyrant will a few powerful magnum round)

Alarm: Warning! warning! Self-detruct sequence has been activated. All personal, please evacuate immediately. (Repeat)
Jill: Poor bastard

```

We also find

```
umbrella_guest@umbrella_corp:~/.jailcell$ cat chris.txt 
Jill: Chris, is that you?
Chris: Jill, you finally come. I was locked in the Jail cell for a while. It seem that weasker is behind all this.
Jil, What? Weasker? He is the traitor?
Chris: Yes, Jill. Unfortunately, he play us like a damn fiddle.
Jill: Let's get out of here first, I have contact brad for helicopter support.
Chris: Thanks Jill, here, take this MO Disk 2 with you. It look like the key to decipher something.
Jill: Alright, I will deal with him later.
Chris: see ya.

MO disk 2: albert 
```

## 3. Priv-Esc

```
umbrella_guest@umbrella_corp:~$ sudo -l
[sudo] password for umbrella_guest: 
Sorry, user umbrella_guest may not run sudo on umbrella_corp.
umbrella_guest@umbrella_corp:~$ file / -type f -perm -u=s 2>/dev/null
```

Both of them return nothing. :(

How about the user weasker? Well we have something from before that we didn't use uptill now.

`weasker login password stars members are my ****** ***`

PS: use the same concept as we did for the html page above. And ... we are in!

```
umbrella_guest@umbrella_corp:/home/weasker$ su weasker
Password: 
weasker@umbrella_corp:~$ 
```

```
weasker@umbrella_corp:~$ sudo -l
Matching Defaults entries for weasker on umbrella_corp:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User weasker may run the following commands on umbrella_corp:
    (ALL : ALL) ALL

```

This is kinda cute. We are done! Such a long room damn. Fun though :D

