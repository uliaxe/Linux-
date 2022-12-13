# Partie 3 : Configuration et mise en place de NextCloud

🌞 **Préparation de la base pour NextCloud**

- une fois en place, il va falloir préparer une base de données pour NextCloud :

```bash
[root@storage ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.5.16-MariaDB MariaDB Server

MariaDB [(none)]> CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'nextcloud';
Query OK, 0 rows affected (0.009 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.105.1.11';
Query OK, 0 rows affected (0.008 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]> exit
Bye
```

🌞 **Exploration de la base de données**

- afin de tester le bon fonctionnement de la base de données, vous allez essayer de vous connecter, **comme NextCloud le fera plus tard** :
  - depuis la machine `web.tp5.linux` vers l'IP de `db.tp5.linux`
  - utilisez la commande `mysql` pour vous connecter à une base de données depuis la ligne de commande
    - par exemple `mysql -u <USER> -h <IP_DATABASE> -p`
    - si vous ne l'avez pas, installez-là
    - vous pouvez déterminer dans quel paquet est disponible la commande `mysql` en saisissant `dnf provides mysql`
- **donc vous devez effectuer une commande `mysql` sur `web.tp5.linux`**

install mysql

```bash
[root@web ~]# dnf provides mysql
Last metadata expiration check: 5:45:23 ago on Mon Dec 12 16:53:22 2022.
mysql-8.0.30-3.e19_0.x86_64 : MYSQL client programs and shared libraries
Repo        : appstream
Matched from:
Provide    : mysql = 8.0.30-3.e19_0
```

```bash
[root@web ~]# dnf install mysql
```

```bash
[root@web ~]# mysql -u nextcloud -h 10.105.1.12 -p
Enter password:
```

- une fois connecté à la base, utilisez les commandes SQL fournies ci-dessous pour explorer la base

```sql
SHOW DATABASES;
USE <DATABASE_NAME>;
SHOW TABLES;
```

```sql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| nextcloud          |
+--------------------+
2 rows in set (0.00 sec)

mysql> USE nextcloud;
Database changed
mysql> SHOW TABLES;
Empty set (0.00 sec)
```

🌞 **Trouver une commande SQL qui permet de lister tous les utilisateurs de la base de données**

- vous ne pourrez pas utiliser l'utilisateur `nextcloud` de la base pour effectuer cette opération : il n'a pas les droits
- il faudra donc vous reconnectez localement à la base en utilisant l'utilisateur `root`

```sql
mysql> SELECT user FROM mysql.user;
+------------------+
| user             |
+------------------+
| nextcloud        |
| mariadb.sys      |
| mysql            |
| root             |
+------------------+
5 rows in set (0.003 sec)
```

## 2. Serveur Web et NextCloud

⚠️⚠️⚠️ **N'OUBLIEZ PAS de réinitialiser votre conf Apache avant de continuer. En particulier, remettez le port et le user par défaut.**

```bash
[root@web ~]# systemctl restart httpd
```

🌞 **Install de PHP**

```bash
[root@web ~]# dnf config-manager --set-enabled crb
[root@web ~]# dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
[root@web ~]# dnf module list php
[root@web ~]# sudo dnf module enable php:remi-8.1 -y
[root@web ~]# sudo dnf install -y php81-php
```

🌞 **Install de tous les modules PHP nécessaires pour NextCloud**

```bash
[root@web ~]# sudo dnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp
```

🌞 **Récupérer NextCloud**

- créez le dossier `/var/www/tp5_nextcloud/`
  - ce sera notre *racine web* (ou *webroot*)
  - l'endroit où le site est stocké quoi, on y trouvera un `index.html` et un tas d'autres marde, tout ce qui constitue NextCloud :D

```bash
[root@web ~]# mkdir /var/www/tp5_nextcloud/
```

- récupérer le fichier suivant avec une commande `curl` ou `wget` : <https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip>

```bash

[root@web ~]# curl -O https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip
```

- extrayez tout son contenu dans le dossier `/var/www/tp5_nextcloud/` en utilisant la commande `unzip`
  - installez la commande `unzip` si nécessaire
  - vous pouvez extraire puis déplacer ensuite, vous prenez pas la tête

install unzip

```bash
[root@web ~]# dnf install unzip
```

```bash
[root@web ~]# unzip nextcloud-25.0.0rc3.zip -d /var/www/tp5_nextcloud/
```

- contrôlez que le fichier `/var/www/tp5_nextcloud/index.html` existe pour vérifier que tout est en place

```bash
[root@web ~]# ls -l /var/www/tp5_nextcloud/nextcloud/index.html
-rw-r--r--. 1 root root 0 Dec 12 17:09 /var/www/tp5_nextcloud/nextcloud/index.html
```

- **assurez-vous que le dossier `/var/www/tp5_nextcloud/` et tout son contenu appartient à l'utilisateur qui exécute le service Apache**
  - utilisez une commande `chown` si nécessaire

```bash
[root@web ~]# chown -R apache:apache /var/www/tp5_nextcloud/
```

🌞 **Adapter la configuration d'Apache**

- regardez la dernière ligne du fichier de conf d'Apache pour constater qu'il existe une ligne qui inclut d'autres fichiers de conf

```bash

[root@web ~]# cat /etc/httpd/conf/httpd.conf | grep Include
IncludeOptional conf.modules.d/*.conf
Include conf.d/*.conf
```

- créez en conséquence un fichier de configuration qui porte un nom clair et qui contient la configuration suivante :

```bash
[root@web ~]# nano /etc/httpd/conf.d/tp5_nextcloud.conf
<VirtualHost *:80>

  DocumentRoot /var/www/tp5_nextcloud/nextcloud/
 
  ServerName  web.tp5.linux

  <Directory /var/www/tp5_nextcloud/> 
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>

[root@web ~]# cat /etc/httpd/conf.d/tp5_nextcloud.conf
```

🌞 **Redémarrer le service Apache** pour qu'il prenne en compte le nouveau fichier de conf

```bash
[root@web ~]# systemctl restart httpd
```

## 3. Finaliser l'installation de NextCloud

➜ **Sur votre PC**

- modifiez votre fichier `hosts` (oui, celui de votre PC, de votre hôte)
ou se trouve le fichier hosts
  - sous Windows, c'est ici :
C:\Windows\System32\drivers\etc\hosts

  - pour pouvoir joindre l'IP de la VM en utilisant le nom `web.tp5.linux`

- avec un navigateur, visitez NextCloud à l'URL `http://web.tp5.linux`
  - c'est possible grâce à la modification de votre fichier `hosts`
