# Reverse engineering des systèmes de détection de PC Lap Counter

**Objectif :** comprendre comment PC Lap Counter (PCLC) reçoit, interprète et renvoie les informations de ses différents systèmes de détection afin de concevoir :

1. un moteur de détection indépendant du matériel ;
2. des adaptateurs compatibles avec les systèmes historiques ;
3. à terme, un boîtier universel capable de concentrer plusieurs familles d'interfaces.

**Archive analysée :** `Pc Lap Counter.zip`  
SHA-256 : `e66cf3136bd76f1a8cbf16d201c83cdd383519c9e0ce733b57bf70fae5682383`

Cette analyse combine : inventaire binaire, chaînes ASCII/UTF-16, fichiers INI et langues, extraction des objets PowerBuilder 6 contenus dans les PBD, inspection des DLL/OCX et confrontation avec les documentations publiques encore disponibles.

---

## 1. Résultat principal

PCLC ne possède pas **un** protocole de détection. Il possède une collection d'adaptateurs spécialisés qui ramènent des matériels très différents vers quelques événements internes communs.

Les familles rencontrées sont :

1. **Entrées/sorties brutes** : LPT, Velleman K8000, Phidget. PCLC voit un changement électrique et décide lui-même qu'il s'agit d'un passage, d'un PIT, d'un bouton, etc.
2. **Chronomètres externes** : DS Racing, RaceControl, exoTIC, XLOT, Scalextric RMS. Le boîtier produit déjà un numéro de voie et parfois un temps ; PCLC interprète la trame.
3. **Systèmes digitaux de slot** : Carrera, Scalextric SSD, SCX Digital, Ninco Digital, Slot.it oXigen, Scorpius, Davic. Le protocole contient potentiellement l'identité de la voiture, les passages, PIT, carburant, accélérateur, boutons, état de course et parfois des commandes PC -> piste.
4. **Systèmes à transpondeur** : AMB/MyLaps, I-Lap, RfLapCounter, Trackmate, Robitronic, TAG Heuer, Mantis II. La donnée fondamentale est l'identifiant du véhicule, souvent accompagnée d'un timestamp ou temps de passage.
5. **Protocole générique PCLC/Arduino** : interface publique bidirectionnelle conçue précisément pour brancher un matériel tiers à PCLC. C'est notre meilleur point de compatibilité PCLC.

### Événements internes communs retrouvés

Les modules spécialisés convertissent leurs données vers un petit vocabulaire PCLC récurrent :

- `detect` : passage / voie / voiture ;
- `LAPTIMEIO` : temps fourni par le matériel au lieu d'être calculé par PCLC ;
- `FUELIO` : niveau ou information carburant ;
- `PITIN` / `PITOUT` : entrée / sortie des stands ;
- `TRANSIO` : identifiant de transpondeur ;
- événements départ, pause, reprise, arrêt, drapeau jaune, power, Stop&Go, etc.

**Conséquence pour notre futur logiciel :** ce vocabulaire PCLC n'a pas vocation à devenir le vocabulaire de notre application. Chaque protocole externe sera décodé par un adaptateur dédié, puis traduit vers le modèle d'événements propre à notre logiciel. Le cœur ne doit dépendre ni des noms, ni du format, ni des particularités historiques de PCLC ou d'un matériel tiers.

---

## 2. Niveaux de connaissance

| Niveau | Signification |
|---|---|
| **A** | protocole ou fonctionnement suffisamment documenté pour implémentation directe ;
| **B** | transport, paramètres et événements bien identifiés, mais format exact de certaines trames encore incomplet ;
| **C** | comportement fonctionnel identifié, protocole brut à capturer/décoder ;
| **D** | présence du module connue mais données insuffisantes pour une implémentation sérieuse. |

---

## 3. Matrice générale

