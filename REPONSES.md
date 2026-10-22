# Réponses — TP 1 Git et GitHub

**Question 1 : Quelle est la différence entre `git add` et `git commit` ?**

Ce sont deux étapes qui se suivent.

`git add` sert à choisir les modifications qu'on veut garder. On les place dans une zone
d'attente, appelée zone de staging (ou index). À ce stade, rien n'est encore enregistré dans
l'historique.

`git commit` vient ensuite. Il prend tout ce qui se trouve dans la zone de staging et
l'enregistre définitivement dans l'historique, avec un message qui explique le changement.

Pour faire simple : avec `add`, on prépare le paquet, et avec `commit`, on l'envoie. Dans le
TP, on a fait un `git add` sur `network_check.py`, puis un `git commit` pour l'enregistrer.

**Question 2 : Que représente un commit dans Git ?**

Un commit, c'est une photo du projet à un moment précis. Il contient :

* l'état des fichiers à ce moment-là ;
* un identifiant unique, le hash SHA-1 (la longue suite de caractères qu'on voit dans
  `git log`) ;
* l'auteur et la date ;
* un message qui décrit le changement.

Chaque commit pointe vers le commit précédent, son « parent » (une fusion en a deux). Mis bout
à bout, les commits forment l'historique du projet. On peut ainsi revenir à n'importe quelle
version antérieure.

**Question 3 : Pourquoi créer une branche plutôt que modifier directement la branche principale ?**

Parce que `main` doit rester stable. Si on la modifie directement et qu'on casse quelque chose,
tous ceux qui l'utilisent sont touchés.

En travaillant sur une branche à part, comme `feature/custom-port` dans le TP, on peut :

* développer et tester sans risque, avec autant de commits intermédiaires qu'on veut ;
* faire relire son travail grâce à une Pull Request avant de l'intégrer ;
* abandonner la branche si l'idée ne fonctionne pas, sans aucun impact sur `main`.

**Question 4 : Quel est le rôle d'une Pull Request dans un projet collaboratif ?**

Une Pull Request (PR), c'est une demande pour intégrer le travail d'une branche dans une
autre. Dans le TP, on a demandé à fusionner `feature/custom-port` dans `main`.

Concrètement, elle permet de :

* voir exactement ce qui a changé (le diff) ;
* discuter des modifications avec des commentaires ;
* faire relire et valider le code par d'autres membres de l'équipe ;
* lancer des vérifications automatiques (c'est ce qu'on met en place au TP2).

C'est un point de contrôle : rien n'arrive sur `main` sans passer par là. Et comme tout est
conservé, on sait toujours qui a changé quoi, et pourquoi.
