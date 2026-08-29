# PokemonScan

Application client-serveur en C qui recherche, dans une banque locale d'images BMP, celle dont le hachage perceptif est le plus proche d'une image demandée. Le serveur renvoie le chemin de l'image retenue et la distance de Hamming calculée.

## Sommaire

- [Fonctionnalités](#fonctionnalités)
- [Prérequis](#prérequis)
- [Compilation](#compilation)
- [Utilisation prévue](#utilisation-prévue)
- [Tests](#tests)
- [Structure du dépôt](#structure-du-dépôt)
- [Document et licence](#document-et-licence)

## Fonctionnalités

- serveur TCP à l'écoute sur le port `5555` ;
- client se connectant par défaut à `127.0.0.1`, ou à l'adresse IPv4 fournie en argument ;
- parcours des images présentes dans `img/` ;
- calcul d'un hachage perceptif (pHash) et de la distance de Hamming entre deux hachages ;
- traitement de la recherche réparti sur trois threads côté serveur ;
- jeu d'images et script de test fournis dans `test/`.

## Prérequis

- un environnement POSIX avec `bash` ;
- un compilateur C compatible GNU C11, tel que `gcc` ;
- `make` pour compiler la bibliothèque `img-dist` via son Makefile ;
- les bibliothèques système de mathématiques et de threads, accessibles avec `-lm` et `-lpthread`.

Les images prises en charge par la bibliothèque sont des BMP de 24 ou 32 bits par pixel.

## Compilation

La bibliothèque `img-dist` peut être compilée depuis la racine :

```bash
make -C img-dist
```

En revanche, les exécutables client et serveur ne peuvent pas être compilés dans l'état actuellement versionné :

- le Makefile racine référence `serveur/main.c` et `client/main.c`, deux fichiers absents du dépôt ;
- `serveur/img-search.c` appelle `recv` avec trois arguments, alors que l'API des sockets en requiert quatre.

La correction de ces sources est nécessaire avant de pouvoir construire et lancer l'application. Aucune commande de compilation des exécutables n'est donc fournie ici afin de ne pas suggérer un résultat inexistant.

## Utilisation prévue

Après correction et compilation des exécutables, le serveur est conçu pour être lancé depuis la racine afin d'accéder à `img/`, tandis que le client se connecte par défaut à `127.0.0.1` (ou à l'adresse IPv4 fournie en argument).

Le client demande un chemin d'image BMP et transmet ce chemin au serveur. Le fichier doit donc être accessible depuis le répertoire de travail du serveur. Les images de test, telles que `test/img/6-1.bmp`, permettent d'exercer cette recherche contre la banque `img/`.

## Tests

Le script `test/tests` contient des cas définis dans `test/test-new-images.data`, mais il nécessite les exécutables `img-search` et `pokedex-client`. Il ne peut donc pas être exécuté avec les sources actuelles tant que le blocage de compilation n'est pas résolu.

Lorsqu'il est exécuté, ce script lance un serveur et termine les processus nommés `img-search` avant et après les tests.

## Structure du dépôt

```text
.
├── client/
│   └── pokedex-client.c   # Client TCP interactif
├── commun/
│   └── commun.h           # Fonctions de vérification des appels système
├── img/                   # Banque locale d'images BMP
├── img-dist/              # Lecture BMP, pHash et distance de Hamming
├── serveur/
│   ├── img-search.c       # Serveur de recherche d'images
│   └── imgdist.h          # Interface de la bibliothèque d'images
├── test/                  # Images et script de test
├── Makefile               # Règles de compilation racine
└── Projet_2_OS.pdf        # Document PDF fourni
```

## Document et licence

Le document [Projet_2_OS.pdf](Projet_2_OS.pdf) est fourni dans le dépôt.

Ce projet est distribué sous licence [MIT](LICENSE).

## Auteurs

- paug0002
- jche0027
- rrab0007