| Système | Famille | Transport visible | Données reçues par PCLC | PC -> matériel | Niveau |
|---|---|---|---|---|---|
| PCLC Arduino | générique | COM, 8N1 | voie, timestamp/temps, PIT, boutons, RFID/transpondeur | feux, power, états, carburant | **A** |
| LPT1 | E/S brute | port parallèle, base 0x378 par défaut | état/pulsation des pins | relais, power, feux | **A/B** |
| Velleman K8000 | E/S brute | K8000 / I²C via port parallèle | canaux I/O | sorties/relais/feux | **A/B** |
| Phidget | E/S brute | USB SDK Phidget | entrées digitales, PIT | power, stations, feux | **A/B** |
| Scalextric RMS | chrono | COM 9600 8N1 | passages | très limité | **B/C** |
| DS-030 | chrono | RS-232, 4800 8N1 | voie/temps | non | **B** |
| DS-200 | chrono | RS-232, 4800/56000 selon matériel | voie/temps + boutons | limité | **B** |
| DS-300 | chrono | RS-232, 4800/56000 selon matériel | voie/temps + boutons | limité | **B** |
| DS-0045 | chrono | série 8N1 | voie/temps + événements | oui, dont contrôle course/Stop&Go indirect | **B** |
| XLOT | chrono | série 8N1 | passages | inconnu/limité | **B/C** |
| RaceControl SensorBox | chrono/capteurs | jusqu'à 4 COM, DLL `jnutl.dll` | événement voie + timestamp + PIT | API propriétaire | **B/C** |
| exoTIC Systems | chrono | jusqu'à 4 COM, 8 voies/boîtier | voie + temps optionnel | non identifié | **B/C** |
| Ninco Digital | digital | série | voiture, temps, fuel, PIT | états/boutons selon version | **B** |
| Ninco PB 1.08 | digital | série, 7N1 pour flux PB | voiture, temps, fuel + capteurs externes PIT | limité | **B** |
| Scalextric C7030 / PB-Pro | digital | série | voiture, throttle, brake, LC, fuel, PIT | power/commandes | **B** |
| Scalextric C7042 / SSDA6 | digital | **RS-485 half-duplex, 19200 8N1** | contrôleurs, passages, timer, état piste | conduite des voitures, LEDs, power | **A** |
| SCX Digital SEB modes 1/2/USB | digital | série/USB-série | voiture, throttle, LC, fuel, temps | commandes/état selon mode | **B/C** |
| Carrera 30342 | digital | TTL/série | voiture, passage, fuel/temps selon mode | limité | **B/C** |
| Carrera CU 30352 | digital | TTL/série, PCLC expose 38400 | passage, fuel, PIT, start/pause | nombreuses commandes CU | **B** |
| Slot.it Universal Live Timing Box | chrono/telemetry | série/USB COM | temps, throttle, fuel, PIT | certaines commandes | **B** |
| Slot.it oXigen | digital RF + dongle USB COM | dongle / COM | voiture, lap, PIT, throttle, fuel, état | power, limites, race status, etc. | **A*** |
| Scorpius | digital RF + dongle | USB COM / RF 2,4 GHz | voiture, throttle, Lane Brain PIT | commandes via dongle | **B** |
| Davic | digital | COM 19200 dans le module PCLC | voiture, fuel, PIT | partiel | **B/C** |
| Trackmate | transpondeur/IR | COM 8N1 | voie ou ID + temps | réglages envoyés au décodeur | **B/C** |
| RfLapCounter | transpondeur RF | COM virtuel | ID 16 bits répété | pas de handshake requis | **A/B** |
| I-Lap RC | transpondeur IR | COM, module PCLC utilise 8N2 | ID + temps | reset/config | **B/C** |
| AMB20/AMB9200 | transpondeur | COM, module PCLC utilise 8N2 | ID + temps | reset | **B/C** |
| AMBrc | transpondeur | décodeur en protocole P3 | ID + temps | selon décodeur | **B/C** |
| Robitronic | transpondeur | COM 38400 | ID + temps | reset | **B/C** |
| TAG Heuer | transpondeur/chrono | COM 9600/19200 | ID + temps, boucles start/pit | commande start visible | **C** |
| Mantis II RFID | RFID réseau | TCP/IP, port 6500 par défaut | tag + temps | sensibilité/reset lecteur | **B/C** |

`A*` pour oXigen : le protocole PC-dongle est officiellement public ; il faudra coder contre la version officielle visée et faire des tests de compatibilité firmware.

---

# 4. Analyse détaillée

## 4.1 Protocole générique PCLC / Arduino — priorité maximale

Le PDF officiel PCLC décrit un protocole ASCII très simple : chaque message commence par `[` et se termine par `]`. Réglage par défaut : **9600 bauds, 8 bits, aucune parité, 1 stop** ; PCLC permet aussi de sélectionner d'autres bauds sur les versions récentes.

### Matériel -> PCLC

- `[SFnn]` : passage départ/arrivée, PCLC calcule le temps ;
- `[SFnn?HH:MM:SS.mmm]` : passage avec timestamp ;
- `[SFnn$laptime]` : passage avec temps au tour en millisecondes ;
- `[PInn]` : PIT IN ;
- `[POnn]` : PIT OUT ;
- `[BT01]` à `[BT09]` : commandes externes départ/restart/pause/power/end/yellow ;
- `[SGnn]` : Stop&Go ;
- `[RFtagid]` : RFID ;
- `[TRid]`, `[TRid?timestamp]`, `[TRid$laptime]` : passage par transpondeur.

### PCLC -> matériel

- `[SLln1/0]` : feux 1..5, GO, STOP, Yellow ;
- `[PWnn1/0]` : power par voie/voiture ou `00` pour toutes ;
- `[FSnn1/0]` : faux départ ;
- `[SGnn1/0]` : Stop&Go ;
- `[FPnn1/0]` : premier ;
- `[OFnn1/0]`, `[LFnn1/0]`, `[MFnn1/0]` : états carburant ;
- `[PTnn1/0]` : PIT ;
- `[CF...]` : notification de franchissement ;
- `[RC...]` : horloge/état de course ;
- `[FL...]` : niveau carburant/refuel.

### Conséquence

**Notre boîtier universel doit obligatoirement proposer un mode “PCLC Arduino compatible”.** Ainsi, peu importe ce qui se trouve derrière le boîtier — LPT, capteur optique, DS, transpondeur, RS-485 — PCLC voit une interface propre et publique.

C'est la manière la plus robuste de rester compatible avec PCLC sans dépendre de ses mécanismes internes PowerBuilder.

---

## 4.2 LPT1 — détection électrique brute

Fichiers : `rclpt.exe`, `PCLCLPT1.exe`, `DLPORTIO.dll`, `DLPORTIO.sys`, `Language/WLPTENG.ini`.

