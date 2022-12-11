# scripting 

all hostname : ``` hostnamectl | grep tp3 | cut -d ":" -f2 ```

l'adresse IP de la machine
ip : ``` ip a | grep 10.13.237. | cut -d " " -f6 | cut -d "/" -f1 ```

RAM totale : ``` free -h | tr -s " " | cut -d " " -f2 | head -n 2 | tail -n 1 ```

ram free : ``` free -h | tr -s " " | cut -d " " -f4 | head -n 2 | tail -n 1 ```


version du noyau : ``` uname -r ```

nom de l'os : ``` cat /etc/os-release | grep PRETTY_NAME | cut -d "=" -f2 | cut -d '"' -f2 ```

"espace restant sur le disque dur : ``` df -h | tr -s " " | cut -d " " -f4 | head -n 2 | tail -n 1 ```
"
top 5 des processus : ``` ps -aux | tr -s " " | cut -d " " -f11 | head -n 6 | tail -n 5 ```

liste des ports qui écoutent : ``` netstat -tulpn | tr -s " " | cut -d " " -f4 | cut -d ":" -f2 | head -n 6 | tail -n 5 ```

lien vers une image/gif : ``` curl 