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


## Principe fonctionnel — ligne de mesure générique

Indépendamment de la technologie physique retenue, Lab-Traks doit pouvoir modéliser une **ligne de mesure générique**.

Une ligne de mesure produit d'abord un événement élémentaire :

- instant de franchissement ;
- position transversale / voie lorsque le matériel permet de la déterminer ;
- sens lorsque disponible ;
- qualité/confiance et données diagnostiques éventuelles.

La fonction sportive n'est pas codée dans le capteur. La configuration peut donner à la même capacité de franchissement un rôle tel que :

- départ ;
- arrivée ;
- comptage de tour ;
- intermédiaire ;
- limite de secteur ;
- PIT IN ;
- PIT OUT ;
- autre point logique d'un parcours.

Cette abstraction doit fonctionner avec un line-scan, une barrière IR, une coupure électrique, un transpondeur ou tout autre matériel compatible.

### Cas analogique

Sur une piste analogique, la position du franchissement permet généralement de déterminer la voie et la voie détermine le concurrent actuellement affecté à celle-ci.

Une tête multi-voies peut donc éviter une identification individuelle de la voiture pour le chronométrage courant.

Pour les stands, une ligne **PIT IN** et éventuellement une ligne **PIT OUT** permettent de dater l'entrée, la sortie et la durée du passage.

Une ligne de franchissement seule ne prouve pas une présence continue dans toute une zone. Si un règlement ou une fonction exige cette information, une capacité de détection de présence distincte sera nécessaire.

### Vitesse

Une ligne unique fournit un instant de franchissement, pas une vitesse.

Une mesure de vitesse nécessite une information supplémentaire :

- deux lignes spatialement séparées : vitesse moyenne entre ces deux lignes ;
- système optique/multi-ligne capable d'estimer un déplacement sur une distance connue : vitesse locale moyenne sur cette très courte distance ;
- Doppler/radar : vitesse radiale mesurée localement.

Lab-Traks doit conserver la méthode de mesure avec la valeur afin de ne pas présenter des grandeurs différentes comme équivalentes.

## Direction de prototype — tête optique Lab-Traks générique

> Statut : **direction privilégiée à étudier**, sous réserve des résultats du prototype line-scan.

Si le line-scan atteint la précision, la fiabilité et le coût recherchés, l'objectif est d'utiliser autant que possible **la même famille de têtes optiques** pour les différents points de détection.

Exemples :

- départ / arrivée ;
- intermédiaires ;
- limites de secteurs ;
- PIT IN ;
- PIT OUT ;
- points de contrôle rallye ;
- points de contrôle drag ;
- éventuellement mesure de vitesse avec une variante à deux lignes optiques espacées d'une distance connue.

### Traitement local

Une tête optique ne devrait pas imposer au PC de traiter plusieurs flux vidéo bruts en permanence.

La cible architecturale est :

```text
capteur optique / line-scan
          ↓
traitement local de la ligne
          ↓
horodatage local
          ↓
événement compact
          ↓
Lab-Traks
```

Le PC reçoit donc principalement des événements de franchissement et des diagnostics, pas nécessairement les données optiques brutes continues.

Cette approche vise à :

- préserver les performances sur des PC anciens ;
- multiplier les points de mesure sans multiplier les traitements lourds côté PC ;
- garder un câblage simple ;
- utiliser un matériel identique pour plusieurs rôles ;
- permettre une configuration logicielle des voies et du rôle de chaque tête ;
- faciliter remplacement, diagnostic et calibration.

### Une tête, plusieurs largeurs de piste

La tête doit idéalement être indépendante du nombre de voies dans sa plage physique de couverture. Une installation 1, 2, 4, 6 ou 10 voies utilise le même principe ; Lab-Traks calibre les zones transversales correspondant aux voies/slots réellement présents.

