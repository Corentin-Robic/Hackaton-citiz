# Données d'affluence : Occitanie

Ce dossier rassemble les données d'usage et d'attractivité utilisées pour le Hackathon Mobility (Epitech x Citiz x Microsoft).

Elles servent à savoir **où les véhicules en libre-service sont réellement utilisés**, **où se trouvent les activités, les services et les emplois**, **quelles gares sont les plus fréquentées** et **où les actifs vont travailler**, pour estimer le potentiel d'une nouvelle station. Elles alimentent le modèle prédictif de l'objectif 3.

## Territoire étudié

**Région Occitanie (13 départements : 09, 11, 12, 30, 31, 32, 34, 46, 48, 65, 66, 81, 82).**

Le réseau Citiz Occitanie couvre Toulouse et sa périphérie mais aussi des villes moyennes et petites (Albi, Tarbes, Millau, Cahors, Saint-Affrique…). Garder toute la région permet d'entraîner le modèle sur des contextes variés. Les fichiers filtrés contiennent une colonne `departement` : on peut se restreindre à la Haute-Garonne (`31`) en une ligne de filtre.

## Fichiers

### Fichiers prêts à l'emploi (dossier `occitanie/`)

Ce sont les fichiers à utiliser pour l'analyse et le modèle : tous filtrés sur l'Occitanie, avec des coordonnées en latitude / longitude.

| Fichier | Contenu | Niveau géographique |
|---|---|---|
| `sirene_occitanie.csv` | Établissements d'entreprises géolocalisés | Point (adresse) |
| `bpe_occitanie.csv` | Équipements et services géolocalisés (colonnes INSEE complètes) | Point (adresse) |
| `bpe_occitanie_categories.csv` | Équipements avec leur nom en clair et leur domaine | Point (adresse) |
| `bpe_communes_occitanie.csv` | Nombre d'équipements par commune et par domaine | Commune |
| `gares_occitanie.csv` | Gares avec position et voyageurs par année, 2015-2024 | Gare |
| `mobpro_flux_occitanie.csv` | Trajets domicile-travail partant d'Occitanie ou y arrivant | Commune → commune |
| `mobpro_communes_occitanie.csv` | Actifs résidents, sortants, entrants et attractivité par commune | Commune |

### Relevés temps réel

| Fichier | Contenu | État |
|---|---|---|
| `gbfs_brut/<service>/` | Relevés toutes les 5 min des 14 services en libre-service | Collecte en cours |
| `analyses/` | Usage calculé de chaque station | À générer en fin de collecte |

### Fichiers sources (France entière, conservés tels quels)

| Fichier | Contenu |
|---|---|
| `geoloc-geolocalisationetablissement-sirene-….parquet` | SIRENE géolocalisé, 792 Mo |
| `BPE25.parquet` | BPE 2025, équipements géolocalisés |
| `TYPEQU_2025.csv` | Dictionnaire des types d'équipements BPE |
| `frequentation-gares.csv` | Voyageurs par gare et par année |
| `gares-de-voyageurs.geojson` | Position des gares voyageurs |
| `base-flux-mobilite-domicile-lieu-travail-….csv` | Trajets domicile-travail entre communes |

Les fichiers sources ne sont pas utilisés directement : ils servent à reconstruire les fichiers du dossier `occitanie/` avec les scripts.

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
- **Contrôle** : 306 547 équipements
- **Clé de croisement** : `TYPEQU` avec `TYPEQU_2025.csv`
- **Version simplifiée** : `bpe_occitanie_categories.csv` (voir plus bas)

### 3. `frequentation-gares.csv`

Nombre de voyageurs de chaque gare, chaque année de 2015 à 2024. Une gare très fréquentée est un bon emplacement pour une station Citiz : le voyageur arrive en train et termine son trajet en voiture partagée.

