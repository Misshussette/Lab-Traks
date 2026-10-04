# Modèle de données — principes

Ce document ne constitue pas encore le schéma physique de base de données. Il fixe les règles avant de choisir les tables et relations définitives.

## Règle fondamentale

Une donnée métier existe une seule fois.

Elle est ensuite référencée par les courses, championnats, configurations et résultats.

## Entités de référence envisagées

Le modèle devra au minimum distinguer les concepts suivants :

- **Pilote**
- **Club / organisation**
- **Circuit**
- **Configuration de circuit**
- **Voie**
- **Véhicule**
- **Catégorie / classe**
- **Matériel de détection**
- **Profil de connexion / protocole**
- **Contrôleur / poignée** lorsque pertinent
- **Événement**
- **Championnat**
- **Course**
- **Session / manche / segment**
- **Inscription / participation**
- **Stint**
- **Passage**
- **Tour**
- **Split / secteur**
- **Pénalité / incident**
- **Profil d'affichage**
- **Traduction**

Les noms et frontières pourront évoluer pendant la modélisation.

## Pilotes

Un pilote n'est pas recréé pour chaque course.

Une course référence les pilotes existants via ses inscriptions/participants.

### Sous-listes

Le système doit permettre de préparer une course à partir d'un sous-ensemble de la base :

```text
Base pilotes
   ↓ sélection / import
Liste de candidats de l'événement
   ↓
Inscriptions / participants de la course
```

Cette sélection ne crée pas une seconde base pilotes.

## Import

Lors d'un import de liste :

1. lire les données ;
2. normaliser les champs utiles ;
3. rechercher une correspondance exacte ;
4. rechercher les correspondances approximatives pertinentes ;
5. signaler les cas ambigus ;
6. réutiliser l'entité existante ou créer une nouvelle entité ;
7. créer uniquement les relations nécessaires avec l'événement/course.

Le système ne doit pas créer silencieusement des doublons parce qu'un prénom, une casse ou une orthographe diffère légèrement.

## Circuit et piste

Le circuit doit pouvoir porter son contexte de référence, notamment :

- nom ;
- club/lieu ;
- discipline/type ;
- nombre de voies ;
- longueurs de voies si connues ;
- configuration physique ;
- systèmes de détection associés.

Une configuration temporaire de course doit référencer le circuit plutôt que dupliquer ses informations permanentes.

## Matériel

Le matériel doit être décrit séparément de la course :

- fabricant/modèle ;
- identifiant local ;
- transport ;
- protocole ;
- version/firmware si utile ;
- paramètres de communication ;
- capacités ;
- mapping capteurs/voies ;
- calibration/configuration.

Une course sélectionne un profil matériel existant.

## Passage, tour et temps

Il faut distinguer la donnée brute de ce qui est calculé.

Exemple conceptuel :

```text
Événement matériel brut
       ↓
Passage interprété
       ↓
Tour / split / PIT / autre événement sportif
```

Cela permet :

- l'audit ;
- le recalcul ;
- le replay ;
- le diagnostic des faux passages ;
- la correction sans perdre la source.

## Données anormales

Les analyses futures doivent pouvoir distinguer données brutes et données exclues/qualifiées.

Un tour peut par exemple être marqué comme anormal pour :

- passage par les stands ;
- réparation ;
- accident ;
- erreur de détection ;
- seuil impossible selon le circuit.

La donnée source n'est pas effacée simplement pour rendre les statistiques plus jolies.

## Historique

Toute correction importante de course doit être traçable.

Le niveau précis d'historisation sera défini lors du schéma détaillé.

## Base technique

Le moteur de persistance n'est pas encore choisi. SQLite est une possibilité naturelle à évaluer pour un fonctionnement local léger, mais ce n'est pas une décision actée.
