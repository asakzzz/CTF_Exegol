# Silence

## Intel

We scan the ip with nmap :

```bash
nmap -sC -sV ip
```

**Output:**

```
Starting Nmap 7.991 ( https://nmap.org ) at 2026-09-18 07:35 +0000
Nmap scan report for 10.10.0.23
Host is up (0.023s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.68 ((Debian))
|_http-server-header: Apache/2.4.68 (Debian)
|_http-title: Adam Ondra
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.86 seconds
```

Since there is a web server, we will check it out:

Nothing interesting on the web page for now, except a comment in the source code : `Silence is golden`

We bust dirs with `gobuster`

```bash
gobuster dir -u http://10.10.0.23 -w ~/medium.txt -b 404
```

**Output:**

```
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.0.23
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/asakz/medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
hidden               (Status: 301) [Size: 349] [--> http://10.10.0.23/hidden/]
server-status        (Status: 403) [Size: 315]
Progress: 207641 / 207641 (100.00%)
===============================================================
Finished
===============================================================
```

We can now visit the hidden folder, it contains a zip file

Downloading and extracting it, we get two files

```
ClimbersStats.xlsx.gpg  .hidden-key
```

The first one is a pgp encrypted file, we assume the second one is the key. We can decrypt it and see what is inside

```bash
gpg --import .hidden-key
gpg --decrypt ClimberStats.xlsx.gpg > ClimberStats.xlsx
cat ClimberStats.xlsx
```

**Output:**

```
t�S
   _rels/.rels���N�0
                    ��{�*�5����LH�!4�$n��ă��D
                                             ��Îq~�b��Ln,�0&K^�UY��&c}����qy/6͢~�8GRoC*r�OJ���Aʤ{t�J
e�MK��c�d=@�r]Uw2�d��Y�qgV����Mmk5nIz>3�W"�!v�JL�|�8�
y�e}����
        0HM�!���ӷ�!���阘���rpb�ͼ�0gt{M#}HL�3_J�Z����P��4���t�Sxl/workbook.xml�S�r�0}�Wx��p)0�
                                                                                             5x���&�ɳl���,y�%@:���e���>���������͹��3+��Y8
                                         �*ׅP��}{H3�Y��R+��,�Y�[��9dZ<z�l�*�f��6���v��>��Ie��:���xk:�[mċV��.7Zʘ�9^�Q��"�vP<����$T�O1�]��O��$
�o:r;�OO����aK�λ�*;� ����$�f!(`��!�0�nN��}TD!l9(?� քv��.�jo@"'�����Ѣ;�R����I��@' �%���ُ��h�̦� Z��Axl/styles.xml�X�n�0}�WX~_�4M��t���NL{��5�*M{p��Uc#�iC�~טH�M��Jɋ��=�_la'�ZU=Q��N|��He�D��y��#m����#\S�<���"�D�T��\����ӵ�$ӖTqo��3�"L�8�*��F�\
�l��sR1�8xҒK�4L���]��L[�׊9�CA�`�mc��8��7��ÚC�H`������XN���Kt�HL�����
H]�&�VĨ%��m���lj��w }J9��K�!����U��E;��b�w]��                       �2X��� �1RHA�}�pMq}��b
lXj!7_��㐬Q){i��n�
       ]��������Ed"k�3]*&�2a�c(S��@\��4[�,Y�A��ʷ*�o��[���v���R�e�~�L�f�0���:�9�9�9�9������������L����\�g3�����s|��1~��:��G���L�u�\�����E�����z��[,7L���K��UE���وp�&����و4{��T����9�#��O�Q����5�-U)���r9���輦0����P�ن;��t�Sxl/worksheets/sheet1.xml�WMo�8��t��a)��Ek��
                                                                     �u�N[`o�DYD(RKRv�_�CR�I(rh�CLG�fތ������:',$�l��u0KxJ�q�~{ؾ���T���r���K���ř�G�c���͕*�<O&9.���38ɸ(���8z��桂z���x"̵��kb�,#	��*0S6��)H_椔M���U�R��Pj�O'ō=i�� ^A�%��$�E�ڰʹ7Qg��&��Ǫ|�K(�@(Q���Z��_������S�K���pV�#�c��4��Cs�^��j��P��8[���M�=��w�ϲ�vd��[ȯ�H6��/A�{�0X��j�?������=�c�A�c��L�!:�1ŉ�i��]�(��/Ł�6@�3TQ�S8.�	2�o"�y��'�I��w�_��}���]xeh�O��p��Q�t\_7�T��-�~q�,\���m6먻��:�?Ӑut��]7�ٚ��V�L
                                                                          ?H��;��ӹ�-KГOX3I��:��k�%��0o�L��mm�
             ��濦��R���A�J*^�Y���$M15�z��0�)�E�GmÄ�$����ZȰ�
�@��Bl\C�#�Q���!�w/Rh��3��[                                � ��,����r:y;	C�Q
�������|!�듍<�Y%?�Q�7�!B�NE��i{�^pC�Q1�t8i��q��{��S��:-1?мC߄���oy78����ɭȰ�F�=�!����z08��%V훽���7K~P9���t�S[Content_Types].xml�T�O� ��h������n?���y�-�|���{��e��sY��{�/^ �md���:�U�&x�PLs���,�k4����րKB�r9��77�8V��k*T �����<s�]�zĢs��V$xVR}F�H���Onk�R���ݬ��;p�7��H ���`����d���#��qU	��bO��Ҝcm%�akKb([���x|E�V�O}�@��w�d��3{�1�A�0�@�FF/ݒ
��}r�	ǭn����R9��ԂQ�$VI'�B�z�k�ܥ_�p@6=��]��`Ty  dLϻ��!M!`��u[�!�S��
�5&!x�<]J��|h�|�z����CM�`�5[���X��U^ָY��B�w~[��!�C��H�L6��?����i�������}>�}�>?�ڸ�3X8��w�NM �E��w������:~�6[9����-�O�QF�_z�	P�;�d�t�S��4���                     l���~v�ܶ=ؔ�o]IC�Ւ��f�
?xl/worksheets/sheet1.xm�t�SO��z�%�            _rels/.rel�t�S4��\�p'xl/workbook.xm�t�S�ن;���$ˠALxl/_rels/workbook.xml.rel�t�S�PxJ��                                                            �docProps/core.xm�t�S�Lml.rels��Mk�0xl/sharedStrings.xm�t�S*v �k�O
                      *�z�docProps/app.xm�t�S9���!docProps/custom.xm�t�S�;�d��[Content_Types].xmlPK��H��4���C
_�{�0�  �r�3�R�b�+/2�m��=�?"�#ʯ��R6w�8Y.�_f�zJN��ɺ��Q/��IӜ�S�[�n+P�s���c�
�H%                                
```

