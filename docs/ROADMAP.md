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
- passages/tours/splits ;
- matériel ;
- configurations ;
- affichages.

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
