# Réponses — TP 1 Git et GitHub

**Question 1 : Quelle est la différence entre `git add` et `git commit` ?**

`git add` place les modifications d'un ou plusieurs fichiers dans la "zone de staging" (index) :
cela prépare ce qui sera inclus dans le prochain commit, sans encore l'enregistrer dans
l'historique. `git commit` enregistre ensuite de façon permanente le contenu de la zone de
staging dans l'historique du dépôt, avec un message décrivant le changement. En résumé : `add`
sélectionne ce qu'on veut sauvegarder, `commit` crée réellement le point de sauvegarde.

**Question 2 : Que représente un commit dans Git ?**

Un commit est une photographie (snapshot) de l'état du projet à un instant donné, accompagnée
d'un identifiant unique (hash SHA-1), d'un auteur, d'une date et d'un message décrivant le
changement. Les commits sont chaînés les uns aux autres (chaque commit référence son ou ses
parents), ce qui forme l'historique du projet et permet de revenir à n'importe quelle version
antérieure.

**Question 3 : Pourquoi créer une branche plutôt que modifier directement la branche principale ?**

Travailler sur une branche dédiée permet d'isoler le développement d'une nouvelle
fonctionnalité (ou d'un correctif) sans risquer de casser la branche principale (`main`), qui
reste stable et potentiellement utilisée par d'autres. Cela permet aussi de tester, faire des
allers-retours et des commits intermédiaires librement, puis de faire relire le travail via une
Pull Request avant de l'intégrer. Si la fonctionnalité ne convient pas, il suffit d'abandonner
la branche sans impact sur `main`.

**Question 4 : Quel est le rôle d'une Pull Request dans un projet collaboratif ?**

Une Pull Request (PR) est une demande d'intégration des changements d'une branche vers une
autre (ici `feature/custom-port` vers `main`). Elle permet de visualiser le diff des
modifications, de les discuter via des commentaires, de les faire relire et valider par
d'autres membres de l'équipe (revue de code), et éventuellement de lancer des vérifications
automatiques (CI) avant la fusion. C'est un point de contrôle qui garantit la qualité et la
traçabilité des changements avant qu'ils n'atteignent la branche principale.