Our terminal cannot read this file, we can read it by using libreoffice of a python script gathered on internet

```bash
python3 -c "                                                  
import zipfile
z = zipfile.ZipFile('ClimbersStats.xlsx')
print(z.read('xl/sharedStrings.xml').decode())
"
```

**Output:**

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<sst xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main" count="16" uniqueCount="16"><si><t xml:space="preserve">Climber Name</t></si><si><t xml:space="preserve">Username</t></si><si><t xml:space="preserve">Password</t></si><si><t xml:space="preserve">Max grade</t></si><si><t xml:space="preserve">Adam Ondra</t></si><si><t xml:space="preserve">adam</t></si><si><t xml:space="preserve">bibliographieSeemsTough2022</t></si><si><t xml:space="preserve">9c (Secret project)</t></si><si><t xml:space="preserve">Magnus Mitbo</t></si><si><t xml:space="preserve">magnus</t></si><si><t xml:space="preserve">youtubeIsClimbingToo</t></si><si><t xml:space="preserve">9b Ali Hulk</t></si><si><t xml:space="preserve">Janja Garnbret</t></si><si><t xml:space="preserve">janja</t></si><si><t xml:space="preserve">theClimbingMonster</t></si><si><t xml:space="preserve">9a Seleccio Natural</t></si></sst>
```

It seems to be an excel file, let's check it

| Climber Name | Username | Password | Max grade |
| :--- | :--- | :--- | :--- |
| Adam Ondra | adam | bibliographieSeemsTough2022 | 9c (Secret project) |
| Magnus Mitbo | magnus | youtubeIsClimbingToo | 9b Ali Hulk |
| Janja Garnbret | janja | theClimbingMonster | 9a Seleccio Natural |

Now we just have to try to log in

The only user that can didn't change their password is `janja`

```bash
ssh janja@ip
janja's password :
```

Now that we are logged in, we can `ls` and grab the flag
