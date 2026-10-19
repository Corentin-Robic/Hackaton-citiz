# Hackaton-citiz
# Données démographiques : Haute-Garonne

Ce dossier rassemble les données de population utilisées pour le Hackathon Mobility (Epitech x Citiz x Microsoft).

Elles servent à savoir **où vivent les habitants**, **comment la population va évoluer** et **quels ménages n'ont pas de voiture**, pour ensuite les croiser avec l'offre de transport.

## Territoire étudié

**Département de la Haute-Garonne (31).**

Ce choix permet de comparer une métropole bien équipée (Toulouse) avec des zones périurbaines et rurales moins desservies. Les projections de population de l'INSEE sont aussi disponibles à l'échelle du département, ce qui garde toutes les données cohérentes.

## Fichiers

| Fichier | Contenu | Niveau géographique |
|---|---|---|
| `filosofi_200m_haute_garonne.csv` | Habitants, âges et revenus | Carreau de 200 m |
| `projections_population_haute_garonne.csv` | Population projetée de 2018 à 2070 | Département |
| `projections_age_haute_garonne.csv` | Répartition par âge en 2018 et 2070 | Département |
| `motorisation_haute_garonne.csv` | Ménages et ménages avec voiture | IRIS (quartier) |
| `contours_communes_haute_garonne.geojson` | Contours des communes | Commune |

## Détail des fichiers

### 1. `filosofi_200m_haute_garonne.csv`

Nombre d'habitants, âges et revenus dans chaque carré de 200 m de la Haute-Garonne.

- **Source** : INSEE, Filosofi 2021, données au carreau de 200 m
- **Colonnes principales** :
  - `idcar_200m` : identifiant du carreau (contient sa position)
  - `lcog_geo` : code(s) commune. Un carreau à cheval sur plusieurs communes a plusieurs codes séparés par une virgule
  - `ind` : nombre d'habitants
  - `ind_0_3` à `ind_80p` : habitants par tranche d'âge
  - `men`, `men_pauv`, `men_1ind`, `men_5ind`, `men_fmp` : ménages (total, pauvres, une personne, 5 personnes ou plus, familles monoparentales)
  - `ind_snv` : somme des niveaux de vie des habitants
  - `i_est_200` : vaut 1 si les valeurs du carreau sont estimées par l'INSEE (secret statistique)
  - `men_prop`, `men_coll`, `men_mais`, `log_av45` à `log_soc` : propriétaires, type de logement, période de construction, logements sociaux
- **Traitement** : filtre sur les carreaux dont `lcog_geo` commence par `31`
- **Contrôle** : 42 699 carreaux, environ 1 344 000 habitants au total, 5 231 carreaux à cheval sur plusieurs communes, 32 334 carreaux avec valeurs estimées

### 2. `projections_population_haute_garonne.csv`

Évolution prévue de la population du département, année par année, de 2018 à 2070.

- **Source** : INSEE, Omphale 2022, scénario central, onglet `Population_DEP`
- **Unité** : **milliers d'habitants** (1381 = 1 381 000 habitants)
- **Traitement** : conservation de la ligne Haute-Garonne

### 3. `projections_age_haute_garonne.csv`

Répartition de la population par tranche d'âge en 2018 et en 2070, avec l'âge moyen.

- **Source** : INSEE, Omphale 2022, scénario central, onglet `Structure_Age_DEP`
- **Unités** : population en milliers, tranches d'âge en **pourcentage**
- **Traitement** : conservation de la ligne Haute-Garonne

### 4. `motorisation_haute_garonne.csv`

Nombre de ménages et de ménages avec voiture dans chaque quartier IRIS.

- **Source** : INSEE, Recensement de la population 2022, base Logement par IRIS
- **Colonnes** :
  - `IRIS` : code du quartier
  - `COM` : code de la commune
  - `P22_RP` : nombre de résidences principales, soit le nombre de ménages
  - `P22_RP_VOIT1P` : ménages disposant d'au moins une voiture
- **Calcul utile** : ménages sans voiture = `P22_RP` - `P22_RP_VOIT1P`
- **Traitement** : filtre sur les communes dont le code commence par `31`, conservation des 4 colonnes ci-dessus
- **Pourquoi cette donnée** : un ménage sans voiture est un client naturel de l'autopartage. Elle permet de repérer les quartiers où Citiz trouvera ses utilisateurs. La motorisation des ménages est citée dans le sujet comme donnée pertinente.

### 5. `contours_communes_haute_garonne.geojson`

Contour de chaque commune, pour dessiner la carte.

- **Source** : geo.api.gouv.fr (service public, à partir des données IGN)
- **Propriétés** : `nom`, `code` (code INSEE), `codeEpci` (intercommunalité), `population`
- **Format** : GeoJSON, coordonnées en longitude / latitude

## Format et points d'attention

- **Séparateur des CSV** : point-virgule `;`
- **Décimales** : écrites avec un point (`0.9`). Les colonnes ont été laissées en texte dans Excel pour ne pas perdre ce format, Fabric les lit directement en nombres.
- **Codes commune et IRIS** : à garder en texte pour ne pas perdre les zéros ou les mélanger avec des nombres.
- **Clé de croisement** :
  - avec les arrêts et stations : rattacher chaque point à son carreau de 200 m
  - avec la motorisation : rattacher chaque carreau à son IRIS (nécessite les contours IRIS)
  - avec les contours : code commune

## Limites

- **Niveaux différents** : la population est au carreau de 200 m, la motorisation par IRIS et les projections par département. Les croisements entre ces niveaux sont des approximations.
- **Secret statistique** : dans les carreaux de moins de 11 ménages, l'INSEE estime les valeurs (`i_est_200` = 1). Ces carreaux, souvent en zone rurale, sont à interpréter avec prudence, et ne doivent pas permettre d'identifier des foyers.
- **Projections** : disponibles seulement à l'échelle du département. Elles indiquent une tendance générale, pas l'évolution quartier par quartier.
- **Dates** : revenus 2021, recensement 2022, projections publiées en 2022.

## Sources

- INSEE, Filosofi 2021, données au carreau de 200 m : https://www.insee.fr/fr/statistiques/8735162
- INSEE, Omphale 2022, projections de population 2018-2070 par département : https://www.insee.fr/fr/statistiques/6652134
- INSEE, Recensement de la population 2022, base Logement par IRIS : https://www.insee.fr/fr/statistiques/8647012
- geo.api.gouv.fr, contours des communes de la Haute-Garonne : https://geo.api.gouv.fr/departements/31/communes?format=geojson&geometry=contour
