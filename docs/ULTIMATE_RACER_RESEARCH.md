# Ultimate Racer 3.0 — étude de référence pour Lab-Traks

## Statut

**Source analysée :** archive utilisateur `Database.zip`.

Cette documentation décrit uniquement des éléments vérifiés dans l'archive. Ultimate Racer 3.0 (UR30) est étudié comme **source d'idées et de compréhension**, au même titre que PC Lap Counter. Son modèle n'est pas le modèle Lab-Traks.

## Identification

L'archive contient notamment :

- `Racer30.exe` ;
- `URWebCam.exe` ;
- `racer30.chm` ;
- `sqlite3.dll` ;
- `phidget21.dll` ;
- `Database/SlotCar30.db` ;
- des bibliothèques de rails ;
- des exemples de circuits `.scc` ;
- des modèles de course ;
- des ressources 2D et 3D.

Le journal fourni indique :

```text
Revision 3.0.31b2-Feb 2014
```

Les chaînes du programme identifient explicitement **Ultimate Racer 3.0** et `www.uracer.org`.

## Pourquoi UR30 est intéressant pour Lab-Traks

UR30 réunit dans le même logiciel deux domaines qui nous intéressent :

1. **designer de circuit** ;
2. **gestion et chronométrage de course**.

Il constitue donc une référence complémentaire à PCLC.

---

# 1. Designer de circuit

## Formats de circuit

Les exemples utilisent l'extension `.scc`.

Ces fichiers sont des documents OLE/Compound Document créés par une application identifiée comme `DrawCli`.

Le contenu visible montre des objets de piste structurés : référence de rail, géométrie, voies, courbes, textures et autres propriétés.

## Bibliothèques de rails

L'archive contient des bibliothèques pour de nombreux systèmes, notamment :

- Airfix ;
- Artin ;
- Aurora ;
- Carrera ;
- Fleischmann ;
- Home Routed ;
- Jouef ;
- LifeLike ;
- Marchon ;
- MaxTrax ;
- Ninco ;
- Polistil ;
- Revell ;
- Scalextric ;
- SCX ;
- Strombecker ;
- Tomy AFX ;
- Tyco.

Des fichiers complémentaires existent pour les bordures et configurations multi-voies.

## Fonctions de designer vérifiées

Les ressources UI montrent notamment :

- édition de circuit ;
- bibliothèque de rails ;
- insertion/remplacement/suppression de rail ;
- rotation ;
- changement d'angle ;
- déplacement d'une portion ;
- élévation/abaissement ;
- sens de roulage ;
- affichage des références et noms de rails ;
- propriétés du circuit ;
- stock de rails ;
- vue 2D/3D ;
- caméra 3D ;
- modèles 3D ;
- objets de décor ;
- publication du circuit sur le Web.

## Données de circuit

La base SQLite distingue notamment :

### `Circuit`

Informations telles que :

- nom ;
- temps au tour minimum ;
- unité de vitesse ;
- unité de longueur ;
- image ;
- tension ;
- échelle ;
- type de système ;
- gestion des stands ;
- nombre de sections ;
- configuration d'écran de course.

### `Lane`

Informations telles que :

- numéro de voie ;
- circuit ;
- couleur ;
- longueur ;
- unité ;
- capteurs intermédiaires.

Cette séparation circuit/voie est intéressante : la longueur peut être connue **par voie**, ce qui est utile pour calculer vitesses et statistiques avec des circuits où les longueurs diffèrent.

## Lien designer ↔ chronométrage

Les ressources montrent qu'un tracé peut être rattaché à un circuit de chronométrage.

UR30 vérifie notamment la cohérence du nombre de voies et l'unicité de leurs couleurs.

Pour Lab-Traks, cela confirme l'intérêt de relier un futur designer au même objet `Circuit` sans dupliquer les informations.

---

# 2. Base de données de chronométrage

`SlotCar30.db` est une base SQLite.

Tables observées :

- `Administration`
- `Circuit`
- `Driver`
- `Hardware`
- `HeatEvent`
- `Lane`
- `Maintenance`
- `Point`
- `Race`
- `RaceType`
- `Score`
- `SlotCar`
- `Stock`
- `Tournament`

## Séparation des concepts

UR30 distingue explicitement :

- pilote ;
- voiture ;
- circuit ;
- voie ;
- matériel ;
- type de course ;
- course ;
- tournoi ;
- événement de manche ;
- score ;
- maintenance voiture.

C'est une bonne source de questions pour notre propre modèle, mais pas un schéma à recopier.

## Événements de manche

La table `HeatEvent` peut conserver notamment :

- pilote ;
- voie ;
- course ;
- voiture ;
- numéro de tour ;
- temps au tour ;
- longueur ;
- date/heure ;
- score ;
- vitesse ;
- temps de réaction ;
- section ;
- type d'événement ;
- équipe ;
- timestamp ;
- pourcentage de section.

Cela confirme l'intérêt de distinguer les **événements/passage** des résultats agrégés.

---

# 3. Formats de course

L'archive contient 29 fichiers de modèles/configurations de course.

On trouve notamment :

- Limited Time ;
- Limited Lap ;
- First Arrived ;
- Practice ;
- Time/Lap Staggered ;
- Race to the Line ;
- Round Robin ;
- IROC ;
- Shared Tournament ;
- All Against All ;
- qualification ;
- FOLM avec qualification + finale.

Les fichiers décrivent des concepts tels que :

