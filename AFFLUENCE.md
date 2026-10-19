# Données d'affluence : Occitanie

Ce dossier rassemble les données d'usage et d'attractivité utilisées pour le Hackathon Mobility (Epitech x Citiz x Microsoft).

Elles servent à savoir **où les stations sont réellement utilisées**, **où les gens se rendent** (commerces, écoles, santé, emplois) et **quelles gares sont les plus fréquentées**, pour estimer le potentiel d'une nouvelle station. Ce sont elles qui alimentent le modèle prédictif de l'objectif 3.

## Territoire étudié

**Région Occitanie (13 départements : 09, 11, 12, 30, 31, 32, 34, 46, 48, 65, 66, 81, 82).**

Le réseau Citiz Occitanie couvre Toulouse et sa périphérie mais aussi des villes moyennes et petites (Albi, Tarbes, Millau, Cahors, Saint-Affrique…). Garder toute la région permet d'entraîner le modèle sur des contextes variés. Chaque fichier contient une colonne `departement` : on peut se restreindre à la Haute-Garonne (`31`) en une ligne de filtre.

## Fichiers

| Fichier | Contenu | Niveau géographique |
|---|---|---|
| `occitanie/sirene_occitanie.csv` | Établissements d'entreprises géolocalisés | Point (adresse) |
| `occitanie/bpe_occitanie.csv` | Équipements et services géolocalisés | Point (adresse) |
| `TYPEQU_2025.csv` | Dictionnaire des types d'équipements BPE | — |
| `gbfs_brut/<service>/station_information_<date>.csv` | Liste des stations : nom, position, capacité | Station |
| `gbfs_brut/<service>/station_status_<date>.csv` | Véhicules disponibles par station, toutes les 5 min | Station |
| `gbfs_brut/<service>/vehicle_status_<date>.csv` | Position des véhicules disponibles, toutes les 5 min | Véhicule |
| `analyses/usage_stations_<service>.csv` | Usage calculé de chaque station | Station |
| `analyses/profil_horaire_<service>.csv` | Départs moyens selon l'heure de la journée | Service |
| `analyses/flotte_<service>.csv` | Nombre de véhicules disponibles dans le temps (services sans stations) | Service |

Services enregistrés (`<service>`) : `citiz_occitanie`, `velotoulouse`, `yego_toulouse`, `velomagg`.

## Détail des fichiers

### 1. `occitanie/sirene_occitanie.csv`

Position de chaque établissement d'entreprise ou d'organisation en Occitanie. Sert à repérer les zones d'emploi et d'activité.

- **Source** : INSEE, Géolocalisation des établissements du répertoire SIRENE pour les études statistiques (fichier de septembre 2026)
- **Colonnes principales** :
  - `siret` : identifiant de l'établissement
  - `plg_code_commune` : code commune d'origine
  - `x_longitude`, `y_latitude` : position fournie par l'INSEE
  - colonnes de zonage (IRIS, quartiers prioritaires…) et de qualité de la géolocalisation, telles que fournies par l'INSEE
  - colonnes ajoutées par le script : `code_commune_5` (code commune sur 5 caractères), `departement`, `longitude_gps`, `latitude_gps`
- **Traitement** : filtre sur les codes commune commençant par un des 13 départements d'Occitanie (script `filtrer_occitanie.py`)
- **Contrôle** : 3 746 143 établissements, 100 % des points dans le cadre géographique de l'Occitanie, aucun sans coordonnées

### 2. `occitanie/bpe_occitanie.csv`

Position de chaque équipement et service en Occitanie : commerces, écoles, santé, sport, services publics, tourisme. Ce sont les **destinations** des déplacements.

- **Source** : INSEE, Base Permanente des Équipements 2025, fichier détail des équipements géolocalisés (Parquet)
- **Colonnes principales** :
  - `TYPEQU` : code du type d'équipement (signification dans `TYPEQU_2025.csv`)
  - code commune et coordonnées, telles que fournies par l'INSEE
  - colonnes ajoutées par le script : `code_commune_5`, `departement`, `longitude_gps`, `latitude_gps`
