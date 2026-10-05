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


## Piste de recherche — points de mesure distribués

> Statut : **architecture candidate à prototyper, pas décision matérielle définitive**.

Pour rendre les intermédiaires accessibles financièrement et simples à installer, une piste prometteuse consiste à créer un **nœud de mesure local** installé près du point de détection.

### Objectif

Un même type de nœud pourrait couvrir de 1 à 10 voies.

Il reçoit des capteurs simples et horodate localement leurs fronts avec un timer matériel, puis transmet les événements au système Lab-Traks.

Exemples d'usage du même nœud :

- départ/arrivée ;
- intermédiaire ;
- secteur ;
- PIT IN/PIT OUT ;
- point de vitesse ;
- rallye ;
- drag.

### Capteurs optiques sous le slot

Pour les pistes bois, une solution particulièrement intéressante est l'optocoupleur placé sous le slot : la lame-guide de la voiture coupe directement le faisceau infrarouge.

Avantages à valider par prototype :

- aucune électronique dans la voiture ;
- pas de portique au-dessus de la piste ;
- indépendant de la couleur de carrosserie ;
- fonctionne dans l'obscurité ;
- coût potentiellement faible ;
- installation répétable ;
- détection directement associée au slot concerné.

Cette technique existe déjà dans des systèmes de slot commerciaux et amateurs.

Les capteurs IR modulés rapides constituent une autre variante lorsque la géométrie impose une barrière séparée.

### Mesure de vitesse moyenne entre deux points

Deux capteurs A et B installés à une distance précisément connue permettent de mesurer une **vitesse moyenne sur le segment A-B**. Cette valeur ne doit pas être présentée comme une vitesse instantanée.

```text
---- A -------- distance connue -------- B ---->
        voiture
```

Le nœud horodate les deux passages avec la même horloge.

```text
vitesse moyenne A-B = distance(A,B) / (timestamp B - timestamp A)
```

Avec 10 voies, un nœud destiné à la vitesse peut donc nécessiter jusqu'à 20 entrées numériques.

La distance A-B doit être configurable et adaptée à la vitesse et à l'installation. La précision réelle devra être mesurée sur prototype plutôt que supposée.

### Horodatage

Le PC ne doit pas être chargé de détecter précisément le front du capteur.

Le nœud local doit capturer l'instant au plus près du signal, idéalement par timer/input-capture matériel.

Pour comparer des temps issus de nœuds différents, il faudra définir une stratégie de synchronisation d'horloge ou une autre méthode garantissant la précision cible.

### Bus entre les points de mesure

Un circuit peut avoir plusieurs points éloignés : départ, plusieurs intermédiaires, stands, arrivée.

Une liaison filaire différentielle multipoint est une piste logique pour relier les nœuds et éviter un câble USB vers chaque point.

**CAN et RS-485 sont des candidats à comparer. Aucun n'est choisi à ce stade.**

Critères :

- fiabilité ;
- coût ;
- câblage simple ;
- alimentation possible des nœuds le long du circuit ;
- plusieurs nœuds ;
- événements quasi simultanés ;
- diagnostic ;
- synchronisation ;
- facilité de fabrication et de maintenance.

### Radar

Le radar reste une technologie à étudier, notamment pour des usages spécifiques de mesure de vitesse.

Il ne doit cependant pas être considéré comme nécessaire au système de base.

Pour plusieurs voies très proches, les principaux points à valider sont :

- séparation spatiale des véhicules ;
- largeur du champ de détection ;
- identification de la voie/voiture ;
- mouvements parasites ;
- coût ;
- complexité du traitement.

Un radar simple peut mesurer mouvement, direction et vitesse mais n'offre pas nécessairement une résolution angulaire suffisante pour distinguer plusieurs slots rapprochés.

### Analogique, rallye et digital

En analogique, le slot/voie permet généralement d'associer directement le passage au concurrent de la manche.

En rallye avec un concurrent à la fois, l'association est également simple.

En digital, plusieurs voitures peuvent utiliser la même voie physique : un capteur optique de passage ne fournit alors pas nécessairement l'identité de la voiture. Lab-Traks doit pouvoir compléter le point de mesure par une technologie d'identification (protocole digital, transpondeur ou autre) lorsque le cas d'usage l'exige.

Le **passage** et l'**identité** restent donc deux capacités distinctes.


## Piste de recherche — vision pour les intermédiaires

> Statut : **candidat à prototyper**, pas solution de chronométrage officiel validée.

Le principal coût/complexité d'un intermédiaire multi-voies n'est pas nécessairement l'électronique du capteur mais son installation : un capteur par voie, alignement, pont éventuel, câblage et adaptation aux entraxes.

Une caméra **global shutter** rapide constitue donc une piste différente : déplacer une partie de la complexité du matériel vers le logiciel.

Un seul module placé au-dessus ou à proximité du point de mesure pourrait observer plusieurs voies et utiliser des lignes virtuelles configurées dans Lab-Traks.

