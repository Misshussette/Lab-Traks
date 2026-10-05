# Lab-Traks

Lab-Traks est le projet de logiciel de chronométrage et de gestion de course universel.

## Vision

Créer un outil :

1. **hyper fiable** en situation de course ;
2. **simple et agréable** à utiliser ;
3. **universel**, quel que soit le type de course, l'échelle ou le système de détection ;
4. **léger**, afin de rester utilisable sur des PC anciens.

PC Lap Counter est étudié comme une **source d'expérience, de cas réels et d'interopérabilité**. L'objectif n'est pas d'en faire une copie moderne.

## Principe de référence

Ce dépôt GitHub est la **source de vérité du projet**. Les conversations, idées et expérimentations doivent être consolidées ici lorsqu'elles deviennent des décisions, des contraintes ou des connaissances utiles.

## Principes déjà actés

- Windows en priorité.
- Fonctionnement hors ligne indispensable.
- Internet ne doit pas être nécessaire pour installer, licencier ou chronométrer une course.
- Le moteur de course doit être séparé de l'interface et des drivers matériels.
- Le logiciel doit rester peu gourmand en CPU et mémoire.
- La précision cible minimale est la milliseconde.
- La persistance doit permettre une reprise fiable après crash ou coupure.
- Une donnée métier ne doit exister qu'une seule fois et être référencée partout.
- Les anciens matériels et protocoles doivent être intégrés par des adaptateurs dédiés.
- Le vocabulaire des matériels externes ne doit jamais imposer le modèle interne de Lab-Traks.
- Les affichages doivent pouvoir être configurés, sauvegardés et réutilisés.
- Les splits, points de chronométrage, stands, changements de pilote, transpondeurs et systèmes multi-voies font partie du périmètre.
- Les configurations d'affichage sont des données réutilisables et persistantes.
- Les services Web futurs restent indépendants du cœur local.

## Documentation

- [Bible projet](docs/PROJECT_BIBLE.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Modèle de données](docs/DATA_MODEL.md)
- [Matériels et détection](docs/HARDWARE_AND_DETECTION.md)
- [Formats de course](docs/RACE_FORMATS.md)
- [Décisions](docs/DECISIONS.md)
- [Questions ouvertes](docs/OPEN_QUESTIONS.md)
- [Roadmap](docs/ROADMAP.md)
- [Reverse engineering PCLC](docs/PCLC_REVERSE_ENGINEERING.md)
- [Étude Ultimate Racer 3.0](docs/ULTIMATE_RACER_RESEARCH.md)
- [Designer de circuit et bibliothèque de rails](docs/TRACK_DESIGNER.md)

## État technique

La stack applicative n'est **pas encore choisie**. C#/.NET, Avalonia et d'autres solutions peuvent être étudiés, mais aucun choix ne doit être considéré comme acté tant qu'il n'est pas enregistré dans [DECISIONS.md](docs/DECISIONS.md).
