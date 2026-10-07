# Yer a Wizard

## 1. Intel

Scan the target with nmap:

`
nmap -sC -sV <ip>
`

**Output:**

```

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.8.0.202
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel`

```

## 2. Connect

Since we can see a ftp service, we can connect to it:

`
ftp {ip}
`

## 3. Explore

We see two things:

    - a .hidden file
    - a directory named "..."

We take a look at the file:

`cat .hidden`

**Output:**

`I swear that my password is Il0veTheMalefoys, trust me, I'm Hagrid, I never lie!
81 bytes transferred`

We `cd` in the dir and find another file:

`cat .reallyHidden`

**Output:**

`FINE! My password is IAlreadySaidTooMuch `



## 4. Login

Since we potentially have an username and password , we try to log in the machine with ssh

`ssh hagrid@{ip}` 

`password : IAlreadySaidTooMuch`


Now that we are in the machine we find a `user.txt` file with an encrypted content

## 5. Decrypt

The file in uncrypted in base64 twice , a simple decrypt gives us the flag.