- **Source** : SNCF Gares & Connexions, Fréquentation en gares
- **Contenu** : nom de la gare, code UIC, code postal, segment de la gare, nombre de voyageurs par année
- **Traitement** : relié à la position des gares et filtré sur l'Occitanie dans `occitanie/gares_occitanie.csv` (script `nettoyer_affluence.py`)
- **Clé de croisement** : code UIC avec `gares-de-voyageurs.geojson`, pour obtenir la position de la gare

### 4. `gares-de-voyageurs.geojson`

Position géographique de chaque gare voyageurs. Complète le fichier précédent, qui n'a pas de coordonnées.

- **Source** : SNCF, Gares de voyageurs du réseau ferré national
- **Contenu** : identifiant et nom de la gare, trigramme, segment, position géographique, code commune, code(s) UIC
- **Format** : GeoJSON, coordonnées en longitude / latitude
- **Traitement** : relié à la fréquentation dans `occitanie/gares_occitanie.csv`
- **Clé de croisement** : code UIC avec `frequentation-gares.csv` ; code commune pour filtrer sur l'Occitanie

### 5. `base-flux-mobilite-domicile-lieu-travail-….csv`

Pour chaque commune de résidence, le nombre d'actifs qui vont travailler dans chaque commune. Permet de repérer les communes d'où partent beaucoup de navetteurs, et les pôles d'emploi qui en attirent.

- **Source** : INSEE, Recensement 2022, base flux de mobilité « déplacements domicile-lieu de travail »
- **Colonnes** :
  - `CODGEO` : code de la commune de résidence
  - `DCLT`, `L_DCLT` : code et nom de la commune de travail
  - `NBFLUX_C22_ACTOCC15P` : nombre d'actifs de 15 ans ou plus faisant ce trajet
- **Traitement** : filtré sur l'Occitanie et résumé par commune dans `occitanie/mobpro_flux_occitanie.csv` et `occitanie/mobpro_communes_occitanie.csv`
- **Attention** : l'INSEE indique que les flux de moins de 200 personnes sont à considérer comme des ordres de grandeur.

### 6. `TYPEQU_2025.csv`

Dictionnaire des types d'équipements de la BPE : permet de savoir qu'un code correspond à une école, une pharmacie, etc.

- **Source** : INSEE, BPE 2025, liste hiérarchisée des types d'équipements
- **Colonnes** : `TYPEQU` (code), `LIB` (libellé), `DESCRIPTION`, `SDOM` (sous-domaine), `ANNEE`
- **Traitement** : aucun, fichier gardé tel quel
- **Clé de croisement** : `TYPEQU` avec `bpe_occitanie.csv` ; utilisé pour produire `bpe_occitanie_categories.csv`

### 7. `occitanie/gares_occitanie.csv`

Une ligne par gare d'Occitanie, avec sa position et sa fréquentation. C'est la version prête à l'emploi des fichiers 3 et 4.

- **Colonnes** :
  - `code_uic`, `nom_gare`, `code_postal`, `segment` (catégorie SNCF : a = nationale, b = régionale, c = locale)
  - `voyageurs_2015` à `voyageurs_2024` : voyageurs par année
  - `voyageurs_dernier` : voyageurs de la dernière année (2024)
  - `evolution_2019_2024_pct` : évolution de la fréquentation depuis 2019, en %
  - `code_commune`, `departement`, `longitude`, `latitude`
- **Traitement** : codes UIC ramenés à 8 chiffres pour relier les deux fichiers SNCF (une gare peut avoir plusieurs codes UIC), filtre sur l'Occitanie par le code commune ou, à défaut, le code postal
- **Contrôle** : 279 gares en Occitanie, dont 34 sans position. En tête : Toulouse Matabiau (14,5 millions de voyageurs en 2024), Montpellier Saint-Roch (7,5 M), Nîmes (4,4 M), Montpellier Sud de France (2,5 M), Narbonne (2,4 M)

### 8. `occitanie/mobpro_flux_occitanie.csv`