Le nombre de voies observables dépendra néanmoins de la résolution optique, de la hauteur de montage, du champ de vision et de la précision obtenue. La capacité « 1 à 10 voies » doit donc être démontrée par prototype et non supposée.

### Variante vitesse

Une tête à deux lignes optiques parallèles séparées d'une distance connue pourrait produire deux timestamps avec la même horloge locale et calculer une **vitesse moyenne locale A-B**. Elle resterait explicitement distincte d'une mesure Doppler/radar.

## Piste de recherche — ligne laser active + capteur linéaire

> Statut : **candidat de prototype**, pas encore un choix matériel.

Une évolution potentiellement plus robuste du principe line-scan consiste à créer activement le plan de détection avec un **laser ligne**.

```text
                 tête optique
             laser + récepteur
                    ↓
      ─────────────────────────   plan / ligne laser
       voie 1  voie 2 ... voie N
```

Le laser ligne matérialise un plan optique très fin à l'emplacement exact du point de chronométrage. Un capteur linéaire / réseau de photodiodes observe la lumière réfléchie ou reçue le long de cette ligne. Lorsqu'une voiture traverse le plan, le profil optique change à une position transversale donnée.

Cette position peut potentiellement être convertie en voie après calibration.

### Point important

Un laser ligne seul ne donne pas la voie. Il faut un récepteur capable de conserver l'information de position le long de la ligne : capteur d'image linéaire, barrette de photodiodes ou autre dispositif équivalent. Une photodiode unique ne fournirait qu'un événement global.

### Intérêt pour Lab-Traks

- le plan laser définit physiquement le lieu du franchissement ;
- éclairage actif potentiellement plus robuste qu'une analyse de contraste sous lumière ambiante ;
- une seule ligne peut couvrir plusieurs voies ;
- la position transversale de la perturbation peut identifier la voie en analogique ;
- même principe possible pour départ/arrivée, intermédiaires, PIT IN et PIT OUT ;
- deux lignes séparées d'une distance connue peuvent fournir une vitesse moyenne locale A-B.

Des systèmes expérimentaux de détection automobile ont déjà utilisé une ligne laser projetée sur la chaussée, une optique et un réseau linéaire de photodiodes. La présence d'un véhicule y est déterminée par la modification/disparition de la lumière réfléchie. Des prototypes ont fonctionné avec un échantillonnage d'environ 10 kHz et plusieurs éléments de capteur répartis sur la largeur observée.

### Questions de prototype

1. largeur de piste couverte à une hauteur de montage acceptable ;
2. résolution transversale suffisante pour distinguer jusqu'à 10 voies ;
3. précision du timestamp au franchissement du plan ;
4. comportement avec carrosseries claires, sombres, chromées ou transparentes ;
5. influence de l'éclairage ambiant et filtrage optique à la longueur d'onde du laser ;
6. détection de deux voitures simultanées ;
7. géométrie laser/récepteur et effet de la hauteur variable des carrosseries ;
8. sécurité laser et puissance minimale utilisable ;
9. coût et miniaturisation de la tête ;
10. comparaison avec un line-scan passif.

Cette piste peut être considérée comme une **détection optique active de ligne**, distincte de la reconnaissance vidéo classique.

### Faisabilité économique et lumière ambiante — première estimation

> Estimation de recherche au 5 octobre 2026. Les références exactes du produit ne sont pas choisies.

Les composants observés montrent que le concept n'est pas économiquement aberrant :

- modules laser ligne maker 650 nm / 5 mW : environ 8 € ;
- modules laser ligne industriels Class 1, 90° : environ 27 à 70 € selon modèle, davantage pour versions réglables/spéciales ;
- TCD1304 3648 pixels : environ 38 € pièce chez certains distributeurs industriels, mais des modules sont observés à moins de 20 € sur des marketplaces grand public ; pour un prototype, ce niveau de prix rend le TCD1304 très intéressant malgré sa résolution largement supérieure au besoin minimal ;
- microcontrôleur de classe Pico 2 : environ 5 USD en carte de développement ;
- un filtre passe-bande industriel 650 nm peut coûter environ 44 €, alors que les filtres optiques de laboratoire très étroits peuvent dépasser largement 100 €.

