# Questions ouvertes

Les sujets de ce fichier ne sont **pas** des décisions.

## Stack desktop

À comparer selon des mesures et prototypes :

- C#/.NET ;
- Avalonia UI ;
- solutions Windows natives ;
- Rust pour tout ou partie du système ;
- autres options réellement pertinentes.

Critères :

- consommation RAM/CPU ;
- démarrage ;
- compatibilité vieux PC ;
- accès série/USB et drivers ;
- robustesse ;
- facilité de maintenance ;
- qualité UI ;
- packaging Windows ;
- possibilité de porter plus tard sans pénaliser Windows aujourd'hui.

## Persistance

À choisir après définition du modèle :

- format de stockage local ;
- journal d'événements ;
- snapshots ;
- stratégie anti-corruption ;
- reprise après crash.

SQLite est une piste, pas encore une décision.

## Windows minimum

Déterminer la plus vieille version de Windows réellement utile à supporter.

La décision doit être basée sur les machines encore utilisées dans les clubs et sur les contraintes du runtime retenu.

## Budget de ressources

Mesurer puis fixer :

- RAM cible au repos ;
- RAM en course ;
- CPU au repos ;
- temps de démarrage ;
- charge avec plusieurs écrans ;
- comportement sans GPU dédié.

## MVP sportif

Définir le premier format de course suffisamment complet pour valider le moteur sans essayer de tout implémenter.

## Premiers drivers réels

Choisir l'ordre d'implémentation selon :

- matériel réellement disponible pour test ;
- documentation ;
- intérêt terrain ;
- couverture des différentes familles de protocoles.

## Boîtier universel

À décider :

- microcontrôleur ;
- isolation ;
- alimentation ;
- modules physiques ;
- connectique ;
- USB device/host ;
- Ethernet ;
- extension ;
- firmware et protocole PC ;
- mise à jour firmware.

## Licence et distribution

Le logiciel ne doit pas dépendre d'Internet pour fonctionner.

Le modèle économique/licence éventuel reste à définir.

## Nom et identité

`Lab-Traks` est actuellement le nom du dépôt/projet. Le nom produit final peut encore évoluer.
