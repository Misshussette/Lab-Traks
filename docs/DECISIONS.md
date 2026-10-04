# Journal des décisions

Ce fichier consigne uniquement les décisions considérées comme acquises ou les corrections importantes de compréhension.

---

## 2026-10-01 — Windows d'abord

La première cible est Windows.

Les autres plateformes pourront être envisagées plus tard, mais elles ne doivent pas complexifier inutilement le premier produit.

## 2026-10-01 — Hors ligne obligatoire

Internet n'est pas requis pour l'installation, la licence éventuelle ou le déroulement d'une course.

Les fonctions réseau sont additionnelles.

## 2026-10-01 — Faible consommation

Le logiciel doit être conçu pour pouvoir fonctionner sur des PC anciens avec une consommation de ressources faible.

Les objectifs chiffrés seront définis après mesures de prototypes.

## 2026-10-01 — Installation classique et mises à jour non destructives

Un installateur Windows classique est souhaité.

Une mise à jour ne doit pas supprimer les données ou configurations utilisateur.

## 2026-10-01 — Matériel ancien

La compatibilité avec COM/RS-232 et les systèmes historiques est un objectif.

L'ajout de drivers ne doit pas nécessiter de modifier le cœur métier.

## 2026-10-01 — Précision

La milliseconde est suffisante comme résolution fonctionnelle cible initiale.

Les timestamps matériel et PC doivent pouvoir être conservés lorsque disponibles.

## 2026-10-01 — Résilience

La sauvegarde continue doit permettre une reprise cohérente après crash ou coupure.

## 2026-10-01 — Source unique de données

Une donnée métier ne doit pas être ressaisie ou dupliquée pour être réutilisée.

La base pilote est pérenne ; une course utilise une sélection/sous-liste plutôt qu'une nouvelle base.

## 2026-10-01 — Affichages

Des affichages multiples et configurables sont nécessaires, avec un designer WYSIWYG simple.

## 2026-10-01 — Splits

Les split times font partie des fonctions importantes du futur système.

## 2026-10-04 — PCLC est une source d'expérience, pas le produit cible

PC Lap Counter doit être analysé pour comprendre les besoins, protocoles et cas réels.

Lab-Traks ne doit pas devenir une copie 1:1 de PCLC.

## 2026-10-04 — Priorités produit

Priorités :

1. fiabilité ;
2. simplicité ;
3. universalité ;
4. plaisir d'utilisation.

## 2026-10-04 — Protocoles externes traduits vers notre vocabulaire

Le vocabulaire de PCLC ou d'un fabricant ne devient pas le vocabulaire de Lab-Traks.

Chaque driver traduit son protocole vers notre modèle interne.

## 2026-10-04 — Futur boîtier universel modulaire

Un boîtier universel doit agréger plusieurs interfaces électriques/protocolaires séparées et protégées.

Il ne doit pas reposer sur un connecteur unique mélangeant des niveaux électriques incompatibles.

## 2026-10-04 — GitHub devient la mémoire officielle

Le dépôt `Misshussette/Lab-Traks` devient la source canonique des décisions et connaissances consolidées du projet.

---

# Non-décisions à ne pas transformer en décisions

À cette date :

- C#/.NET n'est pas encore choisi ;
- Avalonia n'est pas encore choisi ;
- Rust n'est pas choisi ;
- SQLite n'est pas encore choisi ;
- aucune version minimale de Windows n'est encore fixée ;
- aucun modèle de microcontrôleur pour le futur boîtier n'est choisi.