Un pont/tête de prototype réalisé à l'unité avec des composants industriels sur étagère peut facilement atteindre 100 à 200 €. En utilisant un module TCD1304 à moins de 20 € et des composants maker adaptés, un premier prototype expérimental pourrait plutôt viser environ 40 à 70 € de composants, hors mécanique élaborée et conformité produit. Cela ne représente pas encore le coût d'un produit final.

Pour Lab-Traks, il faut rechercher un récepteur beaucoup moins surdimensionné. Le système automobile expérimental documenté utilisait une barrette de 24 photodiodes et un échantillonnage à 10 kHz ; cela confirme qu'une résolution de plusieurs milliers de pixels n'est pas intrinsèquement nécessaire.

Hypothèse à tester : 32 à 128 positions transversales utiles pourraient suffire pour identifier proprement jusqu'à 10 voies après calibration. Cette plage n'est pas encore validée.

### Cible de coût à étudier

Ordres de grandeur souhaitables, sans engagement tant que le prototype n'existe pas :

- prototype maker fonctionnel : viser ~40–70 € de composants si le module TCD1304 <20 € convient réellement ; prévoir davantage selon laser, optique, alimentation et mécanique ;
- prototype propre/sûr avec laser Class 1, optique et mécanique adaptées : ~120–200 € à l'unité ;
- produit optimisé en petite série : viser un BOM de l'ordre de 40–80 € si un récepteur adapté et une optique économique sont trouvés ;
- prix public souhaitable : idéalement <100 €, encore acceptable vers 100–150 € si la tête est réellement universelle et autonome.

À titre de comparaison marché, les ponts infrarouges DS universels observés en 2026 sont affichés autour de 88 € en 2 voies, 110 € en 4 voies, 168 € en 6 voies et 185–200 € en 8 voies, sans constituer à eux seuls tout le système de chronométrage.

### Immunité à la lumière ambiante

La cible n'est pas d'augmenter brutalement la puissance laser pour dominer la lumière ambiante.

Architecture privilégiée à expérimenter :

1. laser modulé/pulsé à une fréquence connue ;
2. acquisition synchronisée avec cette modulation ;
3. comparaison/soustraction laser ON / laser OFF ou détection synchrone ;
4. filtre optique centré autour de la longueur d'onde du laser si nécessaire ;
5. petit pare-soleil/baffle noir autour du récepteur ;
6. auto-calibration du niveau de fond au démarrage et surveillance de la marge de signal.

La modulation et la détection synchrone sont des techniques industrielles éprouvées pour rejeter lumière ambiante et bruit basse fréquence.

Objectif raisonnable : fonctionnement automatique de l'obscurité à un environnement intérieur très fortement éclairé.

Le **soleil direct dans le récepteur ne doit pas être promis avant essais** : même un signal modulé ne peut pas être extrait si la photodiode ou l'étage analogique est physiquement saturé. Les capteurs photoélectriques industriels eux-mêmes publient des limites d'éclairement et recommandent d'éviter le soleil direct.

### Couverture géométrique indicative

Un laser ligne avec angle d'éventail de 90° produit théoriquement une largeur proche de deux fois la hauteur de montage :

- hauteur 25 cm → ~50 cm de ligne ;
- hauteur 30 cm → ~60 cm ;
- hauteur 50 cm → ~1 m ;
- hauteur 75 cm → ~1,5 m.

Un module commercial 90° confirme par exemple une ligne de 2 m à 1 m de distance. Le récepteur doit naturellement posséder un champ de vue compatible.

Cette géométrie rend plausible une seule tête pour 1 à 10 voies, avec hauteur/montage adaptés, mais la précision aux extrémités et les distorsions optiques devront être mesurées.

