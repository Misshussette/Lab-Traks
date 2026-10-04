# Lab-Traks — Bible projet

Ce document contient les principes qui doivent survivre aux changements de conversation, de développeur et de version.

## 1. Finalité

Lab-Traks doit être un logiciel de **chronométrage et de gestion de course universel**, capable de fonctionner avec des matériels anciens comme modernes, sur différents types de courses et différentes échelles.

Le projet ne cherche pas à reproduire PC Lap Counter. PCLC sert à comprendre :

- les besoins qui existent réellement sur les pistes ;
- les matériels et protocoles historiques ;
- les cas particuliers accumulés avec le temps ;
- les fonctions qui ont prouvé leur utilité ;
- les erreurs de conception ou d'ergonomie que nous voulons éviter.

Toute fonction Lab-Traks doit être justifiée par un besoin propre au projet.

## 2. Priorités absolues

Dans cet ordre :

1. **Fiabilité**
2. **Simplicité**
3. **Universalité**
4. **Plaisir d'utilisation**

Un chrono de course n'est pas un ERP. Pendant une course, l'opérateur doit comprendre immédiatement ce qui se passe.

## 3. Indépendance matérielle

Le moteur de course ne doit pas connaître les formats bruts Carrera, DS, LPT, oXigen, Scorpius, RFID, etc.

Chaque matériel est pris en charge par un adaptateur qui :

1. reçoit les données du matériel ;
2. les valide et les décode ;
3. les traduit vers le modèle propre à Lab-Traks ;
4. expose ses capacités ;
5. conserve si nécessaire la trace brute utile au diagnostic.

Les protocoles tiers ne définissent jamais le vocabulaire interne du logiciel.

## 4. Universalité

Le modèle ne doit pas être construit autour d'une seule discipline.

Il doit pouvoir accueillir notamment :

- analogique et digital ;
- sprint ;
- endurance ;
- rallye ;
- drag race ;
- courses par tours ou par temps ;
- changements de pilote ;
- multi-voies et grands plateaux ;
- chronométrage par coupure, optique, transpondeur/RFID ou système digital ;
- différentes échelles de slot racing ;
- règles plus évoluées de type conditions météo, carburant, pneus, pénalités ou gestion de course lorsque le matériel le permet.

Une règle de course ne doit pas dépendre d'un matériel particulier. Le matériel indique uniquement ce qu'il est capable d'appliquer.

## 5. Ressources et machines anciennes

Le logiciel doit être conçu pour rester utilisable sur un PC ancien.

Conséquences :

- pas de dépendances lourdes sans justification ;
- consommation CPU faible au repos ;
- mémoire maîtrisée ;
- aucun GPU puissant requis pour le chronométrage ;
- chargement uniquement des drivers et services nécessaires ;
- moteur de chrono indépendant de la fluidité de l'interface ;
- les effets visuels ne doivent jamais pouvoir faire perdre un passage.

Les seuils matériels minimums seront fixés après prototypes et mesures, pas arbitrairement.

## 6. Fonctionnement hors ligne

Internet peut apporter des fonctions supplémentaires, mais ne doit jamais être nécessaire pour :

- installer le logiciel ;
- lancer le logiciel ;
- valider une licence éventuelle ;
- configurer une course ;
- chronométrer ;
- sauvegarder ;
- reprendre une course.

Une indisponibilité réseau ne doit pas empêcher l'événement.

## 7. Installation et mises à jour

Cible initiale : Windows.

Le logiciel doit pouvoir être distribué avec un installateur classique.

Une mise à jour doit remplacer les composants applicatifs sans détruire :

- les données ;
- les configurations ;
- les profils matériels ;
- les affichages ;
- les traductions personnalisées ;
- les courses et historiques.

## 8. Fiabilité du temps

La milliseconde est suffisante comme précision fonctionnelle cible initiale.

Lorsqu'un matériel fournit son propre temps, Lab-Traks doit pouvoir conserver :

- le timestamp ou temps fourni par le matériel ;
- le timestamp de réception côté PC ;
- la source de l'événement ;
- les informations permettant d'auditer ou rejouer le passage.

L'interface graphique ne doit jamais être la source de temps.

## 9. Résilience

L'état d'une course doit être persisté en continu de manière suffisamment robuste pour permettre une reprise cohérente après :

