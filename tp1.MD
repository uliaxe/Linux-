# TP 1 : Are you dead yet ?

## sommaire

- solution 1
- solution 2
- solution 3
- solution 4
- soluiton ultime (la meilleure)

## solution 1

➜ Première solution :

``` cd .. ```
Pour aller a la racine du dossier

``` ls ```
Pour voir les fichiers

``` cd lib64 ```
Pour aller dans le dossier lib64

``` ls ```
Pour voir les fichiers

``` sudo rm -r security ```
Pour supprimer le dossier security

``` reboot ```
Pour redémarrer la vm

Nous venons alors de casser toutes les sessions y compris le root. Nous ne pouvons plus nous connecter à la vm.

## solution 2

➜ deuxième solution :

Arrêter tous les processus de root en cours (dont le système : Processus ID 1) de manière forcée :

```sudo killall -u root``` -> Arrêt forcé des processus en cours sur l'utilisateur root (killall)

## solution 3

➜ troisième solution :

```sudo passwd -l root``` -> Bloquer le compte root

## solution 4

➜ quatrième solution :

Création d'une tâche répétée qui va éteindre la machine toutes les minutes :

```sudo crontab -e``` -> Ouverture du gestionnaire des tâches répétées (le fichier crontab) en root

```* * * * * /sbin/shutdown -h now``` -> Ajout d'une ligne au fichier crontab signifiant que l'on veut éteindre notre machine toutes les minutes

## solution ultime

➜ Solution ultime :

[solution ultime](giphy.gif)