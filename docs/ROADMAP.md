# Roadmap de travail

Cette roadmap décrit un ordre logique, pas des dates de livraison.

## Phase 0 — Consolider la connaissance

- maintenir la bible projet ;
- archiver l'analyse des systèmes existants ;
- documenter les décisions ;
- distinguer clairement faits, hypothèses et questions ouvertes.

**État : en cours.**

## Phase 1 — Modèle de données

Avant de coder l'UI complète :

- pilotes ;
- circuits ;
- véhicules ;
- catégories ;
- événements/courses/sessions ;
- inscriptions ;
- passages/tours/points de chronométrage/splits ;
- matériel ;
- configurations ;
- affichages ;
- traductions et personnalisations locales ;
- listes/sélections de participants.

Valider la règle de source unique sur tout le modèle.

## Phase 2 — Contrat du moteur

Définir :

- modèle d'événements Lab-Traks ;
- API des adaptateurs ;
- capacités matérielles ;
- horodatage ;
- ordre des événements ;
- persistance ;
- replay.

Créer un **adaptateur simulateur** pour tester sans matériel.

## Phase 3 — Race Engine minimal fiable

Implémenter un premier flux complet :

- préparation ;
- départ ;
- passages ;
- tours ;
- classement ;
- pause/reprise ;
- fin ;
- sauvegarde continue ;
- reprise après crash simulé.

Aucune dépendance à l'UI ou à un constructeur.

## Phase 4 — Premier matériel réel

Implémenter au moins :

- une source E/S simple ;
- un protocole série ;
- un système fournissant l'identité véhicule/transpondeur.

Le protocole PCLC Arduino est un bon candidat de compatibilité et de banc de test, sans devenir le modèle interne.

## Phase 5 — Interface opérateur

Construire une UI simple et rapide :

- configuration ;
- préparation de course ;
- écran live ;
- incidents ;
- résultats.

Mesurer en parallèle RAM, CPU et démarrage sur machine modeste.

## Phase 6 — Affichages

- designer WYSIWYG ;
- écran principal ;
- écran zoom/pilote ;
- écrans déportés ;
- profils sauvegardés ;
- résolution indépendante.

## Phase 7 — Bibliothèque de drivers

Ajouter progressivement les systèmes selon matériel disponible et captures validées.

Chaque driver doit avoir :

- documentation ;
- fixtures ;
- replay ;
- tests ;
- statut de fiabilité.

## Phase 8 — Prototype du boîtier universel

Seulement après stabilisation du contrat logiciel :

- architecture électrique ;
- entrées protégées ;
- RS-232 ;
- TTL ;
- RS-485 ;
- USB/Ethernet selon besoin ;
- firmware ;
- protocole Lab-Traks.

## Phase 9 — Fonctions sportives avancées

Après le chrono fiable :

- formats avancés ;
- stratégie ;
- météo ;
- carburant ;
- pneus ;
- pénalités ;
- règles personnalisées ;
- live timing ;
- autres services optionnels.

## Phase 1 bis — Inventaire fonctionnel PCLC

En parallèle de la modélisation, parcourir les menus et fonctions de PC Lap Counter comme inventaire de besoins réels.

Pour chaque fonction rencontrée :

1. identifier les données manipulées ;
2. vérifier si elles existent déjà dans notre modèle ;
3. distinguer lecture, génération et modification ;
4. conserver uniquement le besoin utile ;
5. ne jamais reprendre automatiquement le vocabulaire ou la structure PCLC.

## Phase future — Services Web indépendants

Après stabilisation du cœur local, étudier séparément :

- **StintLab**, extension Web orientée pilote/progression, compatible si possible avec d'autres sources que Lab-Traks ;
- **Slot Hub**, portail de publication volontaire pour événements, calendriers, résultats, live timing et médias.

Ces services ne doivent jamais être nécessaires au déroulement local d'une course.
