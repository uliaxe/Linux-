# Partie 4 : Automatiser la résolution du TP

L'idée de cette partie 4 est simple : **écrire un script `bash` qui automatise la résolution de ce TP 5**.

Autrement dit, vous devez avoir un script qui :

- **déroule les éléments de la checklist** qui sont automatisables
  - désactiver SELinux
  - donner un nom à chaque machin
- **MariaDB** sur une machine
  - install
  - conf
  - lancement
  - préparation d'une base et d'un user que NextCloud utilisera
- **Apache** sur une autre
  - install
  - conf
  - lancement
  - télécharge NextCloud
  - setup NextCloud
- affiche des **logs** que vous jugez pertinents pour montrer que le script s'exécute correctement
- affiche, une fois terminé, **une phrase de succès** comme quoi tout a bien été déployé