### Sécurité laser

Pour un produit destiné à des clubs et au public, privilégier une conception dont le système final est correctement classifié et sûr. Des modules ligne commerciaux existent en Class 1. Un module maker 5 mW vendu comme Class III peut servir à des essais encadrés mais ne constitue pas une référence acceptable pour le produit final.

Le coût de conformité/certification du produit final devra être intégré au projet et ne se résume pas au prix de la diode.

### Contrainte économique — le coût répété est prioritaire

Le coût d'un point de détection ne doit pas être justifié par comparaison avec les systèmes historiques plus chers. Une installation peut nécessiter plusieurs points (départ/arrivée, intermédiaires, PIT IN/PIT OUT, etc.) : le coût se multiplie immédiatement.

Objectif de conception : rendre la partie répétée sur la piste aussi simple et économique que possible. Une tête à 50 € peut déjà devenir trop coûteuse lorsqu'elle est multipliée par trois, quatre ou davantage.

Lab-Traks ne doit pas dépendre de son propre matériel pour être adopté : les matériels existants compatibles restent utilisables via les adaptateurs. Le futur matériel Lab-Traks doit apporter un avantage réel de simplicité, universalité et coût, pas seulement reproduire un pont existant.

#### Candidat ultra-low-cost : barre optique segmentée

> Statut : piste de recherche, non validée optiquement.

Au lieu d'un CCD linéaire complet dans chaque tête, étudier une barre contenant de nombreux phototransistors/photodiodes très économiques, éclairée par une ligne optique commune.

```text
éclairage ligne actif
────────────────────────────────
       voiture
          ↓
● ● ● ● ● ● ● ● ● ● ● ● ● ● ● ●   barre de réception
        x x x
          ↑
   cellules perturbées
```

Après calibration, les groupes de cellules correspondent aux positions transversales et donc aux voies en analogique. Le système n'a pas besoin de mesurer une image : il mesure uniquement un profil lumineux discret.

Ordres de grandeur composants observés en octobre 2026 :

- phototransistors SMD : environ 0,03 à 0,05 USD pièce à volume modéré ;
- 64 phototransistors : environ 2 à 4 USD ;
- RP2040 nu : environ 0,76 USD à 100 pièces ;
- multiplexeurs analogiques 8:1 économiques : quelques centimes à ~0,20 USD selon référence/volume ;
- module laser ligne maker : quelques euros.

Une architecture 48/64 cellules pourrait donc avoir une électronique de base très peu coûteuse. Le coût réel sera dominé par PCB, mécanique, connectique, optique/éclairage, assemblage, sécurité/conformité et marge commerciale.

Première cible de recherche : vérifier s'il est possible d'obtenir un coût de composants électroniques/optique de l'ordre de 10–15 € par point de détection avant mécanique et conformité. Ce n'est pas encore un objectif de prix public validé.

Le TCD1304 reste très utile comme instrument de prototype et de caractérisation grâce à sa forte résolution. Il ne doit pas être supposé nécessaire dans le produit final.

### Critère éliminatoire — lumière ambiante et géométrie du rideau

Le concept de rideau optique n'est intéressant pour Lab-Traks que s'il supprime, dans son domaine d'utilisation garanti, la dépendance pratique à la luminosité ambiante. L'utilisateur ne doit pas avoir à régler sa détection parce que la salle est plus claire, plus sombre ou éclairée différemment.

Il est physiquement incorrect de promettre « 100 % quelle que soit toute luminosité imaginable » : un récepteur optique peut être saturé, notamment par une source extrêmement intense ou le soleil direct dans l'optique. La cible produit doit donc être :

