# Partie 2 : Mise en place et maîtrise du serveur de base de données

🌞 **Install de MariaDB sur `db.tp5.linux`**

- faites en sorte que le service de base de données démarre quand la machine s'allume
  - pareil que pour le serveur web, c'est une commande `systemctl` fiez-vous au mémo

```bash
[root@db ~]# yum install mariadb-server
[root@db ~]# systemctl enable mariadb
[root@db ~]# systemctl start mariadb
```

🌞 **Port utilisé par MariaDB**

- vous repérerez le port utilisé par MariaDB avec une commande `ss` exécutée sur `db.tp5.linux`
  - filtrez les infos importantes avec un `| grep`

```bash
[root@db ~]# ss -tulpn | grep mariadb
tcp LISTEN 0 80     *:3306 *:* users:(("mariadb",pid=3467,fd=15))
```

- il sera nécessaire de l'ouvrir dans le firewall

```bash
[root@db ~]# firewall-cmd --add-port=3306/tcp --permanent
success
[root@db ~]# firewall-cmd --reload
success
```

🌞 **Processus liés à MariaDB**

- repérez les processus lancés lorsque vous lancez le service MariaDB
- utilisz une commande `ps`
  - filtrez les infos importantes avec un `| grep`

```bash
[root@db ~]# ps aux | grep mariadb
mysql     3467  0.0  0.0  1084832  95844 ?        Ssl   21:50   0:00 /usr/libexec/mysqld --basedir=/usr 
root     3536  0.0  0.1 3876   2024 tty2    S+   21:55   0:00 grep --color=auto mariadb
```

- securisation de la base 

```bash
[root@db ~]# mysql_secure_installation
```