Les trajets domicile-travail dont la commune de résidence **ou** la commune de travail est en Occitanie (y compris les trajets avec les régions voisines).

- **Colonnes** : `commune_residence`, `nom_residence`, `commune_travail`, `nom_travail`, `actifs`, `dep_residence`, `dep_travail`
- **Contrôle** : 87 087 trajets (sur 986 967 en France)

### 9. `occitanie/mobpro_communes_occitanie.csv`

Indicateurs d'emploi et de déplacements pour chaque commune d'Occitanie.

- **Colonnes** :
  - `code_commune`, `nom`, `departement`
  - `actifs_residents` : actifs ayant un emploi qui habitent la commune
  - `actifs_travaillant_sur_place` : ceux qui travaillent dans leur propre commune
  - `actifs_sortants` : ceux qui partent travailler dans une autre commune
  - `actifs_entrants` : actifs venant d'autres communes pour y travailler
  - `emplois_occupes` : emplois de la commune (sur place + entrants)
  - `part_sortants_pct` : part des résidents qui travaillent ailleurs
  - `indice_attractivite` : emplois / actifs résidents. Au-dessus de 1, la commune attire plus de travailleurs qu'elle n'en loge : c'est un **pôle d'emploi**. En dessous, c'est une commune plutôt **résidentielle**, d'où partent des navetteurs.
- **Contrôle** : 4 421 communes. Plus gros pôles d'emploi : Toulouse (352 900 emplois, indice 1,5), Montpellier (177 000, 1,5), Nîmes (82 800, 1,6), Perpignan (71 200, 1,9), Béziers (41 100, 1,7)
- **Pourquoi cette donnée** : une commune résidentielle avec beaucoup de navetteurs est une cible pour réduire l'usage de la voiture ; un pôle d'emploi attire des déplacements en journée.

### 10. `occitanie/bpe_occitanie_categories.csv`

Version simplifiée de la BPE : chaque équipement avec son nom en clair et son domaine.

- **Colonnes** : `TYPEQU`, `libelle` (ex. « Hypermarché », « École maternelle »), `domaine`, `sous_domaine`, `code_commune`, `iris`, `departement`, `longitude_gps`, `latitude_gps`
- **Domaines** (première lettre du code `TYPEQU`) : `services_particuliers` (A), `commerces` (B), `enseignement` (C), `sante_action_sociale` (D), `transports_deplacements` (E), `sports_loisirs_culture` (F), `tourisme` (G)
- **Contrôle** : 306 547 équipements : services aux particuliers 144 474, santé et action sociale 76 811, commerces 39 568, sports-loisirs-culture 28 221, enseignement 7 461, tourisme 5 134, transports 4 878

### 11. `occitanie/bpe_communes_occitanie.csv`

Nombre d'équipements par commune, pour chaque domaine.

- **Colonnes** : `code_commune`, `nb_<domaine>` pour chacun des 7 domaines, `nb_equipements_total`
- **Contrôle** : 4 446 communes

### 12. Flux temps réel : `gbfs_brut/<service>/`

État des stations et véhicules en libre-service enregistré **toutes les 5 minutes** pendant le hackathon (script `collecte_gbfs.py`). Aucune donnée d'usage n'étant publiée directement, c'est ce relevé régulier qui permet de la reconstituer : quand le nombre de véhicules d'une station baisse entre deux relevés, des véhicules sont partis.

On ne collecte que les services **en libre-service** : les bus, trams, métros et trains ont des horaires fixes (GTFS), téléchargés une seule fois dans la partie réseaux.

**Services enregistrés (14)** :