Configuration retrouvée :

- `LPT1ADR=888` décimal = **0x378**, adresse classique LPT1 ;
- fonctions de bas niveau : `PortIn`, `PortWordIn`, `PortDWordIn`, `PortOut` ;
- mesure temporelle avec `QueryPerformanceCounter` ;
- mapping `Pin -> Lane` jusqu'à 32 voies ;
- pins 1 à 17 visibles dans l'interface ;
- pins 2 à 9 explicitement proposées pour power control ;
- feux départ 1..5, GO, STOP ;
- actions externes : départ/pause/restart, pause, restart, start, power off/on, end race ;
- filtre de durée minimale des impulsions ;
- affichage possible des impulsions rejetées ;
- état initial ON configurable ;
- détection PIT possible par maintien d'un capteur pendant une durée donnée, puis PIT OUT au relâchement.

### Logique

PCLC ne reçoit pas un « tour » du LPT : il **échantillonne des bits**, mesure la durée de l'état actif et transforme un changement valide en événement voie. C'est donc notre modèle de référence pour les capteurs bruts.

### Pour notre boîtier

Créer des entrées logiques configurables : inversion, pull-up/pull-down, front montant/descendant, durée minimum, anti-rebond, dwell PIT, mapping capteur -> voie. Le protocole natif ne doit jamais dépendre d'un numéro de pin physique.

---

## 4.3 Velleman K8000 — E/S I²C via port parallèle

Le module qui semblait être `rcserver.exe` est en fait explicitement : **“Velleman K8000 Interface for PC Lap Counter”**.

Il appelle `K800032.dll` avec notamment :

- `SelectI2CprinterPort` ;
- `I2CBusNotBusy` ;
- `ConfigIOchannelAsInput` ;
- `ReadIOchannel` / `ReadIOchip` ;
- `ConfigIOchannelAsOutput` ;
- `SetI2CBusDelay`.

PCLC associe des **canaux I/O à des voies**, configure des sorties de puissance et des feux.

### Conclusion

C'est encore un système d'E/S brutes. Il ne faut pas reproduire le K8000 dans notre cœur logiciel : notre couche `RawDigitalIO` doit couvrir le même besoin. Une compatibilité avec un K8000 physique pourra être un adaptateur séparé si cela reste utile.

---

## 4.4 Phidget — version moderne de l'E/S brute

`phidget.ini` révèle très clairement le modèle de données :

- **48 entrées** possibles `L1..L48` ;
- pour chaque entrée : durée minimum `Lx_MIN` ;
- 48 sorties `LANEPOWER` ;
- 48 `STATION` ;
- feux 5..1, GO, STOP, PaceCar ;
- par voie : `PITIN`, `PITIN_MIN`, `PITOUT`, `PITOUT_MIN`.

Les DLL/OCX Phidget sont fournies avec PCLC.

### Conclusion

Le comportement à reproduire dans notre architecture est évident : E/S digitales configurables, filtrage temporel et sorties. Il n'est pas nécessaire que le futur boîtier « soit un Phidget » ; il doit offrir la même abstraction. Un adaptateur logiciel Phidget pourra aussi rester disponible pour les installations existantes.

---

## 4.5 DS Racing — DS-030 / DS-200 / DS-300 / DS-0045

PCLC possède quatre modules distincts : `rcds.exe`, `rcds200.exe`, `rcds300.exe`, `rcds045.exe`.

### DS-030

- liaison série 8N1 ;
- documentation PCLC : **4800 bauds** ;
- peut utiliser plusieurs boîtiers pour étendre le nombre de voies ;
- PCLC peut utiliser le temps fourni par le DS (`LAPTIMEIO`).

### DS-200 / DS-300

- série 8N1 ;
- des appareils fonctionnent à 4800 ou **56000** bauds selon la documentation PCLC ;
- synchronisation nécessaire au démarrage : passage d'une voiture ou pression Start/Stop ;
- les boutons du DS peuvent être traduits en START, STOP, PAUSE, RESUME pour PCLC ;
- PCLC reste le maître de la course, le DS sert de détecteur.

### DS-0045

- série 8N1 ;
- réception de passages/temps ;
- différence majeure : le DS-0045 **accepte des commandes du PC** ;
- PCLC peut donc indirectement piloter un Stop&Go DS via le DS-0045.

### Point non résolu

Le format octet par octet des trames DS n'est pas encore extrait avec une confiance suffisante. Il faudra soit une documentation DS, soit une capture série contrôlée.

> Correction par rapport à une première lecture : il serait faux d'affirmer que « DS-0045 = 57600 bauds ». L'archive présente plusieurs valeurs possibles dans ses contrôles ; la documentation PCLC confirme explicitement 4800 pour DS-030 et 4800/56000 pour certains DS-200/300. Le baud exact DS-0045 devra être validé sur le matériel/manuel visé.

---

## 4.6 Scalextric RMS

`rcrms.exe` :

- COM **9600 8N1** ;
- interface historique de détection ;
- événement principal : passage de voie ;
- peu de télémétrie.

C'est un adaptateur série ancien, nettement plus simple que les systèmes SSD digitaux.

---

## 4.7 RaceControl SensorBox

