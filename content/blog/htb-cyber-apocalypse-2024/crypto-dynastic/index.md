---
title: "[Crypto] Dynastic"
date: 2024-03-22T21:00:00+08:00
summary: HTB Cyber apocalypse 2024 - Trithemius cipher
draft: false
tags: [ "ctf", "crypto" ]
---

In this Crypto challenge, we are given 2 files, a python script and a text file, nammed output.text.

### source.py 
```python
from secret import FLAG
from random import randint

def to_identity_map(a):
    return ord(a) - 0x41

def from_identity_map(a):
    return chr(a % 26 + 0x41)

def encrypt(m):
    c = ''
    for i in range(len(m)):
        ch = m[i]
        if not ch.isalpha():
            ech = ch
        else:
            chi = to_identity_map(ch)
            ech = from_identity_map(chi + i)
        c += ech
    return c

with open('output.txt', 'w') as f:
    f.write('Make sure you wrap the decrypted text with the HTB flag format :-]\n')
    f.write(encrypt(FLAG))
```

### output.txt
```
DJF_CTA_SWYH_NPDKK_MBZ_QPHTIGPMZY_KRZSQE?!_ZL_CN_PGLIMCU_YU_KJODME_RYGZXL
```

## Analysis

From the source code, we quickly see that any non-alpha character is not changed, and a simple transformation is applied to alpha character

Each character is converted to it's byte value, then we remove 65 (0x41 in hexa), so A becomes 0, B becomes 1 and so on, we then add an offset to the numeric value, corresponding to the postition of the character in the string, modulus 26, and readd 65 before reencoding the character to ASCII, so first character of a string does not change, second is offswet byu 1 in the alphabet, third by 2 and so on, for instance string 'AAAA' would become 'ABCD'.

We can write a quick script to reverse the operation:


```python
enc = 'DJF_CTA_SWYH_NPDKK_MBZ_QPHTIGPMZY_KRZSQE?!_ZL_CN_PGLIMCU_YU_KJODME_RYGZXL'

def decrypt(enc):
    r = ''
    for i in range(len(enc)):
        ech = enc[i]
        if not ech.isalpha():
            dch = ech
        else:
            dch = chr(((ord(ech) - 0x41 - i) % 26) + 0x41)
        r += dch
    return r

print(decrypt(enc))
```

Running the script give us that flag: `DID_YOU_KNOW_ABOUT_THE_TRITHEMIUS_CIPHER?!_IT_IS_SIMILAR_TO_CAESAR_CIPHER`

This "encryption" algorithm is known as Trithemius cipher.