| Dossier | Service | Type | Zone |
|---|---|---|---|
| `citiz_occitanie` | Citiz Occitanie | Autopartage, en stations | Toulouse et 29 autres communes |
| `velotoulouse` | VélôToulouse | Vélos, en stations | Toulouse Métropole |
| `yego_toulouse` | YEGO | Scooters électriques, sans stations | Toulouse Métropole |
| `velomagg` | Vélomagg | Vélos, en stations | Montpellier Méditerranée Métropole |
| `nemovelo_nimes` | NemoVélo | Vélos électriques | Nîmes Métropole |
| `tlp_tarbes` | TLP Mobilités | Vélos | Tarbes-Lourdes-Pyrénées |
| `pony_perpignan` | Pony | Vélos et trottinettes | Perpignan |
| `cyclolibre_carcassonne` | CYCLOlibre | Vélos électriques | Carcassonne Agglo |
| `veloc_auch` | Vélo'c | Vélos | Grand Auch |
| `velodaqui_argeles` | Vélo d'Aqui | Vélos électriques | Argelès-sur-Mer |
| `velopartag_gevaudan` | VélopartaG | Vélos | CC du Gévaudan |
| `dott_france` | Dott | Vélos et trottinettes, sans stations | National, filtré Occitanie |
| `clem_france` | Clem | Autopartage | National, filtré Occitanie |
| `citiz_developpement` | Citiz Développement | Autopartage | National, filtré Occitanie |

Les flux nationaux sont filtrés sur un cadre géographique autour de l'Occitanie : seuls les stations et véhicules situés dans ce cadre sont enregistrés. Un service sans aucun véhicule en Occitanie ne produit pas de dossier.

**Getaround n'est pas collecté** : il publie un flux par commune de France (3 754), trop lourd à relever toutes les 5 minutes, pour un service de location entre particuliers peu comparable à Citiz (un propriétaire peut rendre sa voiture indisponible sans qu'elle soit louée).

**Fichiers produits dans chaque dossier** :

- **`station_information_<date>.csv`** (une fois par jour) : `collecte`, `station_id`, `nom`, `lat`, `lon`, `capacite`, `adresse`
- **`station_status_<date>.csv`** (toutes les 5 min) : `collecte` (date et heure du relevé), `last_reported`, `station_id`, `vehicules_disponibles`, `places_libres`, `vehicules_hors_service`, `en_location`, `en_retour`
- **`vehicle_status_<date>.csv`** (toutes les 5 min) : `collecte`, `vehicle_id`, `station_id`, `lat`, `lon`, `vehicle_type_id`, `reserve`, `hors_service`, `autonomie_m`

Les services sans stations (YEGO, Dott…) n'ont que le fichier `vehicle_status`.

- **Période** : du **20/10/2026 à 9h56** au _à compléter_, pour les 14 services
- **Test préalable** : une première collecte sur 4 services (Citiz, VélôToulouse, YEGO, Vélomagg) a été faite le 19/10 à partir de 17h04, puis arrêtée. Elle n'est pas utilisée dans l'analyse (fichiers rangés à part dans `gbfs_test_19oct/`).

### 13. Usage calculé : `analyses/`

Résultat du script `analyse_usage.py`, à lancer à partir des relevés. _Fichiers à générer en fin de collecte._

- **`usage_stations_<service>.csv`** : une ligne par station
  - `station_id`, `nom`, `lat`, `lon`, `capacite`
  - `departs`, `retours` : nombre total détecté sur la période
  - `heures_observees`, `nb_releves`
  - `departs_par_jour` : **indicateur principal d'usage**, cible du modèle prédictif
  - `departs_par_jour_par_place` : usage rapporté à la taille de la station
  - `dispo_moyenne` : véhicules disponibles en moyenne
  - `part_temps_vide` : part des relevés où la station n'avait aucun véhicule. Une station souvent vide signale une demande non satisfaite.
- **`profil_horaire_<service>.csv`** : `heure`, `departs_moyens`
- **`flotte_<service>.csv`** (services sans stations) : `collecte`, `vehicules_disponibles`
- **Clé de croisement** : position `lat` / `lon` de la station, à rattacher au carreau de 200 m

## Scripts