- crash de l'application ;
- plantage d'un driver ;
- déconnexion/reconnexion d'un matériel ;
- coupure électrique du PC.

Une anomalie matérielle doit pouvoir produire une alerte et, lorsque le matériel le permet, provoquer ou proposer une pause, une coupure de piste ou une autre mesure de sécurité de course.

## 10. Règle absolue sur les données

> **Une donnée ne doit jamais être ressaisie ou dupliquée pour être réutilisée. Si elle existe, elle est la référence pérenne jusqu'à sa suppression ou son remplacement explicite.**

Exemples :

- un pilote existe une fois ;
- un circuit existe une fois ;
- une configuration de piste existe une fois ;
- un matériel existe une fois ;
- une voiture existe une fois ;
- une traduction existe à un seul endroit de référence.

Les courses et listes utilisent des références vers ces données.

## 11. Pilotes et listes de course

Supprimer toute la base pilotes pour préparer une course est un anti-pattern.

Le système doit permettre :

- une base pilotes pérenne ;
- des sous-listes ou sélections pour une course/événement ;
- l'import d'une liste spécifique à une course ;
- la recherche de correspondances existantes avant création ;
- l'alerte sur doublons exacts ou approximatifs ;
- la mise à jour maîtrisée d'une fiche existante lorsque cela est pertinent.

## 12. Interface et affichages

L'interface doit rester accessible et rapide à comprendre.

Le système doit prévoir :

- écran opérateur ;
- affichage global ;
- affichage par piste/voie ou pilote si nécessaire ;
- écrans déportés ;
- configurations sauvegardables ;
- designer WYSIWYG simple pour les affichages ;
- split times ;
- informations live adaptées à la course.

Les affichages lisent l'état du moteur ; ils ne doivent pas recalculer la vérité de course.

## 13. Commandes pilote

Le logiciel ne doit pas supposer que le pilote dispose d'une manette complexe.

Cas courant :

- analogique : gâchette ;
- digital : gâchette + généralement un bouton de changement de voie.

Les fonctions avancées doivent donc être compatibles avec cette réalité matérielle.

## 14. Traductions

Le logiciel doit être au minimum prévu pour FR/EN et extensible.

Les traductions doivent pouvoir évoluer sans nécessiter de dupliquer le reste des données.

## 15. GitHub comme mémoire officielle

Les conversations servent à réfléchir.

Le dépôt sert à conserver :

- ce qui est décidé ;
- ce qui est connu ;
- ce qui reste à vérifier ;
- les recherches protocolaires ;
- les choix d'architecture ;
- les raisons des choix.

En cas de contradiction entre une ancienne conversation et une décision documentée plus récente dans ce dépôt, **le dépôt fait foi**.

## 16. Point de départ du développement : les données avant l'interface

La construction du produit doit commencer par l'identification des données métier réellement nécessaires et de leurs relations, avant de figer une interface graphique complète.

PC Lap Counter peut servir d'inventaire fonctionnel pour repérer les concepts utiles accumulés avec l'expérience : pilotes, circuits, championnats, catégories, matériels, courses, affichages, résultats, etc.

Cette observation ne signifie pas que Lab-Traks doit reprendre ses menus, son vocabulaire ou son architecture.

Pour chaque information manipulée, la question de référence est :

> **Le logiciel connaît-il déjà cette donnée ?**

Si oui, elle doit être réutilisée par référence et ne pas être redemandée ou recréée.

## 17. Services Web indépendants

Les futurs services Web évoqués autour de Lab-Traks ne font pas partie du cœur nécessaire au chronométrage.

Deux pistes distinctes existent :

- **StintLab** : extension Web orientée pilote, progression, chronos, voitures, circuits, exercices et statistiques. Elle doit pouvoir rester utilisable indépendamment de Lab-Traks, avec saisie/import simple ou intégration automatisée.
- **Slot Hub** : portail public permettant aux clubs de publier volontairement événements, calendriers, résultats, live timing et éventuellement flux vidéo. Il vise à mutualiser l'accès au contenu slot actuellement dispersé entre différents réseaux et plateformes.

Lab-Traks doit pouvoir faciliter l'alimentation de ces services, mais ils ne doivent jamais devenir une dépendance du chrono local.
