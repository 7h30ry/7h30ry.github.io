---
title: ACDF-Cyberlympics 2023
date: 2023-9-24 00:10:51 +0100
categories: [ACDF, CTF]
tags: [web, Jail Break, rev]
math: true
mermaid: true
media_subpath: /assets/posts/2023-9-24-ACDF-Cyberlympics
image:
  path: preview.jpeg
---

## Binary Exploitation
- Flag Bank
  
## Forensic
- EdenZero
- SabekIntro
- Sabek01
  
## Miscellaneous
-  Disc0rd
-  Jail Break
-  Jail Break 2
-  Jail Break 3
  
## OSINT
- Mr. R0b0t
- Mr. R0b0t 2

## Reverse Engineering
- 34sy-r3v
- CodeX

## WEB
- Demon Slayer


# Flag Bank

![Flagbank](https://github.com/7h30ry/writeups/assets/51336409/63a07bba-ca25-4bb0-80db-10b26f170f8f)

This was basically an easy one.
After runing the binary file i got this
```
🏦 Welcome to the Flag Bank! To purchase a Flag costs $20000, but your current balance is $10000. Purchase a Test Flag at $3000
[1] Purchase Flag - $20000
[2] Purchase Test Flag - $3000
[3] View current balance
```
Since i know that there $10000 in balance and i need $20000 to purchase the 
I did to pick the secode option, i was then prompted with
```
Number of Test Flags do you need?
```
so i thought if i endered a positive number let say 3, money will be deducted so i decided to input a negative number -5 insted
and my money was increased, so i was able to purcase the flag

![Flag-bank](https://github.com/7h30ry/writeups/assets/51336409/260d7ca9-8313-43d3-968b-9bd521ef0601)


# EdenZero

![Screenshot from 2023-09-24 17-48-39](https://github.com/7h30ry/writeups/assets/51336409/c8ac2a83-d61e-43e0-bcdf-a63fa3111678)

We were given a zip file and Zip Password: Cyberlympics2023

Anfter unziping the file i got a file named flagishere.jpg but it wasn't actually a jpg it was a pdf file

![Screenshot from 2023-09-24 17-51-14](https://github.com/7h30ry/writeups/assets/51336409/6e097db3-3602-424b-acf8-73e401728b70)

so i did ```mv flagishere.jpg flagishere.pdf``` and got a pdf file

next thing i did was 
```
exiftool flagishere.pdf
```
and i got part of the flag

![Screenshot from 2023-09-24 17-54-07](https://github.com/7h30ry/writeups/assets/51336409/feaef4ce-3e02-4533-b0bd-516e4b5ea09b)

Next i open up the pdf file

![Screenshot from 2023-09-24 17-56-20](https://github.com/7h30ry/writeups/assets/51336409/0a5cf3a0-e583-4810-80b9-60f8d8ff7542)

![Screenshot from 2023-09-24 17-56-59](https://github.com/7h30ry/writeups/assets/51336409/b83cecf8-8c57-4033-a3b5-13f8642e8f7c)

There was nothing like a flag inside, so probably it's there and i cam't see it, so i highlted the whole page and copied everything
and paste it some where else and got a cipher ```D9_Hd0c==0D_>bE`>bdN```

![Screenshot from 2023-09-24 17-59-09](https://github.com/7h30ry/writeups/assets/51336409/c78991b7-7db5-49e9-8f99-4d21f294aa65)

So decrypting it using ROT47 i got the secode half of the flag

![Screenshot from 2023-09-24 18-00-36](https://github.com/7h30ry/writeups/assets/51336409/18c84dda-a724-4b30-9036-a2ffc6db00ff)
```FLAG: acdfCTF{c0mm3nt_buddy_sh0w5_4ll_s0m3t1m35}```


# SabekIntro
![Sabek-Intro](https://github.com/7h30ry/writeups/assets/51336409/4779049e-4ccc-404e-9051-8d25be76343f)

We were just asked ```How Many Packets were captured```

![Sabek-intro](https://github.com/7h30ry/writeups/assets/51336409/beda1a1a-fb54-484f-8a3a-52551cba83d8)

Opening up the pcap file with wireshark, at the botttom right we see the number of packets
```FLAG: acdfCTF{3322}```

# Sabek01

![Screenshot from 2023-09-24 18-07-42](https://github.com/7h30ry/writeups/assets/51336409/4f879af4-9719-411a-8541-d24d3230359d)

For this using the same pcap file
At the top bar in wireshark go to

```
View > Time Display Format > Date and Time of Day
```
once the seetings have been changed you will see the normal time format in the wireshark, juat copy the time for the first packet
whcih is the one at the very top, and the packet at the very last.. Join it and that is the flag

# Disc0rd

```
Here is your discord flag: Welcοme tｏ ｏｕr Ｄіｓｃord server! Ｈｅrｅ＇s ａ lіｔｔｌe ｓeｃrｅｔ ｊｕｓt ｆor ｙｏｕ: tｈerｅｉs ａ hⅰddｅｎ flａｇ ｈiｄｄｅnwｉｔhіｎ ｔhⅰs mｅｓsagｅ. Ｌeｔ'skeep it between ourselves.
```
Using unicode decode 

```Flag: acdfCTF{k1sm3t_4nd_b3rry_1s_my_1d0l}```

# Jail Break

![Screenshot from 2023-09-24 18-19-09](https://github.com/7h30ry/writeups/assets/51336409/dcae4e55-ece2-4a8f-a27f-839c75afaaa4)

Payload used ```__import__('os').system('bash')```

![Screenshot from 2023-09-24 18-22-34](https://github.com/7h30ry/writeups/assets/51336409/a8fc4065-6f3f-498a-857d-812eeb779868)
```Flag: acdfCTF{Cyb3rlymp1cs_w4rmup_pyj41l_v3ry_345y_r1ght?}```

# Jail Break 2

![Screenshot from 2023-09-24 18-24-28](https://github.com/7h30ry/writeups/assets/51336409/bb4b34b5-6529-4c7b-a2bf-48f646b9fde5)

Payload used 

```
damn= ().__class__.__base__.__subclasses__()[59]()

print(damn._module.__builtins__['__import__']('os').popen("cat flag.txt").read())
```
![Screenshot from 2023-09-24 18-31-45](https://github.com/7h30ry/writeups/assets/51336409/c1c6db79-d82b-4444-b4e2-e3cacb043f78)
```Flag: acdfCTF{35c4p3_pl4n_1337_w0rk3d_y35!!}```

# Jail Break 3

![Screenshot from 2023-09-24 18-33-09](https://github.com/7h30ry/writeups/assets/51336409/3d90d702-9933-44b0-b710-dc533caabd61)

Payload gotten from 

[here](https://ctf.rip/backdoor16-worstpwnever/)
```Flag: acdfCTF{M45t3r_0f_j41l_br34k3r}```



# Mr. R0b0t

![Mr Robot](https://github.com/7h30ry/writeups/assets/51336409/d365c8c3-10d5-45a9-bdba-230fdf365857)

All i did was just search online for the answer

![Mr Robot](https://github.com/7h30ry/writeups/assets/51336409/e997428f-baa7-47c6-aef2-f81e64ab5506)

```
Flag:acdFCTF{ChurchAvenueSubwayStation}
```


# Mr. R0b0t 2

![Mrobot2](https://github.com/7h30ry/writeups/assets/51336409/c7bfb875-26c6-458d-b4a7-bf69f3a9bbe1)

Doing some reverse image search

![Mr-robot2](https://github.com/7h30ry/writeups/assets/51336409/5d572aca-8eb1-4486-b41d-2c6a964605a5)


![Mrrobot2](https://github.com/7h30ry/writeups/assets/51336409/9155d744-d019-40a8-8291-1cb67cbe289b)

```Flag: acdfCTF{Coney Island, Brooklyn, New York}```



# 34sy-r3v

![Screenshot from 2023-09-24 17-32-21](https://github.com/7h30ry/writeups/assets/51336409/af14580d-a40b-4887-ac56-df3d94b48daf)

I used the strings and grep command to get the flag

```
strings easyrev | grep CTF
```
![Screenshot from 2023-09-24 18-44-00](https://github.com/7h30ry/writeups/assets/51336409/c3505018-772e-4ac8-b6a2-623da0fd6326)

```Flag: acdfCTF{5tr1ngs_b1n4ry_t0_g3t_fl4g}```


# CodeX

![Codex](https://github.com/7h30ry/writeups/assets/51336409/e17686ea-e3aa-4013-81b3-90158020a9e0)

```strings recipe```

![Screenshot from 2023-09-24 19-44-13](https://github.com/7h30ry/writeups/assets/51336409/2fc8b8af-4ff2-4bc9-9998-ec9400936848)


So i juste decided to right that out backwards
```Flag: acdCFTF{Th3_p3rf3ct_r3c1p3_for_3t3rn1ty_l1f3}```


# Demon slayer

![Demon-Slayer](https://github.com/7h30ry/writeups/assets/51336409/389f5cf1-fc5a-418b-90a7-9822c3afb7c1)

This challenge is based on Command injection but we need to bypass the black list

![Demon slayer](https://github.com/7h30ry/writeups/assets/51336409/05468899-dc15-4f5a-943a-5ce2eb355d33)

Firstly ecode your command in base64


![Screenshot from 2023-09-24 19-52-34](https://github.com/7h30ry/writeups/assets/51336409/5a305f02-158a-40ae-8e71-37031dc8eba9)

```Flag: acdfCTF{bl4ckl15t3d_c0mmand_3xpung3r5} ```
