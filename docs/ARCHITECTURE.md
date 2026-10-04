# Architecture générale

## Objectif

Construire un système dont le moteur de course reste stable même lorsque l'on ajoute ou remplace un matériel, un protocole, une interface ou un type d'affichage.

## Chaîne générale

```text
Capteur / système existant
        ↓
Interface électrique / transport
        ↓
Driver + décodeur de protocole
        ↓
Adaptateur Lab-Traks
        ↓
Événements métier Lab-Traks
        ↓
Race Engine
        ↓
Persistance / journal / replay
        ↓
UI opérateur / affichages / services optionnels
```

## 1. Couche matériel et transport

Elle gère ce qui est physique ou transport :

- entrées digitales ;
- LPT historique ;
- RS-232 ;
- UART TTL ;
- RS-485 ;
- USB ;
- Ethernet/TCP ;
- SDK ou DLL fabricant ;
- dongles radio propriétaires.

Cette couche ne décide pas qu'un passage vaut un tour.

## 2. Driver / décodeur

Un driver connaît le protocole externe.

Exemples :

- lire une trame DS ;
- décoder un événement Carrera ;
- recevoir un ID de transpondeur ;
- lire l'état d'une entrée ;
- envoyer une commande de puissance.

Le driver peut conserver les octets ou valeurs brutes utiles au diagnostic.

## 3. Adaptateur Lab-Traks

L'adaptateur traduit le monde extérieur vers notre modèle.

Il doit exposer des **capacités**, par exemple :

- détection de passage ;
- temps matériel ;
- identité voiture ;
- transpondeur ;
- PIT IN/PIT OUT ;
- throttle ;
- bouton ;
- carburant ;
- contrôle de puissance ;
- feux ;
- Stop&Go ;
- commande voiture ;
- télémétrie.

Le moteur demande une capacité ; il ne demande jamais « une commande Carrera » ou « une fonction PCLC ».

## 4. Modèle d'événements interne

Le vocabulaire appartient à Lab-Traks.

Les événements externes peuvent avoir des noms et formats totalement différents : ils sont traduits.

Un événement interne pourra porter selon le besoin :

- identifiant de source ;
- identifiant de capteur ;
- voie ;
- véhicule ;
- pilote ;
- transpondeur ;
- timestamp matériel ;
- timestamp PC ;
- temps fourni par le matériel ;
- qualité/statut ;
- payload brut de diagnostic ou référence vers celui-ci.

Le schéma exact sera figé avec le modèle de données et les premiers prototypes.

## 5. Race Engine

Le Race Engine est l'autorité sur l'état sportif.

Il gère notamment :

- état de course ;
- tours ;
- temps ;
- secteurs/splits ;
- classement ;
- drapeaux et pauses ;
- stands ;
- changements de pilote ;
- pénalités ;
- règles de course ;
- événements de course.

Il ne dépend pas de l'interface graphique.

## 6. Temps et concurrence

Le chemin critique de détection doit être indépendant du thread/UI.

Principes :

- l'événement est timestampé au plus près de sa réception ;
- le traitement graphique est asynchrone par rapport au chrono ;
- les événements sont ordonnés et traçables ;
- les doublons matériels peuvent être filtrés sans perdre la trace brute ;
- les traitements non critiques ne doivent jamais bloquer la détection.

## 7. Persistance et reprise

L'état courant doit être sauvegardé de façon continue.

Il faut pouvoir reconstruire :

- l'état de course ;
- les tours validés ;
- les passages bruts pertinents ;
- les corrections opérateur ;
- la configuration matérielle ;
- les participants ;
- le contexte de course.

Le mécanisme précis de stockage n'est pas encore choisi.

## 8. Trace et replay

Chaque driver sérieux doit pouvoir être testé sans disposer en permanence du matériel réel.

Objectif :

1. capturer des échanges réels ;
2. stocker les captures avec leur contexte ;
3. les rejouer dans le driver ;
4. vérifier que la même séquence produit le même résultat.

Cela deviendra une partie essentielle des tests de non-régression.

## 9. Interface

L'UI reçoit l'état du moteur et émet des intentions opérateur.

Elle ne doit :

- ni calculer les temps officiels ;
- ni interpréter directement les ports COM ;
- ni contenir les règles d'un protocole matériel.

Un gel de l'UI ne doit pas stopper le chronométrage.

## 10. Services réseau

Tout service réseau est optionnel vis-à-vis du chrono local.

Exemples futurs possibles :

- live timing ;
- consultation mobile ;
- synchronisation ;
- téléchargement de mises à jour ;
- services externes.

La course locale reste fonctionnelle sans eux.

## 11. Principe d'extension

Ajouter un nouveau système de détection doit principalement signifier :

- ajouter un driver/adaptateur ;
- déclarer ses capacités ;
- ajouter ses tests et captures ;
- ajouter sa configuration UI.

Cela ne doit pas nécessiter de modifier le Race Engine pour chaque fabricant.

## 12. Points de chronométrage

Le modèle du Race Engine ne doit pas être limité à une unique ligne départ/arrivée.

Un parcours peut exposer plusieurs **points de chronométrage** :

- départ/arrivée ;
- split 1, split 2, etc. ;
- entrée stands ;
- sortie stands ;
- autres points qualifiés selon le format.

Le matériel peut fournir ces événements par des technologies différentes. Le Race Engine reçoit des événements normalisés et ne doit pas dépendre de l'origine physique du point.

## 13. Affichages comme consommateurs de données

Les affichages consomment l'état calculé par le Race Engine et ne possèdent pas leur propre vérité métier.

Une configuration d'affichage est une donnée pérenne indépendante de l'écran physique. Elle doit pouvoir être :

- sauvegardée ;
- rouverte ;
- modifiée ;
- dupliquée ;
- exportée/importée si pertinent ;
- associée à un écran ou un usage ;
- réutilisée lors d'un prochain lancement.

Les vues peuvent notamment filtrer la même source de données vers :

- un classement global ;
- les participants d'une piste donnée ;
- d'autres vues spécialisées à définir.

Le choix d'une technologie d'affichage déporté n'est pas encore acté.

## 14. Services externes

Le cœur Lab-Traks doit pouvoir exposer ou exporter ses données sans dépendre d'un service distant.

Les futurs services comme StintLab ou Slot Hub doivent utiliser des contrats d'échange propres et documentés plutôt que devenir propriétaires du modèle interne ou de la base locale.

Cette séparation doit permettre :

- d'utiliser Lab-Traks sans aucun service Web ;
- d'utiliser certains services Web sans Lab-Traks lorsque cela a du sens ;
- de remplacer un service externe sans réécrire le Race Engine.