`rcrc.exe` utilise une DLL spécifique : `jnutl.dll`, qui s'identifie comme **“Utility Interface Library for RaceControl Sensor Box”**.

Exports visibles :

- `InitBoxes` ;
- `OpenBox` ;
- `RX_data` ;
- `Get_Event` ;
- `Get_TStamp` ;
- `Get_TXDat` ;
- `Timeout_Check`.

PCLC peut ouvrir jusqu'à quatre ensembles : voies 1–4, 5–8, 9–12, 13–16. Le module sait générer :

- passage + temps ;
- PIT IN / PIT OUT ;
- PIT calculé après maintien du capteur pendant un délai configurable.

Le module expose 4800/56000 dans son UI mais la DLL encapsule une partie du protocole. Pour une reproduction propre, une capture matériel est préférable à la copie du comportement interne de la DLL.

---

## 4.8 exoTIC Systems

`rcexotic.pbd` :

- jusqu'à 4 COM ;
- 8 voies par ensemble -> 32 voies ;
- réception d'un passage et option de récupérer le temps du matériel (`LAPTIMEIO`) ;
- module configuré autour de 9600 dans l'objet extrait, avec d'autres valeurs historiques présentes dans les composants série.

Format brut non encore établi : capture nécessaire.

---

## 4.9 XLOT

`rcxlot.exe` :

- série 8N1 ;
- jusqu'à quatre COM ;
- valeur 9600 présente comme réglage principal, avec 4800/56000 également dans les contrôles historiques ;
- PCLC transforme les données reçues en passages.

Format des trames non encore suffisamment isolé.

---

## 4.10 Ninco Digital / Powerbase 1.08

Deux interfaces : `rcninco.exe` et `rcninco108.exe`.

Éléments retrouvés :

- flux série utilisant notamment **7N1** côté Powerbase ;
- valeurs 1200 et 38400 présentes suivant le sous-système/interface ;
- passages voiture ;
- temps au tour ;
- carburant ;
- PIT IN / PIT OUT ;
- événements start/pause/restart ;
- possibilité d'ajouter un Multi Lane Sensor pour PIT/SF additionnels ;
- option d'inversion PIT IN/OUT.

Le protocole exact dépend de la version Ninco et des capteurs annexes. Il faudra figer les matériels à supporter avant d'implémenter.

---

## 4.11 Scalextric SSD C7042 / Advanced 6 Car Powerbase — très intéressant

C'est l'un des protocoles les mieux documentés.

Transport officiel :

- UART sur **RS-485 half-duplex** ;
- **19200 bps, 8N1** ;
- paquet PC -> powerbase puis réponse powerbase -> PC ;
- CRC-8 polynôme `x^8 + x^2 + x + 1` = **0x07** ;
- firmware modernes : réponse étendue à 15 octets avec état des boutons de powerbase.

Le module PCLC `rcssda6.pbd` contient bien :

- `auf_fill_crc8_lookup_table` ;
- `auf_crc8` ;
- vérification `CRC error` ;
- throttle, brake, lane change ;
- temps/lap ;
- fuel ;
- commandes envoyées à la powerbase ;
- programmation/affectation CarID ;
- intégration Pit-Pro, SmartSensor, capteurs C7030 ;
- intégration contrôleurs Scorpius ;
- PIT déduit de combinaisons Lane Change / Brake / zéro throttle et PIT OUT sur seuil de throttle ;
- limite de throttle en pitlane.

### Valeurs protocole utiles

La documentation C7042 fournit un timer S/F 32 bits avec pas d'environ **6,4 µs** et un CRC explicite. Les données contrôleur encodent brake, lane-change et puissance/throttle. C'est donc un très bon candidat pour un pilote natif de notre futur logiciel.

**Implémentable sans reverse engineering supplémentaire.**

---

## 4.12 Scalextric C7030 / PB-Pro

`rcsdpro.pbd` :

- interface dédiée C7030/PB-Pro ;
- support de plusieurs powerbases ;
- throttle, brake, lane change, fuel, laps ;
- Pit-Pro et capteurs externes ;
- commandes vers la powerbase.

La logique fonctionnelle est bien visible mais le framing série exact doit encore être établi à partir de la documentation PB-Pro ou d'une capture.

---

## 4.13 SCX Digital — interfaces SEB

Modules `rcscx.exe`, `rcscxdup.exe`, `sebmode3.pbd`.

Fonctions retrouvées :

- voiture / passage ;
- temps de tour ;
- throttle ;
- lane change ;
- fuel SCX ou fuel calculé par PCLC ;
- PIT déduit du bouton Lane Change et/ou d'un maintien à throttle zéro ;
- PIT OUT lorsqu'un seuil de throttle est dépassé ;
- pause/reprise possible via bouton LIGHT ;
- plusieurs interfaces possibles pour étendre le nombre de voitures.

Le module USB (`sebmode3`) expose 38400 et 115200. Les autres modules contiennent aussi plusieurs configurations série/parité. **Ne pas figer un baud/parité unique avant capture d'un modèle réel.**

---

## 4.14 Carrera Digital 30352 — protocole riche et bidirectionnel

`rcca30352.pbd` est particulièrement révélateur.

PCLC reçoit :

