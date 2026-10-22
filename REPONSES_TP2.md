# Réponses — TP 2 GitHub Actions : CI/CD et sécurité

**Question 1 : pourquoi les tests ne dépendent-ils pas de l'ouverture réelle du port 443 ?**

`is_valid_port` ne fait que vérifier le type et la plage de valeur du port (un entier entre 1
et 65535) : elle ne tente aucune connexion réseau. `test_reject_invalid_port` vérifie que
`check_port` lève bien une `ValueError` *avant* d'ouvrir un socket, dès lors que le port est
invalide. Les tests valident donc uniquement la logique de validation, pas l'état réel du
réseau : ils sont déterministes et donnent le même résultat sur n'importe quelle machine
(y compris le runner GitHub Actions), peu importe si le port 443 est réellement ouvert ou non.

**Question 2 : quelle différence entre `push` et `pull_request` ?**

`push` déclenche le workflow à chaque fois qu'un commit est poussé directement sur une branche
qui correspond au filtre (ici `main` ou `feature/**`) ; le job s'exécute alors sur le commit
poussé. `pull_request` déclenche le workflow quand une Pull Request est ouverte ou mise à jour
vers une branche cible (ici `main`) ; il s'exécute sur le résultat de la fusion hypothétique
entre la branche source et la branche cible, ce qui permet de valider les changements *avant*
qu'ils ne soient intégrés. C'est aussi l'événement utilisé pour les vérifications de statut
affichées directement sur la PR (onglet "Checks").

**Question 3 : que se passe-t-il si pytest renvoie un code d'erreur ?**

Quand `pytest` échoue, la commande se termine avec un code de sortie différent de 0. GitHub
Actions considère alors l'étape "Run unit tests" comme échouée : le job s'arrête immédiatement
et les étapes suivantes (ici "Security scan with Bandit") sont automatiquement **sautées**
(sauf si elles sont configurées avec `continue-on-error` ou `if: always()`). Le job entier, et
donc le run du workflow, est marqué en échec (croix rouge). C'est exactement ce qu'on a observé
dans la capture du run "Demonstrate failing CI test".

**Question 4 : pourquoi est-il utile de bloquer l'intégration d'une Pull Request lorsque les tests échouent ?**

Bloquer la fusion tant que les tests échouent protège la branche principale (`main`) contre
l'introduction de régressions ou de code cassé : `main` reste toujours dans un état stable et
déployable. Cela force aussi le développeur à corriger le problème immédiatement, pendant qu'il
a encore le contexte en tête, plutôt que de laisser le bug se propager et être découvert plus
tard (souvent par quelqu'un d'autre, en production). C'est une porte de qualité (quality gate)
automatique, qui ne dépend pas de la vigilance humaine.

**Question 5 : quelle différence entre ce contrôle SAST (Bandit) et les tests fonctionnels pytest ?**

Bandit est un outil d'analyse statique (SAST — Static Application Security Testing) : il lit le
code source sans l'exécuter, à la recherche de motifs connus comme risqués d'un point de vue
sécurité (fonctions dangereuses, secrets en dur, usages non sûrs de certaines bibliothèques,
etc.). Il ne vérifie pas que le programme fonctionne correctement. `pytest`, lui, exécute
réellement le code et vérifie que son comportement correspond à ce qui est attendu
(tests fonctionnels) : il valide la logique métier, pas la sécurité du code en tant que tel.
Les deux contrôles sont complémentaires : l'un répond à "le code est-il correct ?", l'autre à
"le code contient-il des pratiques dangereuses connues ?".

## Bonus — `python -m compileall -q network_check.py`

Cette commande compile le fichier Python en bytecode (`.pyc`) sans l'exécuter. Elle vérifie
uniquement que le fichier est **syntaxiquement valide** (pas d'erreur de syntaxe qui empêcherait
même l'interpréteur de le charger). Elle est très rapide et permet d'échouer tôt en cas de
fichier mal formé.

Elle ne remplace ni `pytest` ni `Bandit` car :
- elle ne vérifie **aucune logique métier** (un code peut compiler parfaitement tout en étant
  fonctionnellement incorrect) — c'est le rôle de `pytest` ;
- elle ne détecte **aucune vulnérabilité de sécurité** (un code syntaxiquement valide peut très
  bien contenir des pratiques dangereuses) — c'est le rôle de `Bandit`.

C'est donc un contrôle complémentaire et préalable (fail fast), pas un substitut.
