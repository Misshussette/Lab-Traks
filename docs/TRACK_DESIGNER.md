# Designer de circuit Lab-Traks

## Direction retenue

Le designer de circuit Lab-Traks doit être **beaucoup plus simple à utiliser qu'Ultimate Racer 3.0**, tout en proposant une bibliothèque de rails très riche.

L'objectif n'est pas de reproduire un logiciel de CAO.

L'utilisateur doit pouvoir construire son circuit comme il assemblerait les vrais rails.

## Expérience utilisateur cible

1. choisir une marque / gamme de rails ;
2. voir les pièces disponibles dans une palette claire ;
3. cliquer ou glisser une pièce pour l'ajouter ;
4. la pièce s'accroche automatiquement à l'extrémité du circuit ;
5. continuer jusqu'à fermeture du tracé ;
6. obtenir automatiquement dimensions, longueurs de voies et informations utiles au chronométrage.

Fonctions de base :

- ajout ;
- suppression ;
- remplacement ;
- inversion/orientation ;
- déplacement du circuit complet ;
- zoom ;
- annuler/rétablir ;
- fermeture et indication de l'écart restant ;
- affichage facultatif des références des rails.

Les fonctions avancées ne doivent pas encombrer l'usage courant.

## Bibliothèque de rails

La bibliothèque Lab-Traks doit être indépendante du designer et constituer une donnée de référence réutilisable.

Chaque pièce doit pouvoir décrire au minimum :

- fabricant ;
- gamme/système ;
- référence fabricant ;
- nom ;
- échelle ;
- type de pièce ;
- longueur pour une droite ;
- rayon et angle pour une courbe ;
- largeur totale ;
- nombre de voies ;
- position/écartement des slots ;
- connecteurs début/fin ;
- sens imposé éventuel ;
- changement de voie/croisement éventuel ;
- caractéristiques digitales éventuelles ;
- source et statut de vérification des dimensions.

La représentation graphique doit être générée par Lab-Traks à partir de cette géométrie autant que possible.

## Sources

L'archive Ultimate Racer 3.0 étudiée contient 29 bibliothèques commerciales/historiques principales représentant environ 754 définitions de pièces, hors bibliothèques SXR.

Elles couvrent notamment :

- Airfix ;
- Artin ;
- Aurora ;
- Carrera Digital 132 ;
- Carrera Digital 143 ;
- Carrera Go ;
- Carrera Exclusiv ;
- Carrera Universal ;
- Fleischmann ;
- Jouef ;
- LifeLike ;
- Marchon ;
- MaxTrax ;
- Ninco ;
- Polistil ;
- Revell/Riggen ;
- Scalextric Sport ;
- Scalextric Digital ;
- SCX ;
- SCX Digital ;
- SCX Compact ;
- Stabo ;
- Strombecker ;
- Tomy AFX ;
- Tyco.

Les fichiers UR30 montrent qu'une bibliothèque peut être décrite avec des données géométriques simples : largeur/longueur pour les droites, rayon/largeur/angle pour les courbes, positions des voies, formes spéciales et références fabricant.

## Stratégie de constitution de la bibliothèque Lab-Traks

Les données UR30 constituent un excellent **index de départ** pour savoir quelles pièces rechercher et quelles caractéristiques sont nécessaires.

Pour la bibliothèque distribuée avec Lab-Traks :

1. identifier la pièce et sa référence ;
2. vérifier ses caractéristiques avec documentation fabricant, catalogue ou autre source fiable ;
3. enregistrer les caractéristiques factuelles dans notre propre format ;
4. conserver la provenance et le niveau de confiance ;
5. générer nous-mêmes la représentation graphique ;
6. ne pas recopier automatiquement logos, photos, textures ou autres assets tiers dont les droits ne sont pas établis.

Cela permet de construire progressivement une bibliothèque moderne, maintenable et indépendante.

## Bibliothèque ouverte et extensible

Le format de bibliothèque doit permettre :

- ajout de nouvelles marques sans modifier le logiciel ;
- mise à jour d'une référence ;
- ajout d'une pièce personnalisée ;
- import/export d'une bibliothèque ;
- contribution communautaire avec validation ;
- alias pour anciennes/nouvelles références d'une même géométrie ;
- pièces abandonnées ou historiques conservées pour les anciens circuits.

Une donnée de rail ne doit exister qu'une fois dans la bibliothèque de référence.

## Palette simplifiée

Une grosse bibliothèque ne doit pas rendre l'interface compliquée.

La palette doit pouvoir proposer :

- pièces fréquentes en premier ;
- recherche par référence ou nom ;
- filtres droite/courbe/digital/accessoire ;
- favoris ;
- éventuellement le stock réel du club/utilisateur.

Le stock est une relation vers la pièce de référence ; il ne duplique pas sa géométrie.

## Circuit et chronométrage

Le tracé dessiné ne doit pas être une simple image.

Il doit devenir une représentation du même objet Circuit utilisé par le chronométrage.

Cela permettra notamment :

- calcul automatique de longueur par voie ;
- vitesse moyenne/réelle ;
- position de la ligne de chronométrage ;
- position des splits ;
- position des entrées/sorties de stands ;
- association des capteurs physiques ;
- affichage live du circuit ;
- statistiques liées à la géométrie réelle.

## Fonctions à envisager plus tard

Sans compliquer le premier designer :

- suggestion automatique pour fermer un circuit ;
- contrôle du stock disponible ;
- proposition de rails manquants ;
- contraintes de dimensions de pièce/table ;
- élévations ;
- ponts ;
- tracé bois/routed ;
- import de formats tiers ;
- vue 3D.

Ces fonctions sont secondaires par rapport à un designer 2D simple et fiable.


## Pistes bois / routed tracks

Une piste bois ne doit pas être modélisée comme une succession de rails standards ni comme une simple ligne centrale avec largeur constante.

Le designer doit permettre de travailler avec plusieurs géométries liées mais distinctes :

- trajectoire ou ligne de construction ;
- slots/voies ;
- contour intérieur de la piste ;
- contour extérieur de la piste.

### Largeur variable

Les contours intérieur et extérieur doivent être **éditables indépendamment**.

La largeur de piste peut donc varier librement selon la zone :

- élargissement dans un virage ;
- dégagement important à l'extérieur ;
- zone de ramassage ;
- resserrement ;
- ligne droite plus étroite ;
- zone des stands ;
- forme particulière imposée par le plateau ou la salle.

Une génération automatique à partir d'une ligne de construction et d'une largeur initiale peut servir de point de départ, mais elle ne doit jamais imposer une largeur constante.

### Édition

L'utilisateur doit pouvoir déplacer les points/nœuds et poignées de courbe des contours intérieur et extérieur.

Les slots peuvent être générés automatiquement puis ajustés lorsque la piste réelle l'exige.

Modifier un contour ne doit pas modifier arbitrairement les slots. Les différentes géométries restent liées au même Circuit mais représentent des réalités physiques distinctes.

### Données calculées

À partir de ces géométries, Lab-Traks peut déterminer notamment :

- encombrement réel du circuit ;
- largeur locale de piste ;
- longueur géométrique de chaque slot ;
- position des capteurs ;
- secteurs/splits ;
- zones de stands ;
- représentation fidèle du circuit pour les affichages.

Une longueur de voie mesurée/calibrée sur la piste réelle peut remplacer la longueur géométrique comme valeur de référence pour les calculs sportifs, sans supprimer la valeur géométrique calculée.