- passage voiture ;
- temps fourni par la CU ;
- fuel ;
- PIT IN / PIT OUT ;
- power ;
- start, pause et autres états ;
- faux départ/événements de course.

Le module expose **38400** dans son écran de configuration.

PCLC envoie ou prépare des commandes nommées :

- `SETBRAKE` ;
- `SETFUELLEVEL` ;
- `RESETPOSITION` / `SETPOSITION` ;
- `POWEROFF` / `POWERON` ;
- `RESTART` ;
- `NBLAP47` / `NBLAP03` ;
- `PITLAPCOUNTON` / `PITLAPCOUNTOFF` ;
- `LIGHTON` / `LIGHTOFF` ;
- `PITLANESPEED` ;
- `MAXPOWER` ;
- `FUELPOWER` ;
- `BRAKESTRENGTH` ;
- `FUELLEVEL` ;
- position tower / position.

La documentation PCLC confirme que la CU transmet passages, start/pause/reprise, carburant et PIT, mais pas la quantité de throttle. L'accès PC se fait via PC-UNIT 30349 ou convertisseur série TTL approprié.

### Conclusion

Ce système ne doit pas être traité comme un simple détecteur : c'est un **bus de contrôle de course**. Notre architecture devra accepter qu'un adaptateur fournisse à la fois `LapEvent`, `PitEvent`, `TelemetryEvent` et `OutputCommand`.

Le framing Carrera brut reste à documenter complètement avant implémentation autonome.

---

## 4.15 Carrera 30342

`rccar132.pbd` : interface Carrera plus ancienne. On retrouve passage voiture, fuel et temps selon les options. La liaison est également de type série/TTL. À conserver comme adaptateur séparé du 30352 : ne pas supposer que les trames sont interchangeables.

---

## 4.16 Slot.it Universal Live Timing Box

`rcslotittb.pbd` s'identifie explicitement comme **“Slot.it Universal Live Timing Box interface for Pc Lap Counter”**.

Bauds proposés : 38400, 115200, 256000, 500000.

PCLC sait distinguer l'état de branchement :

- box uniquement connectée au PC ;
- box connectée au PC + contrôleur SCP ;
- box connectée au PC + Track Interface.

Données :

- lap time ;
- throttle ;
- fuel ;
- PIT IN / PIT OUT ;
- PIT éventuellement généré si le hand brake est maintenu > 2 s avec throttle nul.

Le système Slot.it Track Interface accepte des détecteurs physiques de type dead-strip/DS bridge et prétraite leur signal avant le Live Timing Box.

---

## 4.17 Slot.it oXigen

`rcslotito2.pbd` / `rcslotito2v4.pbd` :

- dongle USB exposé comme COM ;
- bauds proposés 38400 / 115200 / 256000 / 500000 ;
- réception lap, throttle, fuel, PIT, données contrôleur/voiture ;
- PCLC sait compter le tour à l'entrée PIT, à la sortie, ou ne pas le compter ;
- envoi de commandes : power, limites de puissance, freinage, fuel/pit speed, états de course, etc. ;
- PCLC prévoit jusqu'à 20 voitures fonctionnelles dans l'intégration oXigen publique.

Slot.it publie officiellement le protocole PC <-> dongle et présente oXigen comme une interface PC ouverte pour logiciels tiers.

### Conclusion

Pour notre logiciel, on doit **implémenter le protocole officiel**, pas émuler les paquets RF voiture/contrôleur. Le dongle reste la frontière matérielle. Vouloir remplacer immédiatement la radio propriétaire 2,4 GHz serait inutilement complexe.

---

## 4.18 Scorpius

`rcscorpius.pbd` :

- dongle/COM ;
- 38400, 115200, 256000, 500000 exposés ;
- jusqu'à 24 identités dans l'interface PCLC ;
- throttle ;
- Lane Brain IDs configurables pour PIT IN/PIT OUT ;
- fonctions d'initialisation et d'envoi vers Scorpius.

Le système réel est RF 2,4 GHz bidirectionnel. Comme pour oXigen, le bon niveau de compatibilité pour notre projet est **le dongle/PC**, pas la couche radio propriétaire en première intention.

---

## 4.19 Davic

`rcdavic.exe` :

- PCLC configure **19200** dans l'interface ;
- flux principal + possibilité d'un module fuel sur un autre COM ;
- voiture ;
- fuel ;
- PIT IN / PIT OUT ;
- détection faux départ.

Le framing Davic reste à capturer.

---

# 5. Systèmes à transpondeur / identification

## 5.1 RfLapCounter — cas particulièrement simple

Le récepteur crée un port COM virtuel. Une description publique du protocole indique :

- code transpondeur sur **16 bits** ;
- plage `0x0000` à `0xFFFE` ;
- valeur reçue = valeur hexadécimale du numéro du transpondeur ;
- le transpondeur répète son code **9 fois sur environ 150 ms** pour la fiabilité ; une seule occurrence suffit pour valider un passage ;
- pas de commande de validation/handshake nécessaire après connexion.

Le module PCLC `rflc.exe` fonctionne en 8N1 et expose 9600/38400 selon configuration/version.

### Pour notre moteur

Il faut absolument un **deduplicate window** spécifique au protocole : les répétitions radio ne doivent produire qu'un seul `LapEvent`.

L'ordre exact des octets du 16 bits doit encore être validé avec une trame réelle avant codage final.

