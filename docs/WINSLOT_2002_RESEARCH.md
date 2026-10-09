# Étude WinSlot 2002 — compatibilité DS et décodage série

> **Statut : recherche statique, non validée par capture matérielle.**  
> Source fournie : archive `ControlEntrenos.zip`, analysée le 9 octobre 2026.  
> Ne pas intégrer de code propriétaire ni redistribuer les exécutables dans Lab-Traks.

## 1. Identification

- Produit : **WinSlot 2002**, journal d'installation indiquant **1.9.26a**.
- Une chaîne de l'exécutable mentionne **DS-Winslot Professional Software** et le site DS Racing Products.
- Exécutable principal : `WinSlot2002.exe` (4 509 696 octets), **PE32 Windows x86, Visual Basic 6**, import `MSVBVM60.DLL`.
- Journal d'installation `ST6UNST.LOG` : présence du composant **`MSCOMM32.OCX`** (communication série) et de **DAO 3.5 / Jet Access**.
- Contrôles personnalisés :
  - `ControlEntrenos.ocx` — affichage d'entraînement et liste des tours ;
  - `ControCarril.ocx` — affichage de voie ;
  - `Controlinter.ocx` — affichage de course, pilote, dossard, temps et tours.
- Bases `.mdb` :
  - `Slot.mdb` — entités repérées : `Pilots`, `Clubs`, `Equips`, `Pistes`, `Curses`, `Cotxes`, `PilotsEquip`, `TipusCursa` ;
  - `Bases/CarreraVPT.mdb` — `Config`, `EstadoCarrera`, `General`, `Historial`, `Manga`, `MangaCrono`, `PilotsCursa`, `Resum` ;
  - `Bases/CarreraVET.mdb` — notamment `Historial`, `PilotsCursa` ;
  - `Bases/Entrena.mdb` — données d'entraînement ;
  - `Bases/EntrePilot.rpt` — rapport Crystal Reports.
- Présence de fonctions d'import/export Excel et CSV et de corrections manuelles de tours.

Empreintes SHA-256 pour la traçabilité :
- Archive : `48ef0ea52ee32317b753b3ecc78d7b8099fe384ecf834ec289afb8bddaafc2de`
- Exécutable : `181f027656011baa659dcdec603ada4e2960245ce9c88bfa4f7b4fb8a5709a5f`

## 2. Matériels identifiés dans l'exécutable

Chaînes UTF-16 présentes :

- `DS 200`
- `DS 300`
- `DS 4 PORTS`
- `DS INTERFACE`
- `DS INTERFACE PRO`

L'interface présente des ports `COM1` à `COM8`, un contrôle `cmbPort`, deux objets `MSComm1` et `MSComm2`, ainsi que les fonctions de diagnostic :

- `Probar puerto serie` ;
- `Interpreta Bytes` ;
- `Listar Bytes` ;
- `DebugBytes` ;
- `Resetear puerto`.

**Conclusion étayée :** WinSlot 2002 communique par liaison série avec plusieurs générations de chronomètres DS et possède une logique interne d'interprétation des octets.

**Non établi :** débit, parité, bits d'arrêt et différences exactes entre modèles. Le fait qu'un composant MSComm soit utilisé ne suffit pas à les déduire.

## 3. Reconstruction statique d'une trame candidate

L'examen de la routine native de réception/décodage révèle des comparaisons explicites :

- `0xE0` : testé à l'adresse virtuelle **0x679449** et à **0x679FC0** ;
- `0xEB` : testé à **0x6797D6** et à **0x67A122** ;
- compteur de construction comparé à `0x14` (20 décimal) à **0x67A2C9** ;
- buffer indexé de **0 à 20** (21 positions).

Une branche remet l'index à zéro lors de `0xE0`, une autre le place à 20 lors de `0xEB`. La structure probable est donc :

```text
index : 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20
octet : E0 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? EB
```

> **Important :** cette reconstruction concerne une routine identifiable de l'exécutable. Elle ne prouve pas que tous les matériels et modes DS utilisent exactement cette trame, ni que le checksum a été correctement compris.

### Indications fournies par la routine de diagnostic

Le programme construit des lignes de debug contenant les libellés suivants. Les indices sont déduits des accès au buffer proches de ces libellés, **pas encore vérifiés avec des trames réelles** :

