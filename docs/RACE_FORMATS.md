# Formats de course

## Principe

Lab-Traks ne doit pas avoir un seul type de course caché derrière différents écrans.

Le Race Engine doit séparer :

- ce qui définit une compétition ;
- ce qui définit une session/manche ;
- les règles sportives ;
- les capacités réellement offertes par le matériel.

## Formats à pouvoir accueillir

Le périmètre envisagé comprend notamment :

- course au nombre de tours ;
- course au temps ;
- sprint ;
- endurance ;
- relais / changements de pilotes ;
- rallye ;
- drag race ;
- essais libres ;
- qualification ;
- challenges basés sur un objectif de tours/temps ;
- formats personnalisés.

Cette liste ne doit pas être transformée trop tôt en enum rigide impossible à étendre.

## Analogique et digital

Les règles de course sont indépendantes du type de piste.

Le matériel digital peut cependant offrir des capacités supplémentaires :

- identification véhicule ;
- lane change ;
- fuel ;
- freinage ;
- limitation de vitesse ;
- contrôle de puissance ;
- télémétrie ;
- conditions simulées.

Le moteur doit appliquer uniquement ce que l'adaptateur matériel déclare pouvoir supporter.

## Simulation plus riche

À terme, certains formats pourront aller vers une gestion plus proche du sport automobile réel :

- météo/pluie ;
- pneus ;
- carburant ;
- dégâts/avaries ;
- pénalités ;
- drapeaux ;
- safety car ;
- règles WEC/F1/GT ou personnalisées.

Ces éléments ne doivent pas être imposés au MVP du chrono.

## Échelles

Le modèle ne doit contenir aucune hypothèse imposant une échelle particulière.

Les propriétés dépendantes de l'échelle appartiennent aux véhicules, catégories, circuits ou règles concernées.

## Nombre de participants

Le système ne doit pas être conçu autour de 6 ou 8 pilotes.

Les tests devront couvrir des plateaux importants, notamment 40+ participants, même si toutes les voitures ne roulent pas simultanément.

## Splits

Les splits sont une fonction importante.

Ils doivent pouvoir provenir :

- de capteurs physiques ;
- de points de passage numériques ;
- d'informations fournies par un système de timing externe.

Le moteur doit savoir distinguer passage de split, passage de ligne et événements PIT.

## Incidents et corrections

Une course réelle comporte des anomalies.

Le système devra permettre :

- correction contrôlée ;
- qualification d'un passage ;
- exclusion statistique sans destruction de la donnée brute ;
- justification d'une intervention manuelle ;
- audit après course.

## Points de chronométrage

Le format sportif doit pouvoir utiliser plusieurs points de chronométrage sans considérer les splits comme un simple ajout cosmétique.

Exemples :

```text
Départ/Arrivée
Départ/Arrivée + S1 + S2
Départ + S1 + S2 + Arrivée + PIT IN + PIT OUT
```

Les temps de secteur et splits sont calculés à partir de ces points et doivent pouvoir être utilisés pour le classement, l'analyse ou des services futurs.

## Multi-pistes

Une compétition peut utiliser plusieurs pistes ou installations en parallèle, avec un classement général couvrant un plateau plus large que les voitures simultanément présentes sur une seule piste.

Le modèle doit donc distinguer :

- compétition ;
- piste ;
- affectation d'un participant à une piste/session ;
- classement global ;
- vue filtrée par piste.

Le détail des règles d'agrégation multi-pistes reste à définir selon les formats de course.
