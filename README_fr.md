STUdio - Story Teller Unleashed
===============================

[This README in english](README.md)

Un ensemble d'outils pour lire, créer et transférer des packs d'histoires de et vers la Fabrique à Histoires Lunii\*.


PRÉAMBULE
---------

Ce logiciel s'appuie sur mes propres recherches de rétro ingénierie, limitées à la collecte des informations nécessaires à l'intéropérabilité avec la Fabrique à Histoires Lunii\*, et ne distribue aucun contenu protégé.

\* Lunii et "ma fabrique à histoires" sont des marques enregistrées de Lunii SAS. Je ne suis (et ce travail n'est) en aucun cas affilié à Lunii SAS.


UTILISATION
-----------

TODO


MÉMOIRE DE LA FABRIQUE À HISTOIRES\*
------------------------------------

La Fabrique à Histoires\* expose deux mémoires de stockage : la carte SD et une mémoire accessible par SPI (probablement une mémoire flash ?).

Ces deux espaces de stockage sont divisés en secteurs de 512 octets. Les données sont désignées par leur numéro de secteur.

### Carte SD

| Secteurs       | Contenu                                            |
|----------------|----------------------------------------------------|
| 1              | UUID                                               |
| 2              | ???                                                |
| 3              | Erreur, version firmware et taille de la carte SD  |
| 4 - 454        | Image Logo                                         |
| 455 - 905      | Image USB                                          |
| 906 - 1356     | Image batterie faible                              |
| 1357 - 1807    | Image Erreur                                       |
| 1808 - 74999   | ???                                                |
| 75000 - 99999  | Statistiques des packs (quoi que cela signifie ?)  |
| 100000         | Catalogue des packs d'histoires                    |
| 100001 - FIN   | Packs d'histoires                                  |

# SPI

Pas encore analysée.


FORMAT DE FICHIER PACK D'HISTOIRES
----------------------------------

Un pack d'histoires est composé de nœuds. Il y a deux types de nœuds :
* Les nœuds de scène (ou nœuds d'étape, "stage node") affichent une image et jouent un son, les deux étant optionels. En plus de ces ressources, ces nœudd définissent :
  * L'action/transition exécutée lorsque le bouton OK est appuyé
  * L'action/transition exécutée lorsque le bouton MAISON est appuyé
  * Quels contrôles sont disponibles
* Les nœuds d'action sont utilisés comme transition d'une scène à la suivante. Les nœuds d'action peuvent contenir :
  * Une unique scène qui est automatiquement jouée, ou
  * Plusieurs options parmi lesquelles l'utilisateur peut faire un choix, en naviguant entre les options/scènes avec la molette. Chaque fois que l'utilisateur tourne la molette, l'option/scène précédente ou suivante est jouée.


Un pack d'histoires est également divisé en secteurs de 512 octets. Les secteurs incomplets sont complétés par des zéros.

Exemple de structure d'un pack avec 4 nœuds de scène et 2 nœuds d'action :

| Secteurs       | Contenu                |
|----------------|------------------------|
| 1              | Metadonnées            |
| 2 - 5          | Nœuds de scène         |
| 6 - 7          | Nœuds d'action         |
| 8 - 458        | Ressource image 1      |
| 459 - 909      | Ressource image 2      |
| 910 - 9999     | Ressource audio 1      |
| 10000 - 19999  | Ressource audio 2      |
| 20000 - 29999  | Ressource audio 3      |
| 30000 - 39999  | Ressource audio 4      |
| 40000          | Octets de vérification |

### Secteur métadonnées

| Octets         | Donnée                    | Type   | Valeur        |
|----------------|---------------------------|--------|---------------|
| 1 - 2          | Nombre de nœuds de scène  | short  |               |
| 3              | Désactivé                 | byte   | O or 1        |
| 4 - 5          | Version                   | short  | 1 par défaut  |

### Sector nœud de scène

| Octets         | Donnée                           | Type     | Valeur                                            |
|----------------|----------------------------------|----------|---------------------------------------------------|
| 1 - 16         | UUID                             | long * 2 | Bits de poids fort en premier                     |
| 17 - 20        | Secteur de début d'image         | int      | Offset par rapport au premier nœud de scène ou -1 |
| 21 - 24        | Taille de l'image                | int      | Nombre de secteurs                                |
| 25 - 28        | Secteur de début de son          | int      | Offset par rapport au premier nœud de scène ou -1 |
| 29 - 32        | Taille du son                    | int      | Nombre de secteurs                                |
| 33 - 34        | Action si OK appuyé              | short    | Offset par rapport au premier nœud de scène ou -1 |
| 35 - 36        | Options dans la transition       | short    | Nombre d'options disponibles                      |
| 37 - 38        | Option choisie                   | short    | Indice de l'option sélectionnée                   |
| 39 - 40        | Action si MAISON apputyé         | short    | Offset par rapport au premier nœud de scène ou -1 |
| 41 - 42        | Options dans la transition       | short    | Nombre d'options disponibles                      |
| 43 - 44        | Option choisie                   | short    | Indice de l'option sélectionnée                   |
| 45 - 46        | Molette autorisée                | short    | 0 or 1                                            |
| 47 - 48        | OK autorisé                      | short    | 0 or 1                                            |
| 49 - 50        | MAISON autorisé                  | short    | 0 or 1                                            |
| 51 - 52        | PAUSE autorisé                   | short    | 0 or 1                                            |
| 53 - 54        | Avancement auto à la fin du son  | short    | 0 or 1                                            |

### Secteur nœud d'action

Un nœud d'action est simplement une liste d'options disponibles. Chaque option est définie par un champ de type short, représentant l'offset (par rapport au premier nœud de scène) du nœud de scène.

### Secteur ressource image

Les ressources d'image sont des fichiers Windows BMP, 24-bits, de 320x240. Les images peuvent être en couleurs, bien que
certains couleurs ne seront certainement pas affichées fidèlement par l'écran situé derrière le boîtier en plastique.
Si le dernier secteur est incomplet, il est complété par des zéros.

### Secteur ressource audio

Les ressources audio sont des fichiers WAVE, 16-bits signés, 32 000 Hz. Si le dernier secteur est incomplet, il est complété par des zéros.

### Secteur octets de vérification

Le dernier secteur d'un fichier de pack d'histoires doit contenir une séquence prédéfinie de 512 octets.


PILOTE FABRIQUE À HISTOIRES LUNII\*
-----------------------------------

Le transfert de packs d'histoires de et vers la Fabrique à Histoire\* est géré par le pilote Lunii\* officiel. Ce pilote
est distribué avec le logiciel Luniistore\*, et doit y être récupére :

TODO Instructions pour récupérer le pilote


LICENCE
-------

Ce projet est distribué sous la licence Mozilla Public License 2.0.