- très large enveloppe lumineuse testée et garantie ;
- aucune adaptation manuelle à l'éclairage dans cette enveloppe ;
- source active modulée et mesure synchrone / soustraction du fond ;
- filtrage spectral et mécanique si nécessaire ;
- mesure permanente de la marge signal/bruit ;
- détection explicite de saturation ou de signal insuffisant ;
- en dehors de l'enveloppe valide, **refuser la mesure plutôt que générer un faux passage silencieux**.

Cette robustesse à la lumière est un **critère éliminatoire** pour retenir le rideau optique comme solution Lab-Traks.

#### Rien sous la piste

Une barrière traversante classique exige un émetteur et un récepteur opposés. Pour Lab-Traks, placer une électronique de réception sous la piste ou un capteur par voie irait à l'encontre de l'objectif d'installation universelle.

Architecture à privilégier pour la recherche : source active et récepteur dans la même tête au-dessus de la piste, en mode réflexion active. La source crée une ligne optique sur la piste ; une optique image cette ligne sur un capteur linéaire. La voiture masque/modifie localement le retour et sa position transversale fournit la voie.

```text
             tête unique
       source + récepteur linéaire
                 ↓ ↑
                 ↓ ↑ retour
================================= piste
        ligne optique active
```

Le principe peut utiliser une source visible ou proche infrarouge. Un laser visible peut éventuellement n'être qu'une aide d'alignement ; la technologie finale d'illumination reste à choisir.

#### Vitesse des capteurs linéaires — correction importante

- TCD1304 : 3648 pixels, mais Toshiba indique environ 0,2 kHz de line rate ; intéressant pour caractériser l'optique, probablement trop lent comme référence de chronométrage au millième.
- TSL1401CL : 128 pixels, fonctionnement jusqu'à 8 MHz et intégration minimale d'environ 33,75 µs ; très intéressant techniquement et économiquement pour expérimentation, mais **produit officiellement discontinué** par ams OSRAM et donc impropre comme dépendance produit à long terme.
- TCD1103GFG : composant Toshiba actif, 1500 pixels, environ 1,2 kHz de line rate et autour de 10–14 € selon quantité/distributeur ; candidat à étudier mais marge temporelle plus faible, surtout si la stratégie lumineuse nécessite plusieurs acquisitions.
- des capteurs linéaires actifs beaucoup plus rapides existent (plusieurs kHz à >10 kHz), mais leur coût doit rester compatible avec la contrainte économique du projet.

Conclusion : spécifier d'abord les besoins minimaux (line rate, nombre de positions, dynamique, synchronisation lumineuse, coût), puis chercher le capteur actif le moins cher qui les satisfait. Ne pas concevoir le produit autour d'une référence obsolète ou simplement bon marché.

## Direction de recherche — ligne d'arrivée universelle + identité indépendante

> Statut : **orientation de conception à explorer**, pas encore un choix matériel.

Lab-Traks ne doit pas concevoir son futur système de détection comme un accessoire spécifique au slot. Le cas cible peut aussi être une voiture RC ou tout autre mobile franchissant une ligne sans notion de voie physique.

Le système doit donc séparer deux faits élémentaires :

1. **franchissement** : un objet franchit une ligne physique donnée à un instant précis ;
2. **identité** : quel véhicule / concurrent est associé à ce franchissement.

Le capteur de ligne peut produire au minimum un timestamp et, si disponible, une position transversale X. Il ne doit pas supposer qu'une position X est toujours une « voie ».

```text
ligne de mesure -> Crossing(timestamp, position?, direction?, quality)
identification  -> IdentityObservation(vehicle/transponder, timestamp?, quality)
                         ↓
                corrélation Lab-Traks
                         ↓
              passage sportif identifié
```

### Selon la discipline

- **slot analogique** : la position transversale / voie peut suffire à identifier le concurrent ; aucun transpondeur embarqué n'est nécessaire ;
- **slot numérique** : l'identité peut provenir du système numérique existant ou d'un autre mécanisme ;
- **RC / véhicules libres** : aucune hypothèse de voie ; une identité embarquée devient nécessaire si plusieurs véhicules roulent ;
- **matériel existant** : MyLaps/AMB, MRT, OpenStint, autres transpondeurs ou systèmes compatibles doivent pouvoir fournir l'identité via adaptateur plutôt que forcer le remplacement du matériel.

### Micro-transpondeur Lab-Traks — cible de recherche

Si Lab-Traks propose un transpondeur propre, sa mission première peut être volontairement minimale : **émettre une identité robuste et unique**. Il n'est pas nécessaire de lui faire porter toute la précision du chronométrage si la ligne de mesure fournit déjà le timestamp officiel.

Objectifs :

- coût unitaire très faible ;
- consommation très faible ;
- alimentation compatible avec les petits véhicules ;
- dimensions suffisamment petites pour viser les véhicules les plus contraints, avec comme test de miniaturisation une voiture de slot F1 ;
- aucune dépendance à la lumière ou à la visibilité si une technologie magnétique/RF est retenue ;
- protocole ouvert et documenté ;
- identité configurable sans infrastructure centrale obligatoire ;
- coexistence de nombreux véhicules ;
- possibilité d'utiliser les transpondeurs existants sans transpondeur Lab-Traks.

### Références techniques encourageantes

- OpenStint v2 utilise un ATtiny816/1616/3216, un driver push-pull et une antenne magnétique 5 MHz. Une fabrication assemblée de 40 cartes a été annoncée à moins de 200 USD taxes et livraison incluses en 2026, soit moins de 5 USD par carte avant programmation, câblage et protection.
- l'ATtiny816 existe en boîtier VQFN 3 x 3 mm : le microcontrôleur n'est donc pas nécessairement le facteur dimensionnant ;
- des transpondeurs magnétiques RC existants sont de l'ordre de 19 x 16 x 6 mm ;
- des conceptions open-source RCHourglass existent autour de 19,5 x 19,6 mm ;
- OpenStint indique que sa propre antenne PCB pourrait encore être réduite en exploitant davantage les couches du PCB.

Le principal défi de miniaturisation d'un transpondeur magnétique est donc probablement **l'antenne / la boucle et son couplage**, davantage que la logique numérique.

### Pistes d'antenne à étudier

- antenne multi-couches directement dans le PCB ;
- antenne flexible séparée du minuscule PCB logique ;
- boucle imprimée adaptée à la forme du véhicule ;
- éventuelle antenne externe très légère autour d'une zone du châssis ;
- comparaison avec d'autres technologies d'identification uniquement si elles conservent coût, taille, robustesse et coexistence.

### Point de vigilance : corrélation

Découpler franchissement et identité est puissant, mais crée un problème à résoudre proprement lorsque plusieurs véhicules franchissent la ligne presque simultanément. Le moteur doit pouvoir associer sans ambiguïté chaque `Crossing` à la bonne `IdentityObservation`. Les technologies retenues devront être évaluées spécifiquement sur ce cas, pas seulement sur des passages isolés.

Le but n'est donc pas de construire « un meilleur pont slot », mais un **point de mesure universel** auquel différentes technologies d'identification peuvent être associées.

### Intérêt de l'identité embarquée même en slot analogique

En slot analogique, la voie reste suffisante pour le fonctionnement normal et un transpondeur ne doit pas devenir obligatoire. Une identité embarquée optionnelle apporte cependant une sécurité supplémentaire importante.

Cas concret : une voiture affectée à la voie 3 déslote juste avant la ligne et traverse physiquement la zone de mesure de la voie 4. Une détection purement par voie peut attribuer à tort un tour au concurrent de la voie 4.

Si la ligne universelle fournit simultanément une position transversale et qu'une identité embarquée est disponible :

```text
voiture #17 attendue voie 3
Crossing : position = zone voie 4
Identity : #17
        ↓
incohérence identité / trajectoire attendue
        ↓
aucun tour attribué automatiquement à la voie 4
aucun tour attribué automatiquement à #17
événement conservé comme franchissement hors trajectoire / anomalie
```