---

## 5.2 I-Lap RC

`ilaprc.exe` :

- 9600 / 38400 présents ;
- ouverture série avec **8N2** dans le chemin principal ;
- transponder ID ;
- temps au tour optionnel ;
- reset transpondeur ;
- mention d'un mode « 7 digit ».

Framing exact à capturer ou récupérer depuis documentation fabricant.

---

## 5.3 Trackmate

`rctrackmate.exe` :

- série **8N1** ;
- 9600 / 38400 présents ;
- détection lane/ID et temps ;
- PCLC envoie des réglages au Trackmate au démarrage ;
- filtres de mauvais temps au tour ;
- possibilité d'utiliser des capteurs S/F pour PIT.

Le matériel Trackmate utilise un décodeur connecté au PC (USB/COM) et des transpondeurs IR selon les générations.

---

## 5.4 AMB20 / AMB9200 / AMBrc

`amb20.exe` :

- série 8N2 ;
- 9600/38400 présents ;
- transponder ID + lap time ;
- reset.

`ambrc.pbd` demande explicitement de sélectionner le **protocole P3** sur le décodeur AMBrc.

Le détail P3 n'est pas présent en clair dans l'archive. Il faut documentation MyLaps/AMB compatible ou capture.

---

## 5.5 Robitronic

`robitronic.pbd` :

- **38400** ;
- ID transpondeur ;
- lap time ;
- reset de la liste/transpondeur.

Le framing n'est pas décodé.

---

## 5.6 TAG Heuer

`tagheuer.pbd` :

- 9600 / 19200 ;
- ID transpondeur ;
- lap time ;
- notion de boucle start et boucle PIT / bruit sur boucle ;
- commande « start » visible.

Le modèle exact de décodeur TAG Heuer n'est pas suffisamment identifiable depuis l'archive seule : **capture/documentation indispensable** avant développement.

---

## 5.7 Mantis II RFID

`mantis2.exe` est différent des autres : il utilise une connexion réseau vers un lecteur RFID.

Retrouvé :

- Reader IP ;
- port par défaut **6500** ;
- sensibilité du lecteur ;
- reset transpondeur ;
- réception de tag ;
- fragments de commandes `R,...` et `M,...` ;
- PCLC transforme ensuite le tag en passage/transponder event.

C'est un **adaptateur TCP**, pas COM. Une capture Wireshark est la méthode la plus simple pour terminer le protocole.

---

# 6. Ce que PCLC nous apprend pour notre propre architecture

## 6.1 Ne jamais confondre capteur, voie, voiture et transpondeur

Un LPT dit : « entrée 12 active ».  
Un DS peut dire : « voie 3, temps 8,421 s ».  
Un C7042 dit : « voiture ID 4 a franchi la ligne à tel compteur ».  
Un transpondeur dit : « ID 32572 vient de passer », sans notion intrinsèque de voie.

Le cœur doit donc conserver séparément :

- `sourceId` : adaptateur/boîtier ;
- `sensorId` : capteur physique ;
- `laneId` : voie, si elle existe ;
- `carId` : identité digitale, si elle existe ;
- `transponderId` : identité externe, si elle existe ;
- timestamp local et éventuellement timestamp fourni par le matériel ;
- données brutes originales pour audit/debug.

---

## 6.2 Modèle d'événements proposé

```text
LapEvent
  sourceId
  sensorId?
  laneId?
  carId?
  transponderId?
  capturedAtUs          // horloge monotone du boîtier/driver
  deviceTimestampUs?    // si le matériel donne son propre temps
  lapTimeUs?
  rawFrame?

PitEvent
  sourceId
  sensorId?
  laneId?
  carId?
  transponderId?
  direction = IN | OUT
  capturedAtUs

ControlEvent
  START | RESTART | PAUSE | RESUME | STOP | END
  POWER_ON | POWER_OFF | YELLOW | STOP_GO

TelemetryEvent
  carId?
  throttle?
  brake?
  laneChange?
  fuel?
  battery?
  status?

OutputCommand
  lights
  lanePower
  carPowerLimit
  brakeStrength
  fuelLevel
  pitSpeed
  stopGo
  ...
```

Une donnée brute ne doit être transformée qu'une fois. Le driver écrit l'événement canonique ; le chronométrage, l'affichage, le live timing et PCLC lisent ensuite cette même source.

---

# 7. Architecture du futur boîtier universel

## 7.1 Mauvaise approche

Un connecteur universel auquel on brancherait indifféremment LPT, RS-232, TTL, RS-485 ou une sortie de powerbase serait une très mauvaise idée. Les niveaux électriques et les topologies sont différents et on pourrait détruire un périphérique.

## 7.2 Bonne approche

### Cœur commun

- microcontrôleur avec timer matériel haute résolution ;
- timestamp réalisé **au plus près de l'entrée physique**, avant traitement PC ;
- firmware à pilotes/adaptateurs ;
- stockage de séquence + timestamp + source + frame brute ;
- USB device vers PC, éventuellement Ethernet ;
- watchdog et reprise après déconnexion ;
- configuration persistante versionnée.

### Cartes / ports spécialisés

