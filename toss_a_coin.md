# Toss a coin

## Intel

We scan the ip with nmap :

```
nmap -sC -sV 10.10.0.23
```

**Output**

```
nmap -sC -sV 10.10.0.23                      
Starting Nmap 7.991 ( https://nmap.org ) at 2026-09-17 08:01 +0000
Stats: 0:00:26 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 17.91% done; ETC: 08:03 (0:01:57 remaining)
Stats: 0:00:43 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 52.19% done; ETC: 08:02 (0:00:38 remaining)
Stats: 0:00:56 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 08:02 (0:00:06 remaining)
Stats: 0:00:58 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 96.19% done; ETC: 08:02 (0:00:00 remaining)
Stats: 0:01:02 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.31% done; ETC: 08:02 (0:00:00 remaining)
Nmap scan report for 10.10.0.23
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.68 ((Debian))
|_http-title: Toss a coin to your witcher
|_http-server-header: Apache/2.4.68 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

We can also use `gobuster` to check dir in the url

```
gobuster dir -u http://10.10.0.23/ -w ~/text -b 404
```

**Output:**

```

    ~  gobuster dir -u http://10.10.0.23/ -w ~/medium.txt -b 404                ✔  39s  
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.0.23/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/asakz/medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
img                  (Status: 301) [Size: 346] [--> http://10.10.0.23/img/]
t                    (Status: 301) [Size: 344] [--> http://10.10.0.23/t/]
server-status        (Status: 403) [Size: 315]
Progress: 207641 / 207641 (100.00%)
===============================================================
Finished
===============================================================
```

url/t leads us to a page with two sentences on it. 

```
"Do you know the lyrics? this one is pretty famous!"
```

Probably references the song the challengs is named after.

The /t folder is a bit odd

Running the dir bruteforce again, yet adding the /o at the end of the url, we get this

**Output:**

```
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.0.23/t/o/FUZZ
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/asakz/medium.txt
[+] Excluded Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in fuzzing mode
===============================================================
[Status=301] [Length=348] [Word=s] http://10.10.0.23/t/o/s [--> http://10.10.0.23/t/o/s/]
```

Since the name of the song is `Toss a coin to your witcher` , we can assume that the url spells the lyrics of the song.

After multiple bruteforces , we end up with this url : 

```
http://10.10.0.3/t/o/s/s/_/a/_/c/o/i/n/_/t/o/_/y/o/u/r/_/w/i/t/c/h/e/r/_/o/h/_/v/a/l/l/e/y/_/o/f/_/p/l/e/n/t/y/
```

It sends us onto a page with a picture of Jaskier from The Witcher.

In the source code, there is their ssh credentials

```
jaskier:YouHaveTheMostIncredibleNeckItsLikeASexyGoose
```

We can now log into their machine and get the user.txt file
