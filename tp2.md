# TP2 : Appréhender l'environnement Linux

🌞 **S'assurer que le service `sshd` est démarré**

```
[root@vm1 ~]# systemctl status sshd

Active : active (running) since Tue 2022-11-22 15:32:41 CET; 1min 20s ago
```
🌞 **Analyser les processus liés au service SSH**

```
[root@vm1 ~]# ps -ef | grep sshd
root 686 1 0 16:25 ? 00:00:00 /usr/sbin/sshd -D
tartups
root 1217 1187 0 16:30 tty1 00:00:00 grep --color=auto sshd
```

🌞 **Déterminer le port sur lequel écoute le service SSH**

```
[root@vm1 ~]# ss -tulnp | grep sshd
```
c'est sur le port 22

🌞 **Consulter les logs du service SSH**

```
[root@vm1 ~]# journalctl -u sshd
```
