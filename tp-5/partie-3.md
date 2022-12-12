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