1. **Digital Input** isolé/protégé : contacts secs, phototransistors, reed, Hall, IR, dead strip via conditionneur ;
2. **Digital Output** protégé : relais/MOSFET/optocoupleur pour power/feux ;
3. **RS-232** avec vrai transceiver ± tension ;
4. **TTL UART** avec niveaux sélectionnables/protégés 3,3/5 V ;
5. **RS-485 half-duplex isolé** pour C7042 et futurs bus ;
6. **Ethernet/TCP** pour lecteurs type Mantis et autres ;
7. **USB host** seulement quand réellement nécessaire ; pour les dongles propriétaires, il est souvent préférable de les laisser directement sur le PC et d'écrire un adapter logiciel.

### RF propriétaire

Ne pas chercher en V1 à remplacer les radios oXigen/Scorpius. Le boîtier/logiciel doit se connecter **au dongle ou au décodeur officiel**. Remplacer la couche RF ferait exploser la complexité sans bénéfice immédiat.

---

# 8. Interface logicielle recommandée

Chaque protocole doit être un `DetectorAdapter` indépendant :

```text
DetectorAdapter
  probe()
  connect()
  disconnect()
  capabilities()
  onRawFrame()
  emit(LapEvent | PitEvent | ControlEvent | TelemetryEvent)
  send(OutputCommand)
```

Exemples :

```text
RawGPIOAdapter
PclcArduinoAdapter
LptLegacyAdapter
PhidgetAdapter
DS030Adapter
DS200Adapter
DS300Adapter
DS0045Adapter
C7042Adapter
Carrera30352Adapter
OxigenAdapter
ScorpiusAdapter
RfLapCounterAdapter
TrackmateAdapter
...
```

Le cœur du chrono ne contient **aucune** condition du type `if detector == Carrera`.

---

# 9. Mode PCLC du futur boîtier

Le boîtier universel devrait exposer deux protocoles en parallèle :

### 9.1 Mode natif

Protocole moderne avec :

- timestamps en microsecondes ;
- numéro de séquence ;
- horloge monotone ;
- capabilities ;
- diagnostic ;
- événements structurés ;
- checksum/CRC ;
- possibilité d'enregistrer/rejouer les frames pour tests.

### 9.2 Mode « PCLC Arduino »

Une seconde interface COM virtuelle traduit immédiatement les événements :

```text
LapEvent(lane=3)                  -> [SF03]
LapEvent(lane=3, lap=8.421s)     -> [SF03$8421]
PitEvent(lane=3, IN)             -> [PI03]
PitEvent(lane=3, OUT)            -> [PO03]
LapEvent(transponder=32572)       -> [TR32572]
```

Et les commandes PCLC `[PW]`, `[SL]`, etc. sont reconverties en `OutputCommand` interne.

**Résultat : notre matériel reste utilisable avec PCLC, même quand notre propre logiciel de course sera prêt.**

---

# 10. Priorité d'implémentation recommandée

## Phase 1 — définir le standard interne

1. `EventBus` et modèle `Lap/Pit/Control/Telemetry` ;
2. timestamp monotone haute résolution ;
3. traces brutes/replay ;
4. dédoublonnage configurable ;
5. tests déterministes.

## Phase 2 — compatibilité facile et vérifiable

1. **PCLC Arduino** ;
2. **entrées digitales brutes** (remplace logique LPT/Phidget/K8000) ;
3. **C7042** ;
4. **RfLapCounter** ;
5. **oXigen via protocole dongle officiel**.

## Phase 3 — protocoles riches partiellement connus

1. Carrera 30352 ;
2. DS-030/200/300/0045 ;
3. Slot.it Live Timing Box ;
4. Scorpius ;
5. Ninco / SCX / Davic.

## Phase 4 — legacy/transpondeurs plus rares

Trackmate, I-Lap, AMB, Robitronic, TAG Heuer, RaceControl, exoTIC, XLOT, Mantis II.

La priorité peut évidemment changer si nous avons physiquement un de ces appareils : un protocole capturable aujourd'hui peut passer devant un protocole théoriquement plus important mais sans matériel.

---

# 11. Méthode pour terminer les protocoles encore opaques

Pour chaque système B/C, utiliser PCLC comme **oracle de référence** et capturer une séquence contrôlée.

### Scénario de capture standard

1. connexion / power on ;
2. 10 s idle ;
3. passage unique voiture/voie 1 ;
4. même chose sur chaque voie/ID ;
5. deux passages avec intervalle connu ;
6. PIT IN ;
7. PIT OUT ;
8. start ; pause ; resume ; stop ;
9. power on/off ;
10. fuel 100/50/0 si disponible ;
11. throttle 0/25/50/100 si disponible ;
12. lane change / brake ;
13. Stop&Go / Yellow ;
14. déconnexion/reconnexion.

### Outils par transport

- RS-232/TTL/RS-485 : logic analyzer ou sniffer série à haute impédance ;
- USB : USBPcap/Wireshark ou interception au niveau COM si CDC/FTDI ;
- TCP : Wireshark ;
- LPT/GPIO : logic analyzer ;
- RF propriétaire : capturer côté dongle/PC, pas la RF en première intention.

Chaque capture doit être stockée avec : modèle matériel, firmware, réglage baud/parité, action humaine exacte et horodatage.

---

# 12. Ce qui est déjà suffisant pour commencer le développement

