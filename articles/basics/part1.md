# Série: Des circuits au python: voyage à travers les abstractions en Informatique

# Introduction
Beaucoup de développeurs se passionnent aujourd’hui pour le Big Data, le full-stack ou l’IA… mais peu prennent le temps de découvrir la magie qui se cache sous leurs outils quotidiens.

J'ai passé une partie de l'année dernière à explorer les différents niveaux d'abstractions, et j'ai remarqué que c'était un "must"
pour les programmeurs de connaître ce qui se cache sous leurs écrans.

Peut-être que ce n'est pas que moi, mais vous aussi vous avez probablement entendu des phrases comme:
- Les langages de haut-niveau comme le Python sont plus lents
- Les langages de bas-niveau comme le C (ou C++) sont plus rapides et plus performants
- Un vrai programmeur est celui qui utilise l'assembleur(je fus du même accord jadis)
- et d'autres...
Dans cette série, nous allons explorer et mettre à l’épreuve ces idées, du plus bas niveau (assembleur) jusqu’au Python.

Partons d'un exemple concret:

```python
print("Bonjour le monde")
```

> Supposons qu'on utilise l'implémentation par défaut de python(Cpython) car les autres implémentations fonctionnent d'une manière différente.

Normalement, ce programme affiche notre message "Bonjour le monde" sur l'écran.
Supposons encore que nous sommes sur un système Linux et décortiquons ce petit programme qu'on vient d'écrire.

Il faut d’abord comprendre que certaines tâches, comme afficher des caractères à l’écran, sont plus complexes qu’on ne le croit.
C'est très complexe et pourrait remplir des dizaines d'articles pour expliquer comment on part de la lumière, en passant par le pixel,
puis par notre système de fenêtrage (x11, wayland, ...), jusqu'à ce que ces caractères soient visibles à nos yeux sur notre écran.
Et ceci n'est même pas proche du résumé de ce qui se passe réellement!

La prochaine fois que vous utiliserez print, souvenez-vous que chaque caractère traverse un chemin complexe depuis votre code jusqu’à l’écran.
Comme le disait Albert Camus : “Le rôle de l'artiste ne se sépare pas de devoirs difficiles.” et c’est exactement dans ces détails invisibles que réside la vraie beauté du code.
Donc quand on vous a appris à afficher "Bonjour le monde" dans votre langage de programmation préféré, vous avez acquis un grand pouvoir que vous ne le pensez!

Heureusement, nous n’avons pas besoin de réinventer la roue pour afficher chaque fois du texte à l’écran.
Même les concepteurs des langages de programmation (ou des émulateurs) n’ont pas le temps de s’occuper eux-mêmes de cette tâche fastidieuse.

Sous Linux, le noyau nous fournit des centaines d’appels système — des fonctions primitives qui permettent aux programmes d’interagir directement avec le matériel et le système.

Et dans notre cas précisément, on a un appel système pour afficher du texte à l'écran... l'appel système `write`:

```C
ssize_t write(int fd, const void *buf, size_t count);
```

La signature est en C, le langage préféré de ceux qui aiment se “salir les mains” en manipulant directement la machine.

Ces appels système peuvent être utilisés depuis l’assembleur ou le C.
Apprendre toutes les variantes d’assembleur pour chaque processeur serait très fastidieux,
d’où le choix du C ou de Rust pour la plupart des projets bas-niveau : ils offrent une interface plus uniforme tout en restant proches du matériel.

> Un caractère est représenté sur 1 octet (1 octet contient 8 bits).

Alors, comme vous pouvez l'observer, on a 3 arguments pour `write`:
- fd: File Descriptor. Pour l'écran, fd est toujours égal à 1.
Vous pouvez écrire aussi du texte dans un fichier. Dans Linux, chaque fichier ouvert possède un
numéro appelé `file descriptor`. Vous pouvez passer ce numéro en argument pour écrire dans ce
fichier au lieu d'écrire sur l'écran. Ce numéro est toujours un entier (jamais de 1.84 dans fd)!

Dans Linux, tout est fichier! Même l'écran est un fichier avec `fd=1`, votre clavier `fd=0`, ...!

- buf: Buffer. C'est le texte que vous voulez écrire sur l'écran.
Puisque nous sommes en C, ceci est un pointeur `void*` pour indiquer qu'on n'écrit pas que du texte
dans un fichier ou sur l'écran (`void*` est plus utilisé pour généraliser la notion de pointeur
lorsqu'on n'est pas sûr du type de ce qu'on va recevoir en argument).
Si on écrivait que du texte, on utiliserait `char*` au lieu de `void*`. Un peu de C de temps en
temps ne fait pas de mal!

- count: C'est le nombre de bytes (de caractères) qu'on va afficher à l'écran.
Comme vous vous en doutez, ce nombre est calculé dynamiquement dans des langages de haut-niveau pour
que vous n'ayez pas toujours à compter combien de caractères se trouvent dans "Hello world"!

En python, on a une fonction similaire à `write` dans le module `os`:
```python
import os
import sys

fd = sys.stdout.fileno()  # sys.stdout correspond à la sortie standard (l'écran)
os.write(fd, "Bonjour le monde\n".encode())  # Ceci signifie "Écris sur l'écran le message suivant"
```

Lorsque vous lancez ce programme, vous verrez ceci:
```
Bonjour le monde
17
```

Ce nombre qui s'affiche c'est le nombre de caractères (bytes) qu'on vient d'afficher.
Miraculeusement, on a la fonction `os.write` qui ressemble à notre `write` que Linux nous offre en C:
```python
write(fd, data, /)  # Signature de la fonction `write` de `os`
```

Ce qui manque seulement c'est le `count`, mais j'avais bien précisé que le nombre de bytes était
calculé dynamiquement dans les langages de haut-niveau. Si c'était en C, compter qu'on a `17`
caractères serait de votre responsabilité.

À noter que cette fonction fait appel à l'appel système `write`, c'est juste une abstraction comme
vous pouvez l'observer. Après tout, le `count` est dynamiquement calculé pour vous... ce qui montre
que derrière les rideaux se cache une autre fonction primitive qui est appelé avec tous les
arguments bien remplis pour vous (cette fonction primitive étant `write`)!

Ceci était juste un petit exemple pour vous montrer que tout ce que nous utilisons tous les jours
pour faire nos REST APIs, pour exploiter les frameworks, pour faire du logging en temps réel,
pour faire des simulations dans des projets scientifiques, pour faire de la robotique, est basé
sur d'autres choses assez poussées plus que ce que nous faisons.
Certes, en réalité même `write` se base sur d'autres abstractions.

Ainsi le premier article arrive à sa conclusion. La prochaine fois que vous utiliserez `print` dans
votre code, je suis sûr que vous serez reconnaissants envers Guido van Rossum, notre "dictateur
bienveillant à vie", et l'équipe qui maintient régulièrement le langage python! Mais aussi les
pionniers derrière ces joyaux comme Ken Thompson et Dennis Ritchie.

Le véritable artiste ne cherche pas la grandeur, mais la beauté silencieuse cachée dans ce que les
autres négligent. Le véritable programmeur n’est pas différent : il trouve du sens dans les petits détails là où la plupart ne voient rien.

À la prochaine
