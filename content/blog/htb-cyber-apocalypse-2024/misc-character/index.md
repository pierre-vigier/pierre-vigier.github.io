---
title: "[misc] Character"
date: 2024-03-24T20:00:00+08:00
summary: HTB Cyber apocalypse 2024 - Charater, a game of patience
draft: false
tags: [ "ctf", "misc" ]
---

In this challenge, we are given an IP address and a port, nothing else. Connecting to the IP/port with netcat, we get the following:

```
> nc 94.237.48.205 49794

Which character (index) of the flag do you want? Enter an index: 
0
Character at Index 0: H
```

We can get the flag one character at a time manually, to speed things up, a simple bash loop:

```
>> for i in {0..200}; do echo $i;done | nc 94.237.48.205 49794
Which character (index) of the flag do you want? Enter an index: Character at Index 0: H
Which character (index) of the flag do you want? Enter an index: Character at Index 1: T
Which character (index) of the flag do you want? Enter an index: Character at Index 2: B
Which character (index) of the flag do you want? Enter an index: Character at Index 3: {
Which character (index) of the flag do you want? Enter an index: Character at Index 4: t
Which character (index) of the flag do you want? Enter an index: Character at Index 5: H
Which character (index) of the flag do you want? Enter an index: Character at Index 6: 1
Which character (index) of the flag do you want? Enter an index: Character at Index 7: 5
Which character (index) of the flag do you want? Enter an index: Character at Index 8: _
Which character (index) of the flag do you want? Enter an index: Character at Index 9: 1
Which character (index) of the flag do you want? Enter an index: Character at Index 10: s
Which character (index) of the flag do you want? Enter an index: Character at Index 11: _
Which character (index) of the flag do you want? Enter an index: Character at Index 12: 4
Which character (index) of the flag do you want? Enter an index: Character at Index 13: _
Which character (index) of the flag do you want? Enter an index: Character at Index 14: r
Which character (index) of the flag do you want? Enter an index: Character at Index 15: 3
Which character (index) of the flag do you want? Enter an index: Character at Index 16: a
Which character (index) of the flag do you want? Enter an index: Character at Index 17: L
Which character (index) of the flag do you want? Enter an index: Character at Index 18: l
Which character (index) of the flag do you want? Enter an index: Character at Index 19: y
Which character (index) of the flag do you want? Enter an index: Character at Index 20: _
Which character (index) of the flag do you want? Enter an index: Character at Index 21: l
Which character (index) of the flag do you want? Enter an index: Character at Index 22: 0
Which character (index) of the flag do you want? Enter an index: Character at Index 23: n
Which character (index) of the flag do you want? Enter an index: Character at Index 24: G
Which character (index) of the flag do you want? Enter an index: Character at Index 25: _
Which character (index) of the flag do you want? Enter an index: Character at Index 26: f
Which character (index) of the flag do you want? Enter an index: Character at Index 27: L
Which character (index) of the flag do you want? Enter an index: Character at Index 28: 4
Which character (index) of the flag do you want? Enter an index: Character at Index 29: g
Which character (index) of the flag do you want? Enter an index: Character at Index 30: _
Which character (index) of the flag do you want? Enter an index: Character at Index 31: i
Which character (index) of the flag do you want? Enter an index: Character at Index 32: _
Which character (index) of the flag do you want? Enter an index: Character at Index 33: h
Which character (index) of the flag do you want? Enter an index: Character at Index 34: 0
Which character (index) of the flag do you want? Enter an index: Character at Index 35: p
Which character (index) of the flag do you want? Enter an index: Character at Index 36: 3
Which character (index) of the flag do you want? Enter an index: Character at Index 37: _
Which character (index) of the flag do you want? Enter an index: Character at Index 38: f
Which character (index) of the flag do you want? Enter an index: Character at Index 39: 0
Which character (index) of the flag do you want? Enter an index: Character at Index 40: r
Which character (index) of the flag do you want? Enter an index: Character at Index 41: _
Which character (index) of the flag do you want? Enter an index: Character at Index 42: y
Which character (index) of the flag do you want? Enter an index: Character at Index 43: 0
Which character (index) of the flag do you want? Enter an index: Character at Index 44: U
Which character (index) of the flag do you want? Enter an index: Character at Index 45: r
Which character (index) of the flag do you want? Enter an index: Character at Index 46: _
Which character (index) of the flag do you want? Enter an index: Character at Index 47: s
Which character (index) of the flag do you want? Enter an index: Character at Index 48: 4
Which character (index) of the flag do you want? Enter an index: Character at Index 49: k
Which character (index) of the flag do you want? Enter an index: Character at Index 50: 3
Which character (index) of the flag do you want? Enter an index: Character at Index 51: _
Which character (index) of the flag do you want? Enter an index: Character at Index 52: t
Which character (index) of the flag do you want? Enter an index: Character at Index 53: H
Which character (index) of the flag do you want? Enter an index: Character at Index 54: 4
Which character (index) of the flag do you want? Enter an index: Character at Index 55: t
Which character (index) of the flag do you want? Enter an index: Character at Index 56: _
Which character (index) of the flag do you want? Enter an index: Character at Index 57: y
Which character (index) of the flag do you want? Enter an index: Character at Index 58: 0
Which character (index) of the flag do you want? Enter an index: Character at Index 59: U
Which character (index) of the flag do you want? Enter an index: Character at Index 60: _
Which character (index) of the flag do you want? Enter an index: Character at Index 61: s
Which character (index) of the flag do you want? Enter an index: Character at Index 62: C
Which character (index) of the flag do you want? Enter an index: Character at Index 63: r
Which character (index) of the flag do you want? Enter an index: Character at Index 64: 1
Which character (index) of the flag do you want? Enter an index: Character at Index 65: p
Which character (index) of the flag do you want? Enter an index: Character at Index 66: T
Which character (index) of the flag do you want? Enter an index: Character at Index 67: E
Which character (index) of the flag do you want? Enter an index: Character at Index 68: d
Which character (index) of the flag do you want? Enter an index: Character at Index 69: _
Which character (index) of the flag do you want? Enter an index: Character at Index 70: t
Which character (index) of the flag do you want? Enter an index: Character at Index 71: H
Which character (index) of the flag do you want? Enter an index: Character at Index 72: 1
Which character (index) of the flag do you want? Enter an index: Character at Index 73: s
Which character (index) of the flag do you want? Enter an index: Character at Index 74: _
Which character (index) of the flag do you want? Enter an index: Character at Index 75: o
Which character (index) of the flag do you want? Enter an index: Character at Index 76: R
Which character (index) of the flag do you want? Enter an index: Character at Index 77: _
Which character (index) of the flag do you want? Enter an index: Character at Index 78: e
Which character (index) of the flag do you want? Enter an index: Character at Index 79: l
Which character (index) of the flag do you want? Enter an index: Character at Index 80: s
Which character (index) of the flag do you want? Enter an index: Character at Index 81: 3
Which character (index) of the flag do you want? Enter an index: Character at Index 82: _
Which character (index) of the flag do you want? Enter an index: Character at Index 83: i
Which character (index) of the flag do you want? Enter an index: Character at Index 84: T
Which character (index) of the flag do you want? Enter an index: Character at Index 85: _
Which character (index) of the flag do you want? Enter an index: Character at Index 86: t
Which character (index) of the flag do you want? Enter an index: Character at Index 87: 0
Which character (index) of the flag do you want? Enter an index: Character at Index 88: o
Which character (index) of the flag do you want? Enter an index: Character at Index 89: K
Which character (index) of the flag do you want? Enter an index: Character at Index 90: _
Which character (index) of the flag do you want? Enter an index: Character at Index 91: q
Which character (index) of the flag do you want? Enter an index: Character at Index 92: U
Which character (index) of the flag do you want? Enter an index: Character at Index 93: 1
Which character (index) of the flag do you want? Enter an index: Character at Index 94: t
Which character (index) of the flag do you want? Enter an index: Character at Index 95: 3
Which character (index) of the flag do you want? Enter an index: Character at Index 96: _
Which character (index) of the flag do you want? Enter an index: Character at Index 97: l
Which character (index) of the flag do you want? Enter an index: Character at Index 98: 0
Which character (index) of the flag do you want? Enter an index: Character at Index 99: n
Which character (index) of the flag do you want? Enter an index: Character at Index 100: g
Which character (index) of the flag do you want? Enter an index: Character at Index 101: !
Which character (index) of the flag do you want? Enter an index: Character at Index 102: !
Which character (index) of the flag do you want? Enter an index: Character at Index 103: }
```

And like that, we get the flag:
HTB{tH15_1s_4_r3aLly_l0nG_fL4g_i_h0p3_f0r_y0Ur_s4k3_tH4t_y0U_sCr1pTEd_tH1s_oR_els3_iT_t0oK_qU1t3_l0ng!!}

