---
title: "[Crypto] Multipage recycling"
date: 2023-03-24T23:00:00+08:00
summary: HTB Cyber apocalypse 2023 - ECB's property
draft: false
tags: [ "ctf", "crypto" ]
---

In this Crypto challenge, we are given 2 files, a python script and a text file, nammed output.text.

### source.py 
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import random, os

FLAG = b'HTB{??????????????????????}'


class CAES:

    def __init__(self):
        self.key = os.urandom(16)
        self.cipher = AES.new(self.key, AES.MODE_ECB)

    def blockify(self, message, size):
        return [message[i:i + size] for i in range(0, len(message), size)]

    def xor(self, a, b):
        return b''.join([bytes([_a ^ _b]) for _a, _b in zip(a, b)])

    def encrypt(self, message):
        iv = os.urandom(16)

        ciphertext = b''
        plaintext = iv

        blocks = self.blockify(message, 16)
        for block in blocks:
            ct = self.cipher.encrypt(plaintext)
            encrypted_block = self.xor(block, ct)
            ciphertext += encrypted_block
            plaintext = encrypted_block

        return ciphertext

    def leak(self, blocks):
        r = random.randint(0, len(blocks) - 2)
        leak = [self.cipher.encrypt(blocks[i]).hex() for i in [r, r + 1]]
        return r, leak


def main():
    aes = CAES()
    message = pad(FLAG * 4, 16)

    ciphertext = aes.encrypt(message)
    ciphertext_blocks = aes.blockify(ciphertext, 16)

    r, leak = aes.leak(ciphertext_blocks)

    with open('output.txt', 'w') as f:
        f.write(f'ct = {ciphertext.hex()}\nr = {r}\nphrases = {leak}\n')


if __name__ == "__main__":
    main()

```

### output.txt
```
ct = bc9bc77a809b7f618522d36ef7765e1cad359eef39f0eaa5dc5d85f3ab249e788c9bc36e11d72eee281d1a645027bd96a363c0e24efc6b5caa552b2df4979a5ad41e405576d415a5272ba730e27c593eb2c725031a52b7aa92df4c4e26f116c631630b5d23f11775804a688e5e4d5624
r = 3
phrases = ['8b6973611d8b62941043f85cd1483244', 'cf8f71416111f1e8cdee791151c222ad']
```

## Analysis

We see that the algorithm used for encryption is AES mode ECB, encryption will happen by 16 bytes block, and we also know that, if we run the script once, with the same key, the same block of byte will generate the same encrypted output.

First step, the flag is concatenated with itself several times, and padded to a multiple of 16 bytes. For the encryption, that message will be split in block of 16 bytes.

For analysing what's happening, let's assume the flag is `HTB{absdefghujklmnopqrstuv}`, the message, split by block of 16 bytes will be:

```
HTB{absdefghujkl
mnopqrstuv}HTB{a
bsdefghujklmnopq
rstuv}HTB{absdef
ghujklmnopqrstuv
}HTB{absdefghujk
lmnopqrstuv}\x04\x04\x04\x04
```
let's name those block, block1, block 2, ...

```
HTB{absdefghujkl <-- block1
mnopqrstuv}HTB{a <-- block2
bsdefghujklmnopq <-- block3
rstuv}HTB{absdef <-- block4
ghujklmnopqrstuv <-- block5
}HTB{absdefghujk <-- block6
lmnopqrstuv}\x04\x04\x04\x04 <-- block7
```
For the encryption, we start with an 16 bytes IV, and by looking at the algorithm we find the following:

```
encrypted_block1 :  block1 XOR aes_encrypt_ecb(IV)
encrypted_block2 :  block2 XOR aes_encrypt_ecb(encrypted_block1)
encrypted_block3 :  block3 XOR aes_encrypt_ecb(encrypted_block2)
encrypted_block4 :  block4 XOR aes_encrypt_ecb(encrypted_block3)
encrypted_block5 :  block5 XOR aes_encrypt_ecb(encrypted_block4)
encrypted_block6 :  block6 XOR aes_encrypt_ecb(encrypted_block5)
encrypted_block7 :  block7 XOR aes_encrypt_ecb(encrypted_block6)
```

Now if we look at the `leak` function, we see that it will take 2 consecutives 16 blocks of the encrypted message starting at a random index, and return us their encrypted value, with the same key as the message encryption.

Let's pick r = 3, the leack function will return us:

```
aes_encrypt_ecb(encrypted_block3) <-- leak3
aes_encrypt_ecb(encrypted_block4) <-- leak4
```

If we look at both what we know of the encrypted message and the leaked blocks, we have
```
encrypted_block4 :  block4 XOR aes_encrypt_ecb(encrypted_block3)
encrypted_block5 :  block5 XOR aes_encrypt_ecb(encrypted_block4)
```
where `encrypted_block4` and `encrypted_block5` are known (from the cipher test)

and we also have access to the value of `aes_encrypt_ecb(encrypted_block3)` and `aes_encrypt_ecb(encrypted_block4)` (nammed leak3 and leak4 for convenience).

with some substition, we have:

```
encrypted_block4 = block4 XOR leak3

and

encrypted_block5 = block5 XOR leak4
```
Properties of XOR allow us to deduct that:

```
block4 = encrypted_block4 XOR leak3
block5 = encrypted_block5 XOR leak4
```

## Solution code:

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import random, os

class CAES:

    def __init__(self):
        self.key = os.urandom(16)
        self.cipher = AES.new(self.key, AES.MODE_ECB)

    def blockify(self, message, size):
        return [message[i:i + size] for i in range(0, len(message), size)]

    def xor(self, a, b):
        return b''.join([bytes([_a ^ _b]) for _a, _b in zip(a, b)])

    def encrypt(self, message):
        iv = os.urandom(16)

        ciphertext = b''
        plaintext = iv

        blocks = self.blockify(message, 16)
        for block in blocks:
            ct = self.cipher.encrypt(plaintext)
            encrypted_block = self.xor(block, ct)
            ciphertext += encrypted_block
            plaintext = encrypted_block

        return ciphertext

    def leak(self, blocks):
        r = random.randint(0, len(blocks) - 2)
        leak = [self.cipher.encrypt(blocks[i]).hex() for i in [r, r + 1]]
        return r, leak

def byte_xor(ba1, ba2):
    return bytes([_a ^ _b for _a, _b in zip(ba1, ba2)])

def main():
    aes = CAES()
    ct = 'bc9bc77a809b7f618522d36ef7765e1cad359eef39f0eaa5dc5d85f3ab249e788c9bc36e11d72eee281d1a645027bd96a363c0e24efc6b5caa552b2df4979a5ad41e405576d415a5272ba730e27c593eb2c725031a52b7aa92df4c4e26f116c631630b5d23f11775804a688e5e4d5624'
    ciphertext = bytes.fromhex(ct)
    ciphertext_blocks = aes.blockify(ciphertext, 16)

    print(ciphertext_blocks)
    print(byte_xor(ciphertext_blocks[4], bytes.fromhex('8b6973611d8b62941043f85cd1483244')))
    print(byte_xor(ciphertext_blocks[5], bytes.fromhex('cf8f71416111f1e8cdee791151c222ad')))

if __name__ == "__main__":
    main()
```

From the execution, we have both halves of the flag:

```
» python solve.py
b'_w34k_w17h_l34kz'
b'}HTB{CFB_15_w34k'
```

Let's concatenate to have the flag: `HTB{CFB_15_w34k_w34k_w17h_l34kz}`