- sélection des participants ;
- construction de grille ;
- rotations de voies ;
- nombre de tours ;
- durée ;
- délai inter-manche ;
- sections ;
- arrivée réaliste ;
- pénalités en tours ou temps ;
- carburant ;
- coupure carburant ;
- faux départ ;
- sorties de piste ;
- scores par vitesse, tour, place, réaction, faux départ, etc.

### Leçon pour Lab-Traks

Un format de course ne devrait probablement pas être seulement un `enum` du type « sprint/endurance ».

UR30 montre l'intérêt de composer un format avec plusieurs paramètres et règles. Lab-Traks devra cependant proposer cela de manière beaucoup plus simple et cohérente pour l'utilisateur.

---

# 4. Matériel et détection

La table `Hardware` contient 307 configurations/événements dans cette base.

## Familles identifiées

- LPT ;
- ports COM ;
- DS Race Manager ;
- DS300 ;
- DS045 ;
- RMS ;
- webcam/événement externe ;
- Phidget 1012 ;
- Phidget 1014 ;
- Phidget 1017 ;
- Phidget 1018 ;
- Ninco Power Base ;
- extensions Ninco Multi Lane ;
- Carrera Digital 1/32 ;
- Carrera Digital 1/24 ;
- Carrera Digital Control Unit ;
- SCX Digital ;
- Scalextric Digital ;
- Davic ;
- DGT Slot Digital ;
- joystick USB ;
- clavier USB ;
- souris USB.

Le journal d'initialisation confirme la création de composants dédiés pour plusieurs de ces systèmes.

## LPT

UR30 connaît explicitement :

- LPT1 : `0x378`
- LPT2 : `0x278`
- autres adresses configurables.

Les ressources indiquent la gestion des broches d'entrée/sortie et la base contient des mappings pour :

- détection voiture ;
- contrôle alimentation par voie ;
- boutons ;
- feux ;
- ravitaillement ;
- track call ;
- temps intermédiaires ;
- faux départ ;
- crash and burn ;
- décors/scenery.

UR30 gère des événements jusqu'à au moins **16 crews/voies logiques** dans la configuration fournie.

## Événements et sorties

La configuration ne se limite pas à « une voiture passe ».

Elle contient aussi :

- PIT IN/PIT OUT ;
- carburant faible ;
- réservoir plein ;
- ravitaillement ;
- meilleur tour/vitesse ;
- pire tour/vitesse ;
- pénalité ;
- pause/reprise ;
- feux de départ ;
- dernier tour ;
- track call ;
- coupure/alimentation piste.

Cela renforce notre décision Lab-Traks de travailler avec des **capacités matérielles** et un vocabulaire métier interne, plutôt qu'avec une API limitée au comptage de tours.

---

# 5. Sections / splits

UR30 gère explicitement :

- des capteurs intermédiaires par voie ;
- une distance entre ligne de départ et capteur intermédiaire ;
- des événements « Intermediate lap time » ;
- un champ `Section` dans `HeatEvent` ;
- un nombre de sections au niveau du circuit.

C'est une référence utile pour le futur modèle de splits Lab-Traks.

---

# 6. Pilotes et voitures

La base distingue `Driver` et `SlotCar`.

Le pilote peut porter identité, pseudo, club, équipe, discipline, niveau, photo, etc.

La voiture peut porter notamment :

- marque ;
- référence ;
- catégorie ;
- numéro ;
- fabricant ;
- propriétaire ;
- transmission ;
- moteur ;
- pneus ;
- réservoir ;
- consommation ;
- fiabilité ;
- dimensions ;
- poids ;
- état ;
- prix/date d'achat ;
- Digital ID.

Lab-Traks devra rester plus propre sur la séparation entre données permanentes et données propres à une participation/course.

---

# 7. Résilience / fonctionnement

Le journal montre une architecture avec threads dédiés et plusieurs vues matérielles.

Le programme utilise SQLite et contient également des mécanismes d'auto-save/recovery de documents.

Cela ne prouve pas que son moteur de course est résilient selon nos exigences modernes, mais fournit des pistes à examiner.

---

# 8. Points particulièrement intéressants à reprendre comme questions de conception

À étudier pour Lab-Traks :

1. relation **Circuit ↔ tracé graphique** ;
2. longueur **par voie** et pas seulement longueur globale ;
3. bibliothèque de rails réutilisable ;
4. inventaire/stock de rails ;
5. capteurs intermédiaires positionnés dans le tracé ;
6. séparation pilote / voiture / participation ;
7. formats de course composables ;
8. rotations de voies ;
9. tournoi multi-manches ;
10. scoring configurable ;
11. événements matériels entrants **et sortants** ;
12. support d'actions opérateur physiques : track call, ravitaillement, affichage ;
13. simulation/décor contrôlable par sorties matérielles ;
14. maintenance d'une voiture.

Aucun de ces points n'est automatiquement une décision Lab-Traks.

---

# 9. Différence de rôle avec PCLC

### PC Lap Counter

Très utile pour étudier :

- compatibilité de détection ;
- protocoles ;
- comportement du chronométrage ;
- écrans et fonctionnement terrain.

### Ultimate Racer 3.0

Particulièrement utile pour étudier :

- modèle circuit ;
- designer de piste ;
- bibliothèques de rails ;
- relation piste/chrono ;
- voitures ;
- formats de course et tournois ;
- rotations ;
- scoring ;
- I/O génériques.

Les deux logiciels sont des **sources complémentaires**, pas des cahiers des charges à reproduire.
