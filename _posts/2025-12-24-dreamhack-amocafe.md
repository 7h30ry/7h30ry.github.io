---
title: amocafe @ Dreamhack
date: 2025-12-24 09:56:36 +0100
categories: [dreamhack, wargames]
tags: [web, flask, python]
math: true
mermaid: true
media_subpath: /assets/posts/dreamhack
image:
  path: preview.png
---



# CTF Challenge Writeup: Amocafe Menu Decoder

### Source Code
```python
#!/usr/bin/env python3
from flask import Flask, request, render_template

app = Flask(__name__)

try:
    FLAG = open("./flag.txt", "r").read()       # flag is here!
except:
    FLAG = "[**FLAG**]"

@app.route('/', methods=['GET', 'POST'])
def index():
    menu_str = ''
    org = FLAG[10:29]
    org = int(org)
    st = ['' for i in range(16)]

    for i in range (0, 16):
        res = (org >> (4 * i)) & 0xf
        if 0 < res < 12:
            if ~res & 0xf == 0x4:
                st[16-i-1] = '_'
            else:
                st[16-i-1] = str(res)
        else:
            st[16-i-1] = format(res, 'x')
    menu_str = menu_str.join(st)

    # POST
    if request.method == "POST":
        input_str =  request.form.get("menu_input", "")
        if input_str == str(org):
            return render_template('index.html', menu=menu_str, flag=FLAG)
        return render_template('index.html', menu=menu_str, flag='try again...')
    # GET
    return render_template('index.html', menu=menu_str)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)

```

#### Challenge Overview
This Flask web application presents a CTF challenge where you need to reverse-engineer an obfuscated number from a displayed "menu" string to capture the flag.

#### Challenge Analysis
#### The Code Flow
**Flag Extraction:** The app reads a flag from flag.txt and extracts characters 10-29 (20 characters total)
**Conversion:** These 20 characters are converted to an integer org
**Obfuscation:** The integer is split into 16 nibbles (4-bit chunks) and transformed into a display string
**Goal:** Submit the correct original number org to get the flag

#### The Encoding Logic
The code processes each nibble (4-bit value, 0-15) of the number:

```python
for i in range(0, 16):
    res = (org >> (4 * i)) & 0xf  # Extract nibble i
    if 0 < res < 12:
        if ~res & 0xf == 0x4:
            st[16-i-1] = '_'
        else:
            st[16-i-1] = str(res)
    else:
        st[16-i-1] = format(res, 'x')
```

**Key observations:**
```i=0``` extracts the least significant nibble (bits 0-3) and places it at st[15] (rightmost)
```i=15``` extracts the most significant nibble (bits 60-63) and places it at st[0] (leftmost)
The display string reads left-to-right as most-to-least significant

**Encoding Rules**
Nibble Value       Condition	    Output
0	res == 0	   format(0, 'x') → '0'
1-9	0 < res < 12	str(res) → '1' to '9'
10	0 < res < 12	str(10) → '10' (2 chars!)
11	~11 & 0xf == 0x4 ✓	'_'
12-15	res >= 12	format(res, 'x') → 'c', 'd', 'e', 'f'

**Critical insight:** The underscore _ represents nibble value 11 because:
Binary 11 = 1011
Bitwise NOT of 1011 = 0100
0100 & 1111 = 0100 = 4 ✓

## Solve Script
```python
#!/usr/bin/env python3

def reverse_menu_to_org(menu_str):
    """
    Reverse the menu string back to the original number.
    
    The encoding logic:
    - res = 0: format(0, 'x') -> '0'
    - res = 1-9: str(res) -> '1'-'9'
    - res = 10: str(10) -> '10' (2 characters!)
    - res = 11: if ~11 & 0xf == 0x4 -> '_'
    - res = 12-15: format(res, 'x') -> 'c', 'd', 'e', 'f'
    """
    
    # Parse the menu string into nibbles
    nibbles = []
    i = 0
    while i < len(menu_str):
        char = menu_str[i]
        
        # Check if this might be '10' (two characters)
        if char == '1' and i + 1 < len(menu_str) and menu_str[i + 1] == '0':
            nibbles.append(10)
            i += 2
        elif char == '_':
            nibbles.append(11)
            i += 1
        elif char in '0123456789':
            nibbles.append(int(char))
            i += 1
        elif char in 'abcdef':
            nibbles.append(int(char, 16))
            i += 1
        else:
            raise ValueError(f"Unexpected character: {char}")
    
    print(f"Parsed nibbles: {nibbles}")
    print(f"Number of nibbles: {len(nibbles)}")
    
    # The code does st[16-i-1] for i in range(0, 16)
    # When i=0: st[15] gets (org >> 0) & 0xf (least significant)
    # When i=15: st[0] gets (org >> 60) & 0xf (most significant)
    # So st[0] is MSB, st[15] is LSB
    
    # Reconstruct org from nibbles
    org = 0
    for idx, nibble in enumerate(nibbles):
        # idx=0 is leftmost (most significant)
        # We need to place it at the correct bit position
        shift = 4 * (len(nibbles) - 1 - idx)
        org |= (nibble << shift)
    
    return org

def verify_solution(org, expected_menu):
    """
    Verify our solution by running the encoding logic forward.
    """
    st = ['' for i in range(16)]
    
    for i in range(0, 16):
        res = (org >> (4 * i)) & 0xf
        if 0 < res < 12:
            if ~res & 0xf == 0x4:
                st[16-i-1] = '_'
            else:
                st[16-i-1] = str(res)
        else:
            st[16-i-1] = format(res, 'x')
    
    menu_str = ''.join(st)
    print(f"Verification - Generated menu: {menu_str}")
    print(f"Verification - Expected menu:  {expected_menu}")
    print(f"Match: {menu_str == expected_menu}")
    
    return menu_str == expected_menu

# The menu from the challenge
menu = "1_c_3_c_0__ff_3e"

print("="*50)
print("CTF Challenge Solver")
print("="*50)
print(f"Input menu: {menu}")
print()

# Solve it
org = reverse_menu_to_org(menu)
print(f"\nOriginal number (org): {org}")
print()

# Verify
print("Verifying solution...")
if verify_solution(org, menu):
    print("\n✓ Solution verified!")
    print(f"\nSubmit this value: {org}")
else:
    print("\n✗ Solution doesn't match - need to debug")
    print("\nLet me try alternative interpretation...")
    
    # Maybe the issue is with how we count positions
    # Let's try treating it as exactly 16 nibbles
    if len(menu) == 16:
        print("\nTrying direct 16-character interpretation:")
        nibbles = []
        for char in menu:
            if char == '_':
                nibbles.append(11)
            elif char in '0123456789':
                nibbles.append(int(char))
            elif char in 'abcdef':
                nibbles.append(int(char, 16))
        
        org2 = 0
        for idx, nibble in enumerate(nibbles):
            shift = 4 * (15 - idx)
            org2 |= (nibble << shift)
        
        print(f"Alternative org: {org2}")
        verify_solution(org2, menu)
```