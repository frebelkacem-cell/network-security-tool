# Réponses — TP 2 GitHub Actions (CI/CD et sécurité)

**Question 1 : Pourquoi les tests ne dépendent-ils pas de l'ouverture réelle du port 443 ?**

Parce qu'ils ne testent pas le réseau, seulement la logique de validation.

* `is_valid_port` vérifie juste que le port est un entier entre 1 et 65535. Elle n'ouvre
  aucune connexion.
* `test_reject_invalid_port` vérifie que `check_port` lève une `ValueError` quand le port est
  invalide. Cette erreur est levée avant toute tentative de connexion, donc aucun socket n'est
  ouvert.

Les tests donnent donc toujours le même résultat, quelle que soit la machine (notre PC ou le
runner GitHub Actions), et que le port 443 soit ouvert ou non. On dit qu'ils sont
déterministes.

**Question 2 : Quelle différence entre `push` et `pull_request` ?**

La différence porte sur le moment où le workflow se lance et sur le code qu'il teste.

* `push` : le workflow se lance dès qu'on pousse un commit sur une branche surveillée (ici
  `main` ou `feature/**`). Il teste le commit qu'on vient de pousser.
* `pull_request` : le workflow se lance quand une PR vers `main` est ouverte ou mise à jour.
  Il teste le résultat d'une fusion hypothétique entre la branche et `main`, c'est-à-dire ce
  que deviendrait `main` si on acceptait la PR.

Le second permet donc de valider les changements avant de les intégrer. C'est aussi lui qui
affiche le résultat des vérifications directement sur la page de la PR (onglet « Checks »).

**Question 3 : Que se passe-t-il si pytest renvoie un code d'erreur ?**

Quand un test échoue, `pytest` se termine avec un code de sortie différent de 0. GitHub
Actions considère alors que l'étape a échoué. La suite est la suivante :

1. l'étape « Run unit tests » est marquée en échec ;
2. le job s'arrête et les étapes suivantes (ici Bandit) sont sautées ;
3. le job, puis le run entier, s'affichent en rouge.

On l'a vu nous-mêmes en modifiant volontairement un test (`assert is_valid_port(443) is
False`). Le log affichait `assert True is False`, et l'étape Bandit n'a pas été exécutée.

À noter : on peut forcer une étape à s'exécuter malgré un échec avec `if: always()`, ou
empêcher une étape de faire échouer le job avec `continue-on-error`. Ce n'est pas le cas dans
notre workflow.

**Question 4 : Pourquoi est-il utile de bloquer l'intégration d'une Pull Request lorsque les tests échouent ?**

Pour protéger `main`. Si on laisse passer du code dont les tests échouent, on introduit des
bugs (des régressions) dans la branche qui est censée toujours fonctionner.

Bloquer la PR a trois avantages :

* `main` reste stable et prête à être déployée ;
* le développeur corrige le problème tout de suite, pendant qu'il a encore son code en tête,
  au lieu de le découvrir des semaines plus tard (ou qu'un collègue le découvre en
  production) ;
* le contrôle est automatique : il ne dépend pas de la vigilance d'un relecteur.

En pratique, ce blocage se configure sur GitHub avec une règle de protection de branche.

**Question 5 : Quelle différence entre le contrôle SAST (Bandit) et les tests fonctionnels pytest ?**

Ils ne cherchent pas la même chose.

* pytest exécute le code et vérifie qu'il fait bien ce qu'on attend. Il répond à la question :
  « Est-ce que le code fonctionne correctement ? »
* Bandit lit le code sans l'exécuter (analyse statique, ou SAST). Il cherche des pratiques
  connues pour être dangereuses : fonctions risquées, mots de passe écrits en dur, mauvaise
  utilisation de certaines bibliothèques… Il répond à la question : « Est-ce que le code
  contient des failles de sécurité connues ? »

Un code peut passer tous les tests et contenir quand même une faille, et inversement. Les deux
contrôles sont donc complémentaires.

## Bonus : `python -m compileall -q network_check.py`

Cette commande compile le fichier en bytecode Python (`.pyc`) sans l'exécuter. Elle vérifie
seulement que la syntaxe est correcte. Elle est très rapide et repère tout de suite un fichier
mal écrit (principe du fail fast).

En revanche, elle ne remplace ni pytest ni Bandit :

* elle ne vérifie pas la logique : un code peut compiler sans erreur et donner de mauvais
  résultats. Ça, c'est le rôle de pytest ;
* elle ne cherche pas de failles : un code dont la syntaxe est correcte peut contenir des
  pratiques dangereuses. Ça, c'est le rôle de Bandit.

C'est un premier filtre à placer avant les deux autres, pas un remplaçant.