| Script | Rôle |
|---|---|
| `filtrer_occitanie.py` | Filtre SIRENE et la BPE sur l'Occitanie et ajoute latitude / longitude |
| `telecharger.py` | Télécharge un gros fichier en reprenant automatiquement après une coupure |
| `collecte_gbfs.py` | Enregistre toutes les 5 minutes l'état des 14 services en libre-service (11 d'Occitanie, 3 nationaux filtrés sur l'Occitanie) |
| `nettoyer_affluence.py` | Relie et filtre les gares, résume le domicile-travail par commune, ajoute les catégories de la BPE |
| `analyse_usage.py` | Calcule l'usage de chaque station à partir des relevés |

Dépendances : `pip install duckdb pyproj pandas pyarrow` (`collecte_gbfs.py` n'en a besoin d'aucune).

## Format et points d'attention

- **Séparateur** : les fichiers produits par nos scripts (`occitanie/`, `gbfs_brut/`, `analyses/`) utilisent la virgule `,` (différent des fichiers démographie, qui utilisent `;`). Fabric et pandas lisent les deux. Les fichiers SNCF et INSEE sont gardés dans leur format d'origine.
- **Encodage** : UTF-8.
- **Coordonnées** : utiliser `longitude_gps` / `latitude_gps` pour SIRENE et la BPE, `lon` / `lat` pour les relevés GBFS, en degrés. Les coordonnées d'origine de l'INSEE peuvent être en Lambert 93, un autre système.
- **Codes commune** : à garder en texte pour ne pas perdre le zéro des départements 09 (utiliser `code_commune_5` pour SIRENE et la BPE).
- **Codes UIC** : vérifier qu'ils ont le même format dans les deux fichiers SNCF avant de les relier (nombre de chiffres).
- **Heures** : la colonne `collecte` des relevés GBFS est à l'heure de Paris, avec son décalage (`+02:00`).
- **Fabric** : la collecte écrit sur le PC local. Le dossier `gbfs_brut` doit être renvoyé dans le Lakehouse régulièrement (et juste avant l'analyse) pour y avoir les relevés à jour.
- **Clé de croisement** :
  - avec la population : rattacher chaque point (établissement, équipement, gare, station) à son carreau de 200 m
  - avec la motorisation et les contours : code commune
  - avec les stations : compter les établissements et équipements à moins de 400 m (5 min à pied) de chaque station

## Limites

- **SIRENE compte des établissements, pas des emplois** : le fichier ne donne pas le nombre de salariés. Une grande usine et un auto-entrepreneur comptent pareil. Le fichier contient probablement aussi des établissements fermés.
- **Gares sans position** : 34 gares d'Occitanie de la fréquentation n'ont pas été retrouvées dans le fichier des positions (code UIC absent ou gare fermée). Elles sont gardées, sans coordonnées.
- **Services aux particuliers** : ce domaine de la BPE regroupe aussi les artisans (plombiers, électriciens…), qui ne sont pas des destinations de déplacement. Pour le modèle, préférer les commerces, la santé, l'enseignement et les sports-loisirs, ou filtrer par sous-domaine.
- **Usage déduit des relevés** : un départ et un retour dans la même fenêtre de 5 minutes ne sont pas vus, les chiffres sont donc des minimums. Pour les vélos, les camions de régulation qui déplacent des vélos comptent aussi comme des départs et retours.
- **Courte période d'observation** : 2 à 3 jours de relevés, en semaine, qui ne reflètent ni les week-ends ni les variations saisonnières.
- **Filtre géographique approximatif** : les flux nationaux sont filtrés sur un rectangle autour de l'Occitanie, qui déborde un peu sur les régions voisines. Un filtre exact par commune peut être appliqué à l'analyse.
- **Dates** : établissements de septembre 2026, équipements 2025, fréquentation des gares 2024, trajets domicile-travail 2022, relevés d'octobre 2026.

## Sources

Toutes les sources (liens, millésimes, licences) sont listées dans [`SOURCES.md`](SOURCES.md), section **Affluence**.
