---
title: "[rev] Packed away"
date: 2024-03-24T20:00:00+08:00
summary: HTB Cyber apocalypse 2024 - are we talking about a 6 pack?
draft: false
tags: [ "ctf", "rev" ]
---

In this challenge, we are jsut given an executable. Running command strings on it returns an interesting information:

```bash
>> strings packed
...
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 4.22 Copyright (C) 1996-2024 the UPX Team. All Rights Reserved. $
...
```

Executable is packed, let's unpack it.

```
>> upx -d packed -o unpacked                                    [21:08:09]
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2023
UPX 4.1.0       Markus Oberhumer, Laszlo Molnar & John Reiser    Aug 8th 2023

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     22867 <-      8848   38.69%   linux/amd64   unpacked

Unpacked 1 file.
```

A second call to strings give the flag:

```
strings unpacked
...
HTB{unp4ck3d_th3_s3cr3t_0f_th3_p455w0rd}
...
```