| Index | Indication observée | Confiance |
| --- | --- | --- |
| 0 | marqueur `0xE0` | forte pour cette routine |
| 1 | `Nº de contador` ; également comparé à une valeur mémorisée | forte pour le champ ; rôle exact à confirmer |
| 2 | non identifié | inconnue |
| 3 | `Modelo de DS Lap Counter` | forte pour le libellé |
| 4–7 | `4 Ports Canal` / `No es 4 Port` ; branches sur valeurs de canal | moyenne |
| 8 | états : phases de course, fin, pause, abandon, leader | moyenne |
| 9 | `Volta Rápida` / `Volta Normal` | moyenne |
| 10 | `Nº de carril` | forte |
| 11–12 | `Nº de Volta` (deux positions lues) | moyenne |
| 13 | `Horas` | forte |
| 14 | `Minutos` | forte |
| 15 | `Segundos` | forte |
| 16 | `Centesimas` | forte |
| 17 | `Milesimas` | forte |
| 18 | `CheckSum` | forte pour le libellé, algorithme inconnu |
| 19 | `Valor Suma de Bytes` | forte pour le libellé, algorithme inconnu |
| 20 | marqueur `0xEB` | forte pour cette routine |

Les libellés des états sont présents dans le code : `1ª Fase de Carrera`, `2ª Fase de Carrera`, `3ª Fase de Carrera`, `Fin de Carrera`, `Inicio Pausa`, `Fin Pausa`, `Carrera Abortada`, `Va Lider`, `Volta Rápida`, `Volta Normal`.

Il ne faut **pas** conclure sans analyse complémentaire que les indices 8 et 9 sont des champs simples ou que chaque état est représenté par un bit indépendant.

### Hypothèse de synchronisation de trame

Un décodeur expérimental pourrait d'abord :
1. journaliser les octets bruts reçus ;
2. rechercher `E0` et `EB` ;
3. observer les trames de 21 octets ;
4. comparer les champs supposés aux données affichées par le chronomètre ;
5. tester les séquences de tours, les changements de voie, les pauses et les erreurs ;
6. vérifier l'algorithme de checksum avant d'accepter un passage.

**Ne pas** déployer un parseur de production basé uniquement sur ces hypothèses. Le rôle de `0xE0`/`0xEB` comme délimiteurs et les cas d'échappement doivent être confirmés.

## 4. Éléments sportifs et données

Les bases et chaînes SQL montrent notamment :

- historique de passages/tours : `Historial`, `Vuelta`, `Tiempo`, `Tiempototal`, `Carril`, `Manga` ;
- pilotes et équipes : `PilotsCursa`, `CodiPilot`, `NomEquip` ;
- changements de voie : `CambioCarril`, `CarrilActual` ;
- classements : meilleurs temps, total de tours, moyenne et cumul ;
- modes repérés : vitesse à tours, vitesse au temps, endurance par équipes au temps, manches chronométrées ;
- opérations manuelles : ajouter/retirer un tour, répéter une voie/manche, modifier une manche.

Cela sert à comprendre les cas d'utilisation et les données à préserver, **pas** à copier le schéma Access dans Lab-Traks.

## 5. Conséquences pour Lab-Traks

- Ajouter WinSlot 2002 comme **source de recherche** pour les drivers DS historiques.
- Ne pas confondre le protocole du matériel DS avec l'architecture interne de WinSlot.
- Le driver Lab-Traks devra isoler **transport série → synchronisation → validation de trame → décodage du modèle → événements canoniques**.
- Prévoir des fixtures de trames réelles, un journal binaire horodaté, le replay et des tests de corruption/perte/doublons.
- Si le matériel DS fournit un temps de passage, conserver sa valeur et l'heure de réception PC comme deux informations distinctes.
- Les fonctions sportives WinSlot (rotation de voie, manches, équipes) doivent être implémentées au niveau métier, indépendamment du driver.

## 6. À obtenir avant implémentation fiable

1. Une **capture série brute** réelle pour chaque modèle DS accessible (DS 200, DS 300, DS 4 PORTS, DS INTERFACE/PRO).
2. Les paramètres physiques de liaison (baud, parité, stop, contrôle de flux).
3. Des séquences connues : départ, un tour sur chaque voie, deux passages simultanés, pause, reprise, fin, déconnexion.
4. La formule exacte du contrôle de trame et le comportement en cas de perte d'octets.
5. Les différences de firmware et de modèles.
6. Les horodatages : temps absolu du boîtier, temps au tour, temps depuis départ, granularité réelle.

## 7. Limites de l'étude

Analyse **statique** uniquement : fichiers inspectés, chaînes et portions de code machine désassemblées ; **aucun exécutable/OCX lancé**, aucun matériel DS connecté, aucune capture série fournie. Les interprétations du format doivent rester des **hypothèses documentées** jusqu'aux tests matériels.

L'objectif est l'**interopérabilité**, non la reproduction de WinSlot 2002.
