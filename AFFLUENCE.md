# Données d'affluence : Occitanie

Ce dossier rassemble les données d'usage et d'attractivité utilisées pour le Hackathon Mobility (Epitech x Citiz x Microsoft).

Elles servent à savoir **où se trouvent les activités, les services et les emplois**, **quelles gares sont les plus fréquentées** et **où les actifs vont travailler**, pour estimer le potentiel d'une nouvelle station. Elles alimentent le modèle prédictif de l'objectif 3.

## Territoire étudié

**Région Occitanie (13 départements : 09, 11, 12, 30, 31, 32, 34, 46, 48, 65, 66, 81, 82).**

Le réseau Citiz Occitanie couvre Toulouse et sa périphérie mais aussi des villes moyennes et petites (Albi, Tarbes, Millau, Cahors, Saint-Affrique…). Garder toute la région permet d'entraîner le modèle sur des contextes variés. Les fichiers filtrés contiennent une colonne `departement` : on peut se restreindre à la Haute-Garonne (`31`) en une ligne de filtre.

## Fichiers

| Fichier | Contenu | Niveau géographique | État |
|---|---|---|---|
| `occitanie/sirene_occitanie.csv` | Établissements d'entreprises géolocalisés | Point (adresse) | Filtré Occitanie |
| `occitanie/bpe_occitanie.csv` | Équipements et services géolocalisés | Point (adresse) | Filtré Occitanie |
| `frequentation-gares.csv` | Voyageurs par gare et par année, 2015-2024 | Gare | France entière |
| `gares-de-voyageurs.geojson` | Position des gares voyageurs | Gare | France entière |
| `base-flux-mobilite-domicile-lieu-travail-….csv` | Trajets domicile-travail entre communes | Commune → commune | France entière |
| `TYPEQU_2025.csv` | Dictionnaire des types d'équipements BPE | — | Tel quel |

Les fichiers d'origine SIRENE (`geoloc-geolocalisationetablissement-sirene-….parquet`, 792 Mo) et BPE (`BPE25.parquet`), France entière, sont conservés localement mais ne sont pas utilisés directement : on travaille sur les versions filtrées.

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
- **Traitement** : filtre sur les 13 départements d'Occitanie et ajout de la latitude / longitude (script `filtrer_occitanie.py`)
- **Contrôle** : _à compléter (nombre d'équipements, % dans le cadre de l'Occitanie, lignes sans coordonnées)_
- **Clé de croisement** : `TYPEQU` avec `TYPEQU_2025.csv`

### 3. `frequentation-gares.csv`

Nombre de voyageurs de chaque gare, chaque année de 2015 à 2024. Une gare très fréquentée est un bon emplacement pour une station Citiz : le voyageur arrive en train et termine son trajet en voiture partagée.

- **Source** : SNCF Gares & Connexions, Fréquentation en gares
- **Contenu** : nom de la gare, code UIC, code postal, segment de la gare, nombre de voyageurs par année
- **Traitement** : aucun pour l'instant (France entière, environ 3 000 gares)
- **Clé de croisement** : code UIC avec `gares-de-voyageurs.geojson`, pour obtenir la position de la gare

### 4. `gares-de-voyageurs.geojson`

Position géographique de chaque gare voyageurs. Complète le fichier précédent, qui n'a pas de coordonnées.

- **Source** : SNCF, Gares de voyageurs du réseau ferré national
- **Contenu** : identifiant et nom de la gare, trigramme, segment, position géographique, code commune, code(s) UIC
- **Format** : GeoJSON, coordonnées en longitude / latitude
- **Traitement** : aucun pour l'instant (France entière)
- **Clé de croisement** : code UIC avec `frequentation-gares.csv` ; code commune pour filtrer sur l'Occitanie

### 5. `base-flux-mobilite-domicile-lieu-travail-….csv`

Pour chaque commune de résidence, le nombre d'actifs qui vont travailler dans chaque commune. Permet de repérer les communes d'où partent beaucoup de navetteurs, et les pôles d'emploi qui en attirent.

- **Source** : INSEE, Recensement 2022, base flux de mobilité « déplacements domicile-lieu de travail »
- **Colonnes** :
  - `CODGEO` : code de la commune de résidence
  - `DCLT`, `L_DCLT` : code et nom de la commune de travail
  - `NBFLUX_C22_ACTOCC15P` : nombre d'actifs de 15 ans ou plus faisant ce trajet
- **Traitement** : aucun pour l'instant (France entière). À filtrer sur les communes d'Occitanie, en résidence et / ou en lieu de travail.
- **Attention** : l'INSEE indique que les flux de moins de 200 personnes sont à considérer comme des ordres de grandeur.

### 6. `TYPEQU_2025.csv`

Dictionnaire des types d'équipements de la BPE : permet de savoir qu'un code correspond à une école, une pharmacie, etc.

- **Source** : INSEE, BPE 2025, liste hiérarchisée des types d'équipements
- **Colonnes** : `TYPEQU` (code), `LIB` (libellé), `DESCRIPTION`, `SDOM` (sous-domaine), `ANNEE`
- **Traitement** : aucun, fichier gardé tel quel
- **Clé de croisement** : `TYPEQU` avec `bpe_occitanie.csv`

## Scripts

| Script | Rôle |
|---|---|
| `filtrer_occitanie.py` | Filtre SIRENE et la BPE sur l'Occitanie et ajoute latitude / longitude |
| `telecharger.py` | Télécharge un gros fichier en reprenant automatiquement après une coupure |
| `collecte_gbfs.py` | Enregistre toutes les 5 minutes l'état des stations Citiz, VélôToulouse, YEGO et Vélomagg |

Dépendances : `pip install duckdb pyproj pandas pyarrow`

## Format et points d'attention

- **Séparateur** : `sirene_occitanie.csv` et `bpe_occitanie.csv` utilisent la virgule `,` (différent des fichiers démographie, qui utilisent `;`). Fabric et pandas lisent les deux. Les fichiers SNCF et INSEE sont gardés dans leur format d'origine.
- **Encodage** : UTF-8.
- **Coordonnées** : utiliser `longitude_gps` / `latitude_gps` pour SIRENE et la BPE, en degrés. Les coordonnées d'origine de l'INSEE peuvent être en Lambert 93, un autre système.
- **Codes commune** : à garder en texte pour ne pas perdre le zéro des départements 09 (utiliser `code_commune_5` pour SIRENE et la BPE).
- **Codes UIC** : vérifier qu'ils ont le même format dans les deux fichiers SNCF avant de les relier (nombre de chiffres).
- **Clé de croisement** :
  - avec la population : rattacher chaque point (établissement, équipement, gare) à son carreau de 200 m
  - avec la motorisation et les contours : code commune
  - avec les stations : compter les établissements et équipements à moins de 400 m (5 min à pied) de chaque station

## Limites

- **SIRENE compte des établissements, pas des emplois** : le fichier ne donne pas le nombre de salariés. Une grande usine et un auto-entrepreneur comptent pareil. Le fichier contient probablement aussi des établissements fermés.
- **Dates** : établissements de septembre 2026, équipements 2025, fréquentation des gares 2024, trajets domicile-travail 2022.

## Prochaine étape

Le script `collecte_gbfs.py` enregistrera l'usage réel des stations en libre-service pendant le hackathon. Les fichiers produits seront décrits ici une fois la collecte lancée.

## Sources

Toutes les sources (liens, millésimes, licences) sont listées dans [`SOURCES.md`](SOURCES.md), section **Affluence**.
