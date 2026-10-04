# Matériels et détection

## Objectif

Permettre à Lab-Traks de travailler avec le plus grand nombre possible de systèmes historiques et modernes sans contaminer le cœur avec leurs particularités.

Le rapport détaillé issu de l'analyse de PC Lap Counter est disponible dans [PCLC_REVERSE_ENGINEERING.md](PCLC_REVERSE_ENGINEERING.md).

## Familles rencontrées

L'analyse PCLC montre plusieurs familles très différentes :

- E/S brutes : LPT, K8000, Phidget ;
- chronomètres série : DS, RaceControl, exoTIC, etc. ;
- systèmes digitaux : Carrera, Scalextric, SCX, Ninco, oXigen, Scorpius, Davic ;
- transpondeurs : AMB/MyLaps, I-Lap, RfLapCounter, Trackmate, Robitronic, TAG Heuer, Mantis ;
- interface générique PCLC Arduino.

Cette diversité valide le besoin d'une architecture par adaptateurs.

## PCLC n'est pas notre protocole interne

Lorsqu'un système émet :

- `SF03` ;
- un octet propriétaire ;
- un numéro de transpondeur ;
- un timestamp fabricant ;

le driver le décode puis le traduit vers notre modèle.

Le Race Engine n'a pas à connaître la syntaxe d'origine.

## Compatibilité PC Lap Counter

Le protocole public PCLC Arduino est intéressant comme **façade de compatibilité**.

Il peut permettre à notre futur matériel de fonctionner avec PC Lap Counter pendant une période de transition ou pour des tests.

Ce protocole ne doit pas devenir le protocole interne de Lab-Traks.

## Futur boîtier universel

Le concept réaliste est un **concentrateur modulaire**, pas une prise magique unique.

### Interfaces candidates

- entrées digitales protégées ;
- sorties digitales/relais selon besoins ;
- RS-232 ;
- UART TTL 3,3 V / 5 V avec adaptation ;
- RS-485 ;
- USB selon architecture retenue ;
- Ethernet ;
- extensions futures.

### À éviter

Ne jamais mélanger directement sur une même entrée non protégée :

- niveaux RS-232 ;
- TTL ;
- RS-485 ;
- contacts secs ;
- alimentation capteur ;
- signaux propriétaires inconnus.

Ce serait à la fois peu fiable et potentiellement destructeur.

## Radio propriétaire

Pour oXigen, Scorpius ou d'autres systèmes radio, la première approche raisonnable est souvent de dialoguer avec le dongle/boîtier officiel plutôt que de réimplémenter immédiatement la couche RF.

Cela réduit les risques et simplifie la conformité matérielle.

## Ancien LPT

Le support LPT reste pertinent pour la compatibilité historique.

Le nouveau logiciel ne doit toutefois pas dépendre d'un vrai port LPT : le futur boîtier pourra reproduire fonctionnellement les entrées/sorties nécessaires via une interface moderne.

## Anti-rebond / anti-double-détection

Les drivers et le moteur doivent pouvoir gérer :

- durée minimale d'impulsion ;
- fronts ;
- répétitions de transpondeur ;
- délais de réarmement ;
- filtrage des événements impossibles ;
- signal brut conservé pour diagnostic.

Le filtrage exact dépend du capteur et ne doit pas être codé comme une constante universelle.

## Capture de protocoles

Lorsqu'un protocole n'est pas suffisamment documenté, on ne devine pas.

Procédure :

1. matériel réel relié à son logiciel connu ;
2. capture du transport ;
3. une action physique à la fois ;
4. annotation précise ;
5. répétition ;
6. identification du framing, checksum/CRC et champs ;
7. constitution de fixtures de test ;
8. implémentation du driver ;
9. replay automatisé.

## Priorité

Il n'est pas nécessaire de décoder tous les matériels de l'histoire avant de commencer le logiciel.

L'architecture doit permettre de les ajouter progressivement sans réécrire le chrono.

## Phidget et RFID

Les matériels Phidget font partie des technologies réellement rencontrées dans l'écosystème PCLC et les usages terrain.

Ils peuvent intervenir comme :

- cartes d'entrées/sorties ;
- lecteurs RFID ;
- interfaces auxiliaires.

Le changement de pilote en endurance est un cas important : certains utilisateurs effectuent le changement manuellement, d'autres via automatisme ou RFID.

Lab-Traks ne doit donc pas lier la notion de changement de pilote à une technologie particulière. Le driver Phidget, lorsqu'il sera étudié, devra produire des événements normalisés comme les autres adaptateurs.

Une éventuelle alternative plus innovante à la RFID n'est pas une décision. Elle devra être évaluée uniquement si elle apporte un gain réel de fiabilité, de coût ou de simplicité.

## Détection par section isolée / coupure

Une méthode historique et très répandue consiste à isoler une courte section de rail, typiquement de l'ordre de quelques centimètres, et à détecter la fermeture du circuit provoquée par les tresses de la voiture.

Cette solution est décrite comme très fiable en pratique lorsque les tresses sont en état correct.

Pour Lab-Traks, elle doit être considérée comme une source d'entrée digitale légitime à préserver, notamment via :

- ancien LPT ;
- carte I/O ;
- Phidget ;
- futur adaptateur universel.

Le logiciel ne doit pas imposer une technologie plus complexe lorsqu'une détection simple répond correctement au besoin.