- **Traitement** : même filtre que SIRENE ; si le fichier ne contient que des coordonnées Lambert 93, conversion en latitude / longitude
- **Contrôle** : _à compléter après traitement (nombre de lignes, % dans l'Occitanie)_

### 3. `TYPEQU_2025.csv`

Dictionnaire des 209 types d'équipements de la BPE : permet de savoir qu'un code correspond à une école, une pharmacie, etc.

- **Source** : INSEE, BPE 2025, liste hiérarchisée des types d'équipements
- **Colonnes** : `TYPEQU` (code), `LIB` (libellé), `DESCRIPTION`, `SDOM` (sous-domaine), `ANNEE`
- **Traitement** : aucun, fichier gardé tel quel
- **Clé de croisement** : `TYPEQU` avec `bpe_occitanie.csv`

### 4. Flux temps réel : `gbfs_brut/<service>/`

État des stations et véhicules en libre-service enregistré toutes les 5 minutes pendant le hackathon (script `collecte_gbfs.py`). Aucune donnée d'usage n'étant publiée directement, c'est ce relevé régulier qui permet de la reconstituer.

- **Source** : flux GBFS publiés sur transport.data.gouv.fr (liens dans [`SOURCES.md`](SOURCES.md))
- **Services** :
  - `citiz_occitanie` : autopartage Citiz (GBFS 3.0)
  - `velotoulouse` : vélos en libre-service de Toulouse
  - `yego_toulouse` : scooters électriques en libre-service, sans stations (GBFS 2.2)
  - `velomagg` : vélos en libre-service de Montpellier, ville sans Citiz
- **`station_information_<date>.csv`** (une fois par jour) : `collecte`, `station_id`, `nom`, `lat`, `lon`, `capacite`, `adresse`
- **`station_status_<date>.csv`** (toutes les 5 min) : `collecte` (date et heure du relevé), `last_reported`, `station_id`, `vehicules_disponibles`, `places_libres`, `vehicules_hors_service`, `en_location`, `en_retour`
- **`vehicle_status_<date>.csv`** (toutes les 5 min) : `collecte`, `vehicle_id`, `station_id`, `lat`, `lon`, `vehicle_type_id`, `reserve`, `hors_service`, `autonomie_m`
- **Période** : _à compléter (début et fin de l'enregistrement)_

### 5. Usage calculé : `analyses/`

Résultat du script `analyse_usage.py` à partir des relevés. Quand le nombre de véhicules d'une station baisse entre deux relevés, ce sont des départs ; quand il monte, des retours.

- **`usage_stations_<service>.csv`** : une ligne par station
  - `station_id`, `nom`, `lat`, `lon`, `capacite`
  - `departs`, `retours` : nombre total détecté sur la période
  - `heures_observees`, `nb_releves`
  - `departs_par_jour` : **indicateur principal d'usage**, cible du modèle prédictif
  - `departs_par_jour_par_place` : usage rapporté à la taille de la station
  - `dispo_moyenne` : véhicules disponibles en moyenne
  - `part_temps_vide` : part des relevés où la station n'avait aucun véhicule. Une station souvent vide signale une demande non satisfaite.
- **`profil_horaire_<service>.csv`** : `heure`, `departs_moyens`
- **`flotte_<service>.csv`** : `collecte`, `vehicules_disponibles` (pour YEGO)
- **Clé de croisement** : position `lat` / `lon` de la station, à rattacher au carreau de 200 m

## Scripts

| Script | Rôle |
|---|---|
| `filtrer_occitanie.py` | Filtre SIRENE et BPE sur l'Occitanie et ajoute latitude / longitude |
| `telecharger.py` | Télécharge un gros fichier en reprenant automatiquement après une coupure |
| `collecte_gbfs.py` | Enregistre les flux temps réel toutes les 5 minutes |
| `analyse_usage.py` | Calcule l'usage de chaque station à partir des relevés |

Dépendances : `pip install duckdb pyproj pandas pyarrow`

## Format et points d'attention

- **Séparateur des CSV** : virgule `,` (différent des fichiers démographie, qui utilisent `;`). Fabric et pandas lisent les deux.
- **Encodage** : UTF-8.
- **Coordonnées** : toujours utiliser `longitude_gps` / `latitude_gps` (BPE, SIRENE) ou `lon` / `lat` (GBFS), en degrés. Les coordonnées d'origine de l'INSEE peuvent être en Lambert 93, un autre système.
- **Codes commune** : utiliser `code_commune_5`, en texte, pour ne pas perdre le zéro des départements 09.
- **Heures** : la colonne `collecte` est à l'heure de Paris, avec son décalage (`+02:00`).
- **Clé de croisement** :
  - avec la population : rattacher chaque point (équipement, établissement, station) à son carreau de 200 m
  - avec la motorisation : code commune ou IRIS
  - entre usage et équipements : compter les équipements et établissements à moins de 400 m (5 min à pied) de chaque station

## Limites

- **SIRENE compte des établissements, pas des emplois** : le fichier ne donne pas le nombre de salariés. Une grande usine et un auto-entrepreneur comptent pareil. Le fichier contient probablement aussi des établissements fermés.
- **Usage déduit des relevés** : un départ et un retour dans la même fenêtre de 5 minutes ne sont pas vus. Les chiffres sont des minimums. Pour les vélos, les camions de régulation comptent aussi comme des départs et retours.
- **Courte période d'observation** : 2 à 3 jours de relevés, qui ne reflètent ni les week-ends ni les variations saisonnières.
- **Dates** : établissements de septembre 2026, équipements 2025, relevés d'octobre 2026.

## Sources

Toutes les sources (liens, millésimes, licences) sont listées dans [`SOURCES.md`](SOURCES.md), section **Affluence**.
