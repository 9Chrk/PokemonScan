# PokemonScan

![C](https://img.shields.io/badge/C-GNU11-blue?style=flat-square)
![Réseau](https://img.shields.io/badge/R%C3%A9seau-TCP-green?style=flat-square)
![Threads](https://img.shields.io/badge/Threads-POSIX-orange?style=flat-square)

PokemonScan est une **application client-serveur de recherche d’images similaires**, écrite en **C**. Indiquez le chemin d’une image BMP : le serveur la compare à une banque locale et renvoie le chemin de la meilleure correspondance ainsi que sa distance de Hamming.

La comparaison repose sur un hachage perceptif de 64 bits. Le projet utilise des sockets TCP, des threads POSIX et une bibliothèque interne de traitement d’images. La banque contient 68 images BMP, complétées par 16 images de test.

> Projet académique ULB — INFO-F201.
> Systèmes d’exploitation · Projet 2

<a id="captures-decran"></a>

## 📸 Captures d’écran

![Aperçu de la recherche d’images PokemonScan](https://github.com/user-attachments/assets/549c166a-85b5-41b0-81f0-77d85c037b4b)

---

## 📖 Sommaire

- [Fonctionnalités](#fonctionnalites)
- [Prérequis](#prerequis)
- [Configuration](#configuration)
- [Installation](#installation)
- [Lancement](#lancement)
- [Utilisation prévue](#utilisation-prevue)
- [Données d’images](#donnees-dimages)
- [Architecture](#architecture)
- [Flux général](#flux-general)
- [Structure du projet](#structure-du-projet)
- [Tests](#tests)
- [Problèmes fréquents](#problemes-frequents)
- [Documentation](#documentation)
- [Licence](#licence)

<a id="fonctionnalites"></a>

## ✨ Fonctionnalités

- **Recherche par similarité visuelle** : le serveur parcourt les fichiers de `img/`, calcule leur hachage perceptif et sélectionne celui dont la distance de Hamming au hachage de l’image demandée est minimale.
- **Échanges client-serveur TCP** : le client se connecte au port `5555` et envoie un chemin d’image ; le serveur répond avec une phrase contenant le chemin retenu et la distance obtenue.
- **Client interactif** : `client/pokedex-client.c` lit les chemins saisis sur l’entrée standard tout en réservant un second thread à la lecture des réponses du serveur.
- **Prise en charge BMP** : la bibliothèque accepte les BMP non compressés de 24 ou 32 bits par pixel, les charge en mémoire puis les convertit en une représentation exploitée par le calcul de pHash.
- **Jeu de tests fourni** : `test/tests` lit les cas de `test/test-new-images.data` et compare les réponses du programme aux chemins et distances attendus.

<a id="prerequis"></a>

## 🧰 Prérequis

- Un environnement POSIX disposant de `bash`.
- Un compilateur C compatible avec GNU C11, tel que `gcc`.
- `make`, utilisé par les deux Makefiles.
- Les bibliothèques système de mathématiques et de threads, liées avec `-lm` et `-lpthread` côté serveur.

<a id="configuration"></a>

## ⚙️ Configuration

Aucun fichier de configuration ni variable d’environnement n’est présent. Les paramètres d’exécution sont définis dans les sources :

- le port TCP est `5555` dans `client/pokedex-client.c` et `serveur/img-search.c` ;
- le client utilise `127.0.0.1` par défaut et accepte une adresse IPv4 en premier argument ;
- le serveur ouvre le répertoire relatif `img/` au démarrage. Il doit donc être lancé depuis la racine du dépôt pour trouver la banque d’images.

<a id="installation"></a>

## 📦 Installation

La bibliothèque statique de traitement d’images peut être construite depuis la racine :

```bash
make -C img-dist
```

Cette commande produit `img-dist/libimg-dist.a` à partir de `bmp.c`, `pHash.c` et `verbose.c`.

Les exécutables ne peuvent toutefois pas être construits avec l’état versionné du dépôt : le `Makefile` racine attend `serveur/main.c` et `client/main.c`, alors que les fichiers présents sont respectivement `serveur/img-search.c` et `client/pokedex-client.c`. De plus, l’appel à `recv` dans `serveur/img-search.c` ne fournit que trois arguments, alors que l’API des sockets en exige quatre. Aucune commande de compilation des exécutables n’est donc indiquée comme opérationnelle.

<a id="lancement"></a>

## ▶️ Lancement

Le lancement complet n’est pas disponible tant que les blocages de compilation ci-dessus ne sont pas corrigés. Les points d’entrée présents dans les sources montrent néanmoins l’utilisation prévue : le serveur écoute sur le port `5555`, puis le client s’y connecte sur `127.0.0.1` ou l’adresse IPv4 transmise en argument.

Après correction et compilation des exécutables, le serveur doit être lancé depuis la racine afin que son chemin relatif `img/` résolve la banque locale. Le client est alors destiné à se connecter au serveur avant toute recherche.

<a id="utilisation-prevue"></a>

## 🎮 Utilisation prévue

Dans le client, l’utilisateur saisit le chemin relatif d’un BMP, par exemple `test/img/6-1.bmp`. Le client transmet cette chaîne au serveur ; il n’envoie pas le contenu binaire du fichier. Le fichier demandé doit donc être accessible depuis le répertoire de travail du serveur.

Le serveur compare cette image à la banque `img/` et renvoie une réponse de la forme :

```text
Most similar image found: 'img/6.bmp' with a distance of 0.
```

La valeur exacte dépend de l’image envoyée. Le client traite `SIGINT` et `SIGPIPE` pour fermer son socket lors d’une interruption.

<a id="donnees-dimages"></a>

## 🗃️ Données d’images

| Emplacement | Contenu | Rôle |
| --- | --- | --- |
| `img/` | 68 fichiers BMP | Banque parcourue par le serveur pour trouver la correspondance la plus proche. |
| `test/img/` | 16 fichiers BMP | Images utilisées pour évaluer la recherche. |
| `test/test-new-images.data` | Données texte | Associe chaque image de test à une image attendue de la banque et à une distance de Hamming attendue. |

La lecture BMP dans `img-dist/bmp.c` vérifie l’en-tête, accepte 24 ou 32 bits par pixel et refuse les autres profondeurs. Les données sont traitées en mémoire ; aucune base de données ni service distant n’est utilisé.

<a id="architecture"></a>

## 🧱 Architecture

Les deux programmes possèdent chacun leur point d’entrée : `main` de `serveur/img-search.c` initialise le socket d’écoute et indexe `img/`, tandis que `main` de `client/pokedex-client.c` ouvre une connexion et crée les threads d’envoi et de réception. Les appels système susceptibles d’échouer sont centralisés derrière les macros `checked` et `checked_wr` de `commun/commun.h`.

La bibliothèque `img-dist/` sépare le chargement BMP du calcul de similarité. `bmp.c` lit le fichier et représente ses pixels dans `RgbImage`. `pHash.c` redimensionne l’image à 32 × 32, la convertit en niveaux de gris par moyenne pondérée des composantes RGB, applique une transformée en cosinus discrète bidimensionnelle, puis compare les 64 premiers coefficients à leur moyenne afin de former un hachage de 64 bits. `DistancePHash` compte ensuite les bits différents entre deux hachages.

Le serveur stocke les chemins relevés dans `img/` et les données de la recherche en cours dans la structure globale `SharedMemory`. Sa fonction `process` compare les images d’une plage d’indices et protège la mise à jour du meilleur résultat par un mutex. Trois threads sont créés pour les trois plages, mais chaque thread est immédiatement joint avant la création du suivant : le découpage est présent dans le code, sans exécution concurrente effective entre ces trois traitements.

Le client maintient un thread qui lit `stdin` et écrit les chemins sur le socket, et un autre qui lit les réponses. Un mutex protège principalement l’affichage et l’accès à l’envoi. La communication ne comporte pas de couche de persistance : les chemins et résultats restent en mémoire durant l’exécution.

<a id="flux-general"></a>

## 🧬 Flux général

```text
Client : chemin BMP saisi
  -> socket TCP vers le port 5555
  -> serveur : lecture du chemin et parcours de img/
  -> img-dist : chargement BMP -> réduction 32 × 32 -> DCT -> pHash 64 bits
  -> DistancePHash avec chaque image de la banque
  -> meilleur chemin et distance
  -> réponse texte envoyée au client
```

<a id="structure-du-projet"></a>

## 📂 Structure du projet

```text
.
├── client/
│   └── pokedex-client.c     # Point d’entrée du client TCP interactif
├── commun/
│   └── commun.h             # Vérification des retours d’appels système
├── img/                     # Banque locale d’images BMP recherchée par le serveur
├── img-dist/
│   ├── bmp.c, bmp.h         # Lecture et représentation des BMP
│   ├── pHash.c, pHash.h     # pHash, DCT et distance de Hamming
│   ├── verbose.c, verbose.h # Option d’affichage de diagnostic
│   └── Makefile             # Construction de libimg-dist.a
├── serveur/
│   ├── img-search.c         # Point d’entrée du serveur et recherche
│   └── imgdist.h            # Interface exposée à la bibliothèque d’images
├── test/
│   ├── img/                 # Images de test
│   ├── test-new-images.data # Résultats attendus
│   └── tests                # Script de vérification Bash
├── Makefile                 # Règles de construction racine
├── Projet_2_OS.pdf          # Document PDF associé au projet
└── LICENSE                  # Licence MIT
```

<a id="tests"></a>

## 🧪 Tests

Le script Bash `test/tests` lance un serveur en arrière-plan, attend dix secondes, exécute des recherches individuelles puis une série de recherches via un même client, et compare les lignes reçues avec `test/test-new-images.data`.

Il nécessite les exécutables `img-search` et `pokedex-client`, qui ne sont pas produits par le Makefile versionné. Il ne doit donc pas être exécuté dans l’état actuel. Lorsqu’il est utilisable, il appelle `killall img-search` avant et après les tests ; cette action termine les processus portant ce nom sur la machine courante.

<a id="problemes-frequents"></a>

## ❗ Problèmes fréquents

### Les exécutables ne se compilent pas

Le `Makefile` racine référence deux fichiers absents (`serveur/main.c` et `client/main.c`). Le serveur contient aussi un appel `recv` incomplet. Ces problèmes doivent être corrigés avant de produire `img-search` et `pokedex-client`.

### Le serveur ne trouve pas `img/`

Le répertoire est ouvert avec le chemin relatif `img/`. Lancer le serveur depuis un autre dossier empêche donc l’accès à la banque.

### Un BMP est refusé

La bibliothèque ne prend en charge que les BMP de 24 ou 32 bits par pixel. Une image à une autre profondeur déclenche un message d’erreur lors de son chargement.

### Les tests arrêtent un autre serveur

Le script `test/tests` utilise `killall img-search`. Vérifier qu’aucun autre processus important ne porte ce nom avant de l’exécuter.

### Un résultat de test vise une image absente

Le premier cas de `test/test-new-images.data` attend `img/81.bmp`, mais cette image n’est pas présente dans `img/`. Ce cas ne peut donc pas être validé contre la banque versionnée, même après correction de la compilation.

<a id="documentation"></a>

## 📄 Documentation

Le document [Projet_2_OS.pdf](Projet_2_OS.pdf) est fourni à la racine du dépôt.

<a id="licence"></a>

## 📜 Licence

Ce projet est distribué sous licence [MIT](LICENSE).