Avantages potentiels :

- 1 à 10 voies observées par un seul dispositif ;
- peu de câblage ;
- pas de capteur physique à aligner sur chaque slot ;
- adaptation logicielle à l'entraxe ;
- intermédiaires virtuels ;
- possibilité d'estimer trajectoire, ordre de passage et vitesse locale ;
- installation potentiellement temporaire.

Points à valider impérativement :

- précision réelle de l'horodatage image ;
- fréquence d'image nécessaire ;
- exposition et flou à haute vitesse ;
- éclairage ;
- occultation de plusieurs voitures ;
- champ de vision pour 10 voies ;
- charge CPU sur un vieux PC ;
- comportement lorsque plusieurs voitures franchissent simultanément la ligne ;
- calibration géométrique ;
- précision réellement obtenue par rapport à un capteur matériel.

Des caméras global-shutter USB abordables existent aujourd'hui à 120/240 images/s et certains modules annoncent des fréquences plus élevées en réduisant fortement la zone/résolution. Cela justifie un prototype mais **ne suffit pas à garantir la milliseconde**.

Une solution vision pourrait être acceptable pour des intermédiaires/statistiques même si elle n'atteint pas le niveau de confiance requis pour la ligne officielle départ/arrivée.

## Vitesse au point

Il faut distinguer explicitement :

- **vitesse moyenne A-B** : calculée sur une distance connue entre deux passages ;
- **vitesse locale estimée par vision** : dérivée d'un déplacement observé sur une fenêtre temporelle ;
- **vitesse Doppler/radar** : mesure de vitesse radiale par rapport au capteur.

Aucune ne doit être appelée abusivement « vitesse instantanée » sans préciser la méthode.

Le radar Doppler est particulièrement intéressant pour une mesure locale de vitesse, mais sur plusieurs voies il faut résoudre l'association mesure ↔ voiture/voie et corriger la géométrie entre l'axe de déplacement et l'axe du radar.


## Piste de recherche — line-scan / principe photo-finish

> Statut : **candidat très intéressant à prototyper pour les intermédiaires**, pas décision matérielle.

Une caméra 2D classique utilisant une zone rectangulaire de détection présente une ambiguïté temporelle : selon le seuil, le contraste, la forme du véhicule et le traitement, la détection peut être déclenchée à différents endroits de la zone. Cette approche ne doit donc pas être supposée précise au millième.

Une technologie différente est le **line-scan**, utilisée notamment en photo-finish sportive.

Le capteur observe une seule ligne optique alignée avec la ligne de mesure et la relit à très haute fréquence.

```text
       capteur line-scan
             ↓
voie 1  =====|=====
voie 2  =====|=====
voie 3  =====|=====
...          |
voie 10 =====|=====
             ↑
       ligne de mesure
```

La dimension du capteur située dans la largeur de la piste permet de déterminer où le véhicule coupe la ligne, donc potentiellement sa voie. Le temps provient de la succession horodatée des scans.

### Intérêt potentiel pour Lab-Traks

- une seule tête de mesure pour 1 à 10 voies ;
- aucune zone longitudinale de détection à régler ;
- pas un capteur électronique par voie ;
- adaptation aux différents entraxes par configuration logicielle ;
- passages simultanés visibles à des positions différentes de la ligne ;
- possibilité de conserver une preuve visuelle temporelle du passage ;
- fréquence de ligne potentiellement bien supérieure à celle d'une caméra vidéo 2D classique.

### Attention au point de référence du véhicule

Le line-scan résout l'ambiguïté de la **position de la ligne de mesure**, mais il reste nécessaire de définir ce qui constitue le passage d'une voiture :

- premier point de carrosserie ;
- lame-guide si elle peut être observée ;
- autre repère identifiable.

Si une ligne de mesure line-scan est comparée à un autre type de capteur qui détecte la lame-guide, la différence géométrique entre le nez de la voiture et le guide peut introduire un décalage dépendant du véhicule.

Pour des temps comparables, les points de référence doivent donc être cohérents ou le décalage doit être compris et maîtrisé.

### Prototype à étudier

Avant toute décision :

1. capteur linéaire/line-scan abordable ;
2. optique couvrant la largeur maximale visée ;
3. éclairage constant de la seule ligne observée ;
4. fréquence de scan réelle ;
5. timestamp matériel ;
6. détection simultanée sur plusieurs voies ;
7. identification de la voie ;
8. précision mesurée face à un capteur physique de référence ;
9. coût total et simplicité d'installation ;
10. charge CPU et possibilité de traiter localement le signal.

Des travaux publiés montrent qu'un système line-scan rapide peut être réalisé à partir de composants courants à coût fortement inférieur à une caméra line-scan industrielle. Cela justifie l'expérimentation mais ne constitue pas encore une solution Lab-Traks validée.
