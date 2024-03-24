---
title: "[misc] Stop Drop and Roll"
date: 2024-03-24T20:00:00+08:00
summary: HTB Cyber apocalypse 2024 - text interface gaming
draft: false
tags: [ "ctf", "misc" ]
---

This challenge is exposing an IP and a port, connecting to it throught netcat gives the following:

```
===== THE FRAY: THE VIDEO GAME =====
Welcome!
This video game is very simple
You are a competitor in The Fray, running the GAUNTLET
I will give you one of three scenarios: GORGE, PHREAK or FIRE
You have to tell me if I need to STOP, DROP or ROLL
If I tell you there's a GORGE, you send back STOP
If I tell you there's a PHREAK, you send back DROP
If I tell you there's a FIRE, you send back ROLL
Sometimes, I will send back more than one! Like this:
GORGE, FIRE, PHREAK
In this case, you need to send back STOP-ROLL-DROP!
Are you ready? (y/n) y
Ok then! Let's go!
```

The game then start to give instruction, to which we need to reply according to the previous rules, if we see GORGE, we need to reply STOP and so on.

One way to finish this challenge would be to reply manually, but the game stop at the first input error, and is quite slow.

Let's wrtie a small script to play on our behalf:

```python
from pwn import *

p = remote('94.237.60.100', 32581)
print(p.recvregex( b"\\)" ) )

p.sendline(b'y')
print(p.recvline())

count = 0
while True:
    print(f"Iteration {count}")
    count += 1
    instruction = p.recvlineS()
    print(f"Receive instructions: {instruction}")
    instruction = instruction[:len(instruction)-1]
    actions = []
    for one in instruction.split(', '):
        one = one.strip()
        if one == 'GORGE' :
            actions.append(b'STOP')
        if one == 'PHREAK' :
            actions.append(b'DROP')
        if one == 'FIRE' :
            actions.append(b'ROLL')
    if len(actions) > 0:
        print("One action or more")
        print(p.recvregex(b"\\?")) #What do you do ?
        print("reply with", actions)
        p.sendline(b"-".join(actions))
```

After running this script:

```
One action or more
b'What do you do?'
reply with [b'STOP', b'STOP', b'DROP', b'STOP']
Iteration 499
Receive instructions:  GORGE, GORGE, FIRE

One action or more
b'What do you do?'
reply with [b'STOP', b'STOP', b'ROLL']
Iteration 500
Receive instructions:  Fantastic work! The flag is HTB{1_wiLl_sT0p_dR0p_4nD_r0Ll_mY_w4Y_oUt!}
```