Nous n'avons **pas besoin de connaître tous les octets de tous les vieux matériels avant de démarrer**.

Nous avons déjà assez d'informations pour figer correctement :

- la séparation capteur / voie / voiture / transpondeur ;
- l'EventBus ;
- l'API `DetectorAdapter` ;
- le moteur de timestamp ;
- la compatibilité PCLC Arduino ;
- le pilote raw digital ;
- la couche RS-232 / TTL / RS-485 ;
- le système de trace/replay qui permettra ensuite d'ajouter les protocoles sans toucher au chrono.

C'est exactement l'ordre à suivre si l'objectif est un logiciel durable et un boîtier réellement universel.

---

# 13. Points qui restent à établir précisément

Aucune de ces zones d'ombre ne doit être comblée par supposition :

- trames exactes DS-030/200/300/0045 ;
- framing Carrera 30352 et 30342 ;
- protocole complet C7030/PB-Pro ;
- variantes Ninco et SCX ;
- Scorpius dongle ;
- Davic ;
- XLOT ;
- exoTIC ;
- Trackmate ;
- I-Lap ;
- AMB P3 ;
- Robitronic ;
- TAG Heuer ;
- RaceControl `jnutl.dll` ;
- commandes exactes Mantis II ;
- ordre des octets RfLapCounter 16 bits sur une capture réelle.

Pour ceux-là, écrire du code aujourd'hui en devinant le paquet serait du mauvais reverse engineering. Il faut une doc officielle ou une capture.

---

# 14. Fichiers de l'archive particulièrement utiles

```text
racectrl.ini
phidget.ini
rclpt.exe
PCLCLPT1.exe
DLPORTIO.dll / DLPORTIO.sys
rcserver.exe / K800032.dll          # K8000
arduino.exe
rcds.exe / rcds200.exe / rcds300.exe / rcds045.exe
rcrms.exe
rcrc.exe / jnutl.dll
rcexotic.pbd
rcxlot.exe
rcninco.exe / rcninco108.exe
rcsdpro.pbd
rcssda6.pbd
sebmode3.pbd / rcscx.exe / rcscxdup.exe
rccar132.pbd
rcca30352.pbd
rcslotittb.pbd
rcslotito2.pbd / rcslotito2v4.pbd
rcscorpius.pbd
rcdavic.exe
rctrackmate.exe
rflc.exe
ilaprc.exe
amb20.exe / ambrc.pbd
robitronic.pbd
tagheuer.pbd
mantis2.exe
```

Les PBD de cette archive portent l'en-tête **PowerBuilder 6 (`0600`)** ; leurs objets internes ont été extraits pour l'analyse sans modifier l'archive d'origine.

---

# 15. Sources publiques recoupées

- PC Lap Counter — protocole Arduino v1.06 : `https://www.pclapcounter.be/PcLapCounter_arduino_protocol.pdf`
- PC Lap Counter — DS Racing : `https://www.pclapcounter.be/body_ds_racing_ds-030_ds-300_ds-200.html`
- PC Lap Counter — DS-0045 : `https://pclapcounter.be/body_ds_racing_ds-0045_fr.html`
- Scalextric C7042 SNC protocol : documentation rendue publique par Hornby / archives SSDC.
- Slot.it — oXigen : documentation et protocole PC-dongle officiels, interface PC ouverte.
- RfLapCounter : documentation fabricant + description publique du protocole 16 bits.
- Pages PCLC/Slot Car-Union actuelles pour Carrera 30352 et oXigen, utilisées comme recoupement fonctionnel.

---

## Conclusion

Le projet de « boîtier universel » est techniquement réaliste **si on le conçoit comme un concentrateur modulaire de protocoles et non comme un détecteur unique**.

La pièce centrale n'est pas le connecteur : c'est le **modèle métier et événementiel propre à notre logiciel, avec timestamp fiable**. Les protocoles externes ne dictent jamais ce modèle : ils sont décodés puis traduits vers lui. Une fois ce cœur fixé, chaque ancien système devient un adaptateur remplaçable. Le boîtier peut ensuite agréger les interfaces électriques réellement pertinentes, tandis que les dongles USB/RF propriétaires restent gérés côté PC quand c'est plus rationnel.

Le protocole PCLC Arduino fournit dès maintenant une sortie de compatibilité simple et documentée : on peut donc construire notre propre système sans attendre le décodage complet de tous les anciens matériels et rester compatible avec PC Lap Counter pendant la transition. **PC Lap Counter reste ici une source d'expérience, de cas réels et d'interopérabilité ; l'objectif n'est pas de le reproduire fonction pour fonction.**
### Comportement PIT observé à conserver comme référence

Retour d'usage PCLC : l'entrée dans les stands n'est pas assimilée naïvement à un simple front de capteur. Une fois `PIT IN` engagé et la condition d'entrée satisfaite, l'absence de `PIT OUT` maintient le concurrent dans l'état PIT et permet, selon les règles configurées, de commencer le ravitaillement.

Cela rejoint les paramètres retrouvés dans les modules d'E/S brutes (`PITIN_MIN`, `PITOUT_MIN`, maintien/dwell et sortie au relâchement selon configuration).

À reprendre comme **comportement fonctionnel de référence**, sans imposer l'implémentation interne de PCLC à Lab-Traks.