Cette logique évite qu'un déslotage, un rebond ou un passage parasite ne crédite le mauvais concurrent.

Principe : **l'identité répond à « qui ? », la ligne répond à « quand et où ? », le Race Engine décide si le franchissement est sportivement valide.**

Le système reste dégradé proprement selon les capacités présentes :

- position/voie seule : fonctionnement analogique classique ;
- identité seule : passage identifié mais sans validation spatiale ;
- identité + position : contrôle de cohérence renforcé ;
- aucune technologie Lab-Traks imposée si le matériel existant fournit déjà les capacités nécessaires.

Cette capacité renforce l'intérêt d'un micro-transpondeur optionnel suffisamment petit et économique pour être installé même dans les voitures de slot les plus contraintes.

## Recherche — zones de puissance et drapeau jaune automatique local

> Statut : **capacité avancée à explorer**, non requise pour le chronométrage de base et non validée comme matériel obligatoire.

La séparation entre mesure, identité et décision sportive ouvre la possibilité d'une gestion active de la piste. Si le circuit est électriquement découpé en secteurs contrôlables, Lab-Traks pourrait demander à un module de puissance de réduire ou couper temporairement la puissance d'une zone lors d'un incident.

Exemples de niveaux d'action :

- anomalie isolée : journaliser / avertir sans action automatique ;
- sortie probable d'un véhicule : jaune local selon règle configurée ;
- plusieurs anomalies rapprochées dans une même zone : incident multiple / `big one` probable ;
- zone d'approche : limitation de puissance avant la zone d'incident ;
- incident majeur : très forte limitation, coupure locale ou drapeau global selon règles et capacités du matériel.

Le ralentissement ne doit pas commencer uniquement dans la zone où se trouve l'obstacle : il peut être nécessaire d'agir sur un ou plusieurs secteurs **en amont** afin que les véhicules arrivent déjà ralentis.

```text
secteur 1        secteur 2         secteur 3        secteur 4
  VERT        JAUNE / LIMITÉ        INCIDENT          VERT
───────────┬──────────────────┬──────────────────┬──────────
                                     X X
```

### Abstraction proposée

Le Race Engine ne commande pas directement un MOSFET, une tension ou un protocole constructeur. Il émet une intention abstraite, par exemple :

```text
PowerZoneCommand(
  zone = 2,
  state = YELLOW,
  requested_limit = 0.50,
  reason = INCIDENT_AHEAD
)
```

L'adaptateur matériel traduit ensuite cette intention selon ses capacités : limitation PWM/tension, coupure, commande numérique de vitesse, pace-car/yellow flag constructeur, simple signalisation ou absence d'action si non supporté.

### Analogique

Une limitation locale est techniquement possible avec des secteurs de rails électriquement isolés et une électronique de puissance adaptée. Le mode normal doit rester aussi transparent que possible pour le contrôleur du pilote. La conception devra tenir compte des différents contrôleurs, du freinage dynamique, des alimentations, des power taps et des courants de démarrage. Une piste existante non sectorisée conserve naturellement le fonctionnement global classique.

### Numérique

Une coupure électrique locale peut également couper la communication numérique ou provoquer des effets indésirables sur les décodeurs. Quand le système numérique permet de commander la vitesse des véhicules, préférer une commande protocolaire de limitation à une coupure brute de la voie. Les capacités réelles sont exposées par l'adaptateur.

### Détection d'incident

Ne pas déduire automatiquement un accident d'un seul signal faible. Les sources possibles peuvent être combinées :

- identité détectée dans une position incohérente ;
- franchissement hors trajectoire ;
- véhicule attendu qui ne rejoint pas le point suivant ;
- plusieurs anomalies dans la même zone dans une fenêtre temporelle courte ;
- capteurs/mesures supplémentaires disponibles ;
- action manuelle du directeur de course ou des pilotes.

