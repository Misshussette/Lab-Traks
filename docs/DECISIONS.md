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

Les configurations d'affichage sont des données pérennes : elles doivent pouvoir être enregistrées, rouvertes, modifiées, dupliquées, réutilisées et associées à des écrans ou usages sans être recréées.

## 2026-10-01 — Splits

Les split times font partie des fonctions importantes du futur système.

Le modèle ne doit pas réduire le chronométrage à une seule ligne départ/arrivée : il doit pouvoir représenter plusieurs points de chronométrage d'un même parcours.

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

## 2026-10-04 — Préparer les données avant l'UI

La modélisation des données métier précède la conception détaillée de l'interface.

PC Lap Counter sert d'inventaire de besoins et de cas réels, pas de modèle de données à reproduire.

## 2026-10-04 — Changement de pilote indépendant du moyen d'identification

Le changement de pilote est un événement métier distinct de sa source.

Il peut provenir d'une action opérateur, d'un automatisme, d'un lecteur RFID/Phidget ou d'un autre dispositif futur sans changer le modèle interne.

## 2026-10-04 — Services Web non obligatoires

StintLab et un éventuel Slot Hub sont des services Web indépendants du cœur de chronométrage.

Lab-Traks peut les alimenter ou les simplifier, mais son fonctionnement local ne dépend pas d'eux.

---

# Non-décisions à ne pas transformer en décisions

À cette date :

- C#/.NET n'est pas encore choisi ;
- Avalonia n'est pas encore choisi ;
- Rust n'est pas choisi ;
- SQLite n'est pas encore choisi ;
- aucune version minimale de Windows n'est encore fixée ;
- aucun modèle de microcontrôleur pour le futur boîtier n'est choisi.

## 2026-10-05 — Designer de circuit simple, bibliothèque riche

Le designer Lab-Traks doit être volontairement plus simple qu'Ultimate Racer 3.0.

La richesse doit venir principalement d'une bibliothèque de rails normalisée et extensible. Les caractéristiques géométriques sont vérifiées à partir de sources fiables et stockées dans notre propre modèle ; les représentations graphiques sont générées par Lab-Traks autant que possible.

Le tracé doit être relié au même objet Circuit utilisé par le chronométrage afin d'éviter toute ressaisie des longueurs, voies, splits et positions de capteurs.


## 2026-10-05 — Pistes bois à largeur libre

Pour les pistes bois/routed tracks, les contours intérieur et extérieur sont des géométries éditables indépendamment. Lab-Traks ne suppose pas une largeur de piste constante.

Une génération automatique peut fournir un tracé initial, mais l'utilisateur doit pouvoir élargir ou resserrer localement la surface de piste sans imposer ces modifications aux slots.


## 2026-10-05 — Slots bois indépendants

Sur une piste bois, chaque slot peut avoir sa propre géométrie. Les voies peuvent localement se resserrer, s'écarter, se déplacer, se croiser ou permuter leur position.

Le designer doit conserver une utilisation simple : création parallèle par défaut puis outils locaux de zone pour générer automatiquement des transitions douces. L'édition manuelle reste possible.

Les croisements doivent pouvoir distinguer une intersection sur le même plan d'un passage à des niveaux différents (pont/tunnel).


## 2026-10-05 — Nombre de slots et topologie du circuit

À la création d'une piste libre/bois, l'utilisateur choisit le nombre de slots (1, 2, 3, 4, 5, 6, 7, etc.) sans plafond métier arbitraire.

L'option **Slots parallèles** est activée par défaut pour simplifier la création, mais reste une aide et non une contrainte permanente.

Le modèle de circuit supporte aussi bien une **boucle fermée** qu'un **parcours ouvert**. Cela permet au même designer de représenter notamment circuit classique, rallye et dragster.

La géométrie du circuit et le format sportif restent deux concepts distincts.
