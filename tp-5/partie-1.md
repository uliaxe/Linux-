# Partie 1 : Mise en place et maîtrise du serveur Web

## 1. Installation

🌞 **Installer le serveur Apache**

- paquet `httpd`
- la conf se trouve dans `/etc/httpd/`
  - le fichier de conf principal est `/etc/httpd/conf/httpd.conf`
  - je vous conseille **vivement** de virer tous les commentaire du fichier, à défaut de les lire, vous y verrez plus clair
    - avec `vim` vous pouvez tout virer avec `:g/^ *#.*/d`

```bash
[root@web ~] #sudo yum install httpd
```

🌞 **Démarrer le service Apache**

- le service s'appelle `httpd` (raccourci pour `httpd.service` en réalité)
  - démarrez-le
  - faites en sorte qu'Apache démarre automatiquement au démarrage de la machine
    - ça se fait avec une commande `systemctl` référez-vous au mémo
  - ouvrez le port firewall nécessaire
    - utiliser une commande `ss` pour savoir sur quel port tourne actuellement Apache
    - une portion du mémo commandes est dédiée à `ss`

```bash
[root@web ~] #sudo systemctl start httpd
[root@web ~] #sudo systemctl enable httpd
[root@web ~] #sudo firewall-cmd --add-port=80/tcp --permanent
success
[root@web ~] #sudo firewall-cmd --reload
success
[root@web ~] #ss -tulpn | grep httpd
tcp LISTEN 0 551 *:80 *:* users:(("httpd" ,pid=  1248,fd=4), ("httpd", pid=1247,fd=4), ("httpd",pid=1246,fd=4), ("httpd", pid=1244,fd=4))
```

🌞 **TEST**

- vérifier que le service est démarré

```bash
[root@web ~] #sudo systemctl status httpd
httpd.service - The Apache HTTP Server
loaded : loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
active : active (running) since Mon 2022-12-12 16:20:38 CET; 6min ago
```

- vérifier qu'il est configuré pour démarrer automatiquement

```bash
[root@web ~] #sudo systemctl is-enabled httpd
enabled
```

- vérifier avec une commande `curl localhost` que vous joignez votre serveur web localement

```bash
[root@web ~] #curl localhost
<html>
<head>
<title>It works!</title>
</head>
<body>
<h1>It works!</h1>
</body>
</html>
```

- vérifier depuis votre PC que vous accéder à la page par défaut
  - avec votre navigateur (sur votre PC)
  - avec une commande `curl` depuis un terminal de votre PC (je veux ça dans le compte-rendu, pas de screen)

```bash
  # curl http://localhost:80
```

### 2. Avancer vers la maîtrise du service

🌞 **Le service Apache...**

- affichez le contenu du fichier `httpd.service` qui contient la définition du service Apache

```bash
[root@web ~] #cat /usr/lib/systemd/system/httpd.service
[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation==man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
OOMPPolicy=continue

[Install]
WantedBy=multi-user.target
```

🌞 **Déterminer sous quel utilisateur tourne le processus Apache**

- mettez en évidence la ligne dans le fichier de conf principal d'Apache (`httpd.conf`) qui définit quel user est utilisé

```bash
[root@web ~] #cat /etc/httpd/conf/httpd.conf | grep User
User apache
```

- utilisez la commande `ps -ef` pour visualiser les processus en cours d'exécution et confirmer que apache tourne bien sous l'utilisateur mentionné dans le fichier de conf
  - filtrez les infos importantes avec un `| grep`

```bash
[root@web ~] #ps -ef | grep httpd
root     1244     1  0 16:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   1245  1244  0 16:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   1246  1244  0 16:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   1247  1244  0 16:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   1248  1244  0 16:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
root    1253 1244  0 16:50 tty3      00:00:00 /usr/sbin/httpd -DFOREGROUND
```

- la page d'accueil d'Apache se trouve dans `/usr/share/testpage/`
  - vérifiez avec un `ls -al` que tout son contenu est **accessible en lecture** à l'utilisateur mentionné dans le fichier de conf

```bash
[root@web ~] #ls -al /usr/share/testpage/
total 12
drwxr-xr-x. 2 root root 24 Dec 12 16:13 .
drwxr-xr-x. 81 root root 4096 Dec 12 16:13 ..
-rw-r--r--. 1 root root 7620 Jul 27 20.05 index.html
```

🌞 **Changer l'utilisateur utilisé par Apache**

- créez un nouvel utilisateur
  - pour les options de création, inspirez-vous de l'utilisateur Apache existant
    - le fichier `/etc/passwd` contient les informations relatives aux utilisateurs existants sur la machine
    - servez-vous en pour voir la config actuelle de l'utilisateur Apache par défaut (son homedir et son shell en particulier)

```bash
[root@web ~] #useradd -d /home/apache2 -s /bin/bash
apache2

[root@web ~] #cat /etc/passwd | grep apache2
apache2:x:1000:1000::/home/apache2:/bin/bash
```

- modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur

```bash
[root@web ~] #sed -i 's/User apache/User apache2/g' /etc/httpd/conf/httpd.conf
```

- montrez la ligne de conf dans le compte rendu, avec un `grep` pour ne montrer que la ligne importante

```bash
[root@web ~] #cat /etc/httpd/conf/httpd.conf | grep User
User apache2
```

- redémarrez Apache

```bash
[root@web ~] #systemctl restart httpd
```

- utilisez une commande `ps` pour vérifier que le changement a pris effet
  - vous devriez voir un processus au moins qui tourne sous l'identité de votre nouvel utilisateur

```bash
[root@web ~] #ps -ef | grep httpd
root     1604     1  0 17:02 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache2  1605  1604  0 17:02 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache2  1606  1604  0 17:02 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache2  1607  1604  0 17:02 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache2  1608  1604  0 17:02 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
root    1831  1215 0 17:02 tty3      00:00:00 /usr/sbin/httpd -DFOREGROUND
```

🌞 **Faites en sorte que Apache tourne sur un autre port**

- modifiez la configuration d'Apache pour lui demander d'écouter sur un autre port de votre choix
  - montrez la ligne de conf dans le compte rendu, avec un `grep` pour ne montrer que la ligne importante

```bash
[root@web ~] #sed -i 's/Listen 80/Listen 8080/g' /etc/httpd/conf/httpd.conf
[root@web ~] #cat /etc/httpd/conf/httpd.conf | grep Listen
Listen 8080
```

- ouvrez ce nouveau port dans le firewall, et fermez l'ancien

```bash
[root@web ~] #firewall-cmd --add-port=8080/tcp --permanent
success
[root@web ~] #firewall-cmd --remove-port=80/tcp --permanent
success
[root@web ~] #firewall-cmd --reload
success
```

- redémarrez Apache

```bash
[root@web ~] #systemctl restart httpd
```

- prouvez avec une commande `ss` que Apache tourne bien sur le nouveau port choisi

```bash
[root@web ~] #ss -tulpn | grep httpd
tcp    LISTEN     0      511     *:8080                *:*                   users:(("httpd",pid=1862,fd=4),("httpd",pid=1861,fd=4),("httpd",pid=1860,fd=4),("httpd",pid=1857,fd=4))
```

- vérifiez avec `curl` en local que vous pouvez joindre Apache sur le nouveau port
  
```bash
[root@web ~] #curl localhost:8080
<html>
<head>
<title>It works!</title>
</head>
<body>
<h1>It works!</h1>
</body>
</html>
```

- vérifiez avec votre navigateur que vous pouvez joindre le serveur sur le nouveau port

10.105.1.11:8080
