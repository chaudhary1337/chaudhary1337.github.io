# CTF Collection Vol. 2

![Challange on THM](https://i.imgur.com/nS97pns.png)

Challenge home page.
![Home page of the challenge](https://i.imgur.com/IrNcLcf.png)

## 0. Scanning & Enumeration
### Port Scanning
```
PORT   STATE    SERVICE VERSION
6/tcp  filtered unknown
22/tcp open     ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 1b:c2:b6:2d:fb:32:cc:11:68:61:ab:31:5b:45:5c:f4 (DSA)
|   2048 8d:88:65:9d:31:ff:b4:62:f9:28:f2:7d:42:07:89:58 (RSA)
|_  256 40:2e:b0:ed:2a:5a:9d:83:6a:6e:59:31:db:09:4c:cb (ECDSA)
80/tcp open     http    Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/VlNCcElFSWdTQ0JKSUVZZ1dTQm5JR1VnYVNCQ0lGUWdTU0JFSUVrZ1p5QldJR2tnUWlCNklFa2dSaUJuSUdjZ1RTQjVJRUlnVHlCSklFY2dkeUJuSUZjZ1V5QkJJSG9nU1NCRklHOGdaeUJpSUVNZ1FpQnJJRWtnUlNCWklHY2dUeUJUSUVJZ2NDQkpJRVlnYXlCbklGY2dReUJDSUU4Z1NTQkhJSGNnUFElM0QlM0Q=
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: 360 No Scope!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

I do not know about port 6, but we see some interesting stuff in robots.txt right away.

### Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.9.174 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 'php,html,txt' -t 32 -q

/login                (Status: 301) [Size: 310] [--> http://10.10.9.174/login/]                   
/index                (Status: 200) [Size: 94328]                              
/index.php            (Status: 200) [Size: 94328]                              
/button               (Status: 200) [Size: 39148]                              
/static               (Status: 200) [Size: 253890]                             
/cat                  (Status: 200) [Size: 62048]                              
/small                (Status: 200) [Size: 689]                                
/who                  (Status: 200) [Size: 3847428]                            
/robots               (Status: 200) [Size: 430]                                
/robots.txt           (Status: 200) [Size: 430]                                
/iphone               (Status: 200) [Size: 19867]                              
/game1                (Status: 301) [Size: 310] [--> http://10.10.9.174/game1/]
/egg                  (Status: 200) [Size: 25557]                              
/dinner               (Status: 200) [Size: 1264533]                            
/ty                   (Status: 200) [Size: 198518]                             
/ready                (Status: 301) [Size: 310] [--> http://10.10.9.174/ready/]
/saw                  (Status: 200) [Size: 156274]                             
/game2                (Status: 301) [Size: 310] [--> http://10.10.9.174/game2/]
/wel                  (Status: 200) [Size: 155758]                             
/free_sub             (Status: 301) [Size: 313] [--> http://10.10.9.174/free_sub/]
/nicole               (Status: 200) [Size: 367650]                                
/server-status        (Status: 403) [Size: 292]
```

I let this scan run, and explored the tasks one by one - albeit randomly.

### Easter 1
![robots.txt web page containing two flags](https://i.imgur.com/cMUbb8J.png)

Shove the hex thingy into [CyberChef](http://icyberchef.com/). 

### Easter 2
Take the disallowed directory name, and put it as such:

`base64 -> base64 -> remove spaces -> base64 -> remove spaces -> base64`

Feel free to use python3 snippet, `''.join(x.strip().split())`, where x is the string to remove the spaces.

Go to the directory and view the source code.

### Easter 3
`/login/` source code :D

### Easter 4
I used sqlmap to do the attacks automatically.
```
Database: THM_*****_**
Table: nothing_inside
[1 entry]
+-------------------------+
| Easter_4                |
+-------------------------+
| THM{noice}              |
+-------------------------+
```

### Easter 5
Continuing on the previous one, look for the database dump, crack the hash using hashcat. The mode is 0 aka md5sum.

### Easter 6

The hint says "Look out for the response header".
```
┌──(kali㉿kali)-[/tmp]
└─$ curl -i 10.10.9.174 -s | grep -v "Easter 14" | grep -i "easter"
Busted: Hey, you found me, take this Easter 6: THM{yee}
        <!--! Easter 17-->
        <h3> Hey! I got the easter 20 for you. I leave the credential for you to POST (username:DesKel, password:heIsDumb). Please, I beg you. Don't let him know.</h3>
```

Here, 
- curl
    - -i gets the response header as well as the complete response
    - -s is **S**ilent mode
- grep
    - -v to in**V**ert the search - that is, remove it from the search results
    - -i is case **I**nsensitive search


### Easter 7
Checking the cookies.
![Checking the cookies](https://i.imgur.com/YrqubFa.png)


### Easter 8
We change our user agent using curl as,

`curl -A "Mozilla/5.0 (iPhone; CPU iPhone OS 13_1_2 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.1 Mobile/15E148 Safari/604.1" 10.10.9.174 > rich.txt`

Taken from [here](https://developers.whatismybrowser.com/useragents/parse/5390161safari-ios-iphone-webkit).

```
┌──(kali㉿kali)-[/tmp]
└─$ cat rich.txt | grep "Easter 8"
                        <h4>You are Rich! Subscribe to THM server ^^ now. Oh btw, Easter 8: THM{mmmmm}
```

### Easter 9

This one is closely related to 13. Once you press the button, you are taken to an intermediate page. Read the source code.

![Intermediate page source code](https://i.imgur.com/UDKXWzU.png)


### Easter 10
We change the referer to tryhackme.com, without the http:// part.


```
┌──(kali㉿kali)-[~]
└─$ curl -H "referer: tryhackme.com" http://10.10.9.174/free_sub/ 
Nah, there are no voucher here, I'm too poor to buy a new one XD. But i got an egg for you. Easter 10: THM{mmmm} 
```

Recall, `-H` is used to specify the header.


### Easter 11
![main page, dinner menu](https://i.imgur.com/dwGhmdL.png)
Main page, dinner menu. The hint is to select `egg`, but we do not have it as an option. So, we put a request as such:

`curl --data "dinner=egg&submit=submit" 10.10.9.174 | grep -v "Easter 14" | grep -i "easter"`

`--data` is self explanatory. Rest of the flags have been explained above.

### Easter 12
We find a hidden and useless javascript file in the Network tab.
![Hidden useless javascript file](https://i.imgur.com/aw6ld7A.jpg)
Go to console and execute the function.

### Easter 13
Destroy the world to get the flag. Press the big button on the home page. Related to 9. Let the redirect happen.

### Easter 14
Viewing the source code of the main page, we get the following comment:

`<!--Easter 14<img src="data:image/png;base64,iVBORw0KGgo ...`

Remove the comment parts and the html tags, and put it in CyberChef. As usual, magic button works its charm.


### Easter 15: Game 1
This one is a concoction of smart and dumb at the same time. You have to realise that those hashes are just mappings from letters to numbers. Moreover, they are deterministic.

So, how do you get ALL the possible mappings? Its big brain time (for you. I did it already).

PS: Feel free to use python3 again :)
PPS: And what does mapping map to, in python lingo?

### Easter 16: Game 2
This one is very straight forward too, just use curl, and the `-F` flag to specify stuff. It stands for field.

Use the below:

`curl -X POST http://10.10.9.174/game2/ -F "button3=button3"  -F "button2=button2" -F "button1=button1"`

Easy.

### Easter 17
```
	<script>
	function catz(){
        	document.getElementById("nyan").innerHTML = "1000..."
	}
	</script>
    
```

Take the binary and then ... 
`Binary -> Hex -> Text -> Stonks`.

### Easter 18
The YES to say to the eggs one.
`curl -H "egg: YES" 10.10.9.174 | grep "Easter"`

Easy :)

### Easter 19
The thick dark line is an image. Check the source code :)

### Easter 20

`<h3> Hey! I got the easter 20 for you. I leave the credential for you to POST (username:DesKel, password:heIsDumb). Please, I beg you. Don't let him know.</h3>`

`curl --data "username=DesKel&password=heIsDumb" -X POST 10.10.9.174 | grep "Easter 20"`

And ... we are done!