Le seuil de déclenchement, la durée, le niveau de puissance et la politique de reprise sont des **règles configurables**, jamais des constantes câblées dans le matériel.

Principe de sûreté : une automatisation de jaune local doit être testable, désactivable et explicable. Une détection incertaine peut alerter sans agir ; une action automatique ne doit être autorisée que lorsque les critères configurés sont satisfaits.

## Principe d'installation — aucune modification irréversible de la piste

> Orientation produit forte : le matériel Lab-Traks doit, autant que techniquement possible, se poser sur une piste existante sans perçage, découpe, fraisage ni intégration permanente de capteurs dans les rails.

Une même `MeasurementLine` logique peut être matérialisée par plusieurs formes physiques. Le Race Engine utilise ses capacités déclarées et ne dépend pas de sa mécanique.

### Variante A — pont / tête au-dessus de la piste

Installation amovible au-dessus de la piste. Candidat privilégié lorsque l'on souhaite :

- timestamp sur une ligne optique physiquement bien définie ;
- position transversale du franchissement ;
- déduction de voie en analogique ;
- contrôle de cohérence identité / trajectoire ;
- éventuellement fonctionnement sans transpondeur pour le slot analogique.

### Variante B — ruban / antenne sous la piste

Une antenne magnétique très fine, un ruban flexible ou une boucle équivalente peut être placé sous une piste compatible, sans visibilité depuis le dessus et sans percer la piste. Cette variante suppose une identité embarquée ou un transpondeur compatible.

Les systèmes de chronométrage RC à couplage magnétique démontrent déjà qu'une boucle de réception peut être installée sous une piste et détecter un transpondeur sans ligne de vue. La géométrie, la portée et la fiabilité doivent toutefois être validées spécifiquement sur :

- pistes plastiques avec rails métalliques ;
- pistes bois avec cuivre/tresse ;
- différentes épaisseurs ;
- différentes hauteurs et orientations de transpondeur ;
- moteurs, alimentations et parasites électriques du slot ;
- plusieurs voitures proches ou simultanées.

Une boucle magnétique classique possède une **zone de couplage** et ne doit pas être supposée équivalente à une ligne optique très fine. Sa précision de timestamp, son point de référence physique et sa capacité à départager des arrivées très serrées devront être mesurés.

### Variante C — hybride

Un utilisateur peut combiner une ligne optique amovible et une antenne d'identification sous piste :

```text
          pont optique
       QUAND + OÙ exactement
               ↓
================================ piste
--------------- ---------------- ruban / boucle sous piste
               ↑
              QUI
```

Cette combinaison pourrait fournir simultanément une référence géométrique de franchissement très précise et une identité robuste indépendante de la lumière.

### Installation par capacités

Lab-Traks ne doit pas présenter ces variantes comme des matériels concurrents mais comme des fournisseurs de capacités :

```text
OpticalGate       -> crossing_time + transverse_position
UnderTrackLoop    -> vehicle_identity + crossing_zone
HybridGate        -> crossing_time + transverse_position + vehicle_identity
LegacyDetector    -> capacités exposées par son adaptateur
```

Le logiciel adapte ensuite les fonctions disponibles. Une installation simple reste fonctionnelle ; une installation enrichie gagne en contrôle sans nécessiter de ressaisie ni de modèle de course différent.

### Objectif économique supplémentaire

Les éléments répétés autour du circuit doivent être les moins coûteux et les plus passifs possible. Une piste de recherche importante consiste à mutualiser l'électronique coûteuse de décodage et à rendre chaque point supplémentaire essentiellement constitué d'une antenne/ruban/élément simple, si cela peut être fait sans dégrader la précision ni manquer des passages simultanés.

Le but d'expérience utilisateur est : **poser, brancher, calibrer, courir — jamais modifier définitivement la piste pour adopter Lab-Traks.**
