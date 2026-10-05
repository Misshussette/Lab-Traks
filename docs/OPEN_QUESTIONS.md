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

## Horodatage effectif

Lorsque le matériel fournit son propre temps et que le PC possède aussi un timestamp de réception, les deux doivent pouvoir être conservés.

Reste à définir précisément :

- quelle source devient le temps sportif de référence selon chaque protocole ;
- quels contrôles permettent de détecter une dérive ou une incohérence ;
- quels éléments, s'il y en a, doivent être exposés à l'utilisateur normal.

L'objectif UX est d'éviter de demander à l'opérateur un choix technique inutile si le driver peut déterminer automatiquement la meilleure source.

## Traductions personnalisables et contribution communautaire

Le principe d'une traduction modifiable localement est retenu comme besoin, mais le workflow détaillé reste à concevoir.

À décider notamment :

- format des packs de langue ;
- traçage des champs modifiés localement ;
- comparaison lors d'une mise à jour du pack officiel ;
- choix « garder ma version / prendre l'officielle » uniquement sur les conflits ;
- publication volontaire d'une correction vers une plateforme communautaire ;
- modération et contrôles automatiques des contributions.

## Affichage déporté et vues

Les besoins confirmés comprennent au minimum l'affichage global et l'affichage filtré par piste.

L'idée d'un affichage spécifiquement orienté pilote/équipe est prometteuse pour de petits écrans au poste de pilotage, mais elle reste une piste fonctionnelle à valider et ne doit pas être considérée comme une exigence acquise.

## Phidget / RFID et changements de pilote

Phidget est déjà rencontré dans les usages réels, notamment avec des cartes I/O et des lecteurs RFID.

À documenter avec des cas terrain avant toute conception spécifique :

- modèles réellement utilisés ;
- bibliothèque/API employée ;
- séquence actuelle d'un changement de pilote en endurance ;
- règles d'automatisme existantes ;
- cas d'erreur ou de badge non lu ;
- avantages/inconvénients par rapport à une saisie opérateur.

Aucune solution de remplacement de la RFID n'est décidée à ce stade.

## StintLab

Le nom, le périmètre exact et le modèle de service restent ouverts.

Principe à conserver : il s'agit d'une extension Web indépendante, pouvant être alimentée automatiquement par Lab-Traks mais également utilisable sans lui.

## Slot Hub

Le concept est une piste produit distincte : portail public de mutualisation des événements, calendriers, résultats, live timing et éventuellement flux vidéo des clubs.

À définir plus tard :

- gouvernance ;
- hébergement ;
- format d'échange ;
- publication volontaire et permissions ;
- intégration ou simple référencement des flux vidéo ;
- accès des clubs n'utilisant pas Lab-Traks.


## Barèmes et calcul des championnats

Le moteur devra pouvoir gérer des classements de championnat et des attributions de points sur différents niveaux, y compris des segments.

À étudier :

- barème selon la position ;
- calcul lié aux tours ou à la performance ;
- barèmes dégressifs générés automatiquement ;
- formules personnalisées ;
- bonus éventuels ;
- points attribués à une course entière ou à certains segments ;
- génération intelligente d'un barème complet selon le nombre de participants, sans saisie case par case ;
- possibilité de ne rien automatiser et de laisser l'organisateur définir librement son système.

Les propositions ne doivent pas dépendre arbitrairement d'une échelle de slot. PCLC pourra servir à inventorier des méthodes existantes, mais Lab-Traks ne doit pas prétendre qu'un barème est universel sans preuve terrain.

## Designer de circuit

Un futur designer de circuit est envisagé. Il est distinct du champ constructeur/fabricant du circuit.

UR30 sera étudié ultérieurement comme source d'expérience fonctionnelle afin d'identifier les besoins utiles sans recopier son architecture.


## Points de mesure et intermédiaires

À prototyper/comparer :

- optocoupleur sous slot déclenché par la lame-guide ;
- autres barrières IR modulées ;
- détection électrique/dead strip selon type de piste ;
- nœud 1 à 10 voies ;
- variante 2 capteurs par voie pour vitesse moyenne A-B ;
- précision réelle de l'horodatage ;
- synchronisation de plusieurs nœuds ;
- CAN vs RS-485 pour le bus de terrain ;
- alimentation des nœuds ;
- caméra 2D global-shutter multi-voies : vérifier l'ambiguïté de déclenchement dans une zone ;
- line-scan/photo-finish multi-voies : candidat à tester pour supprimer cette ambiguïté ;
- précision/timestamps/charge CPU d'une solution vision ;
- radar comme option de mesure de vitesse locale et non comme dépendance ;
- identification des voitures sur systèmes digitaux.
