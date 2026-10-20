# Données démographiques : Occitanie

Ce dossier rassemble les données de population utilisées pour le Hackathon Mobility (Epitech x Citiz x Microsoft).

Elles servent à savoir **où vivent les habitants**, **comment la population va évoluer** et **quels ménages n'ont pas de voiture**, pour ensuite les croiser avec l'offre de transport.

## Territoire étudié

**Région Occitanie (code région 76), soit 13 départements :**
09 Ariège, 11 Aude, 12 Aveyron, 30 Gard, 31 Haute-Garonne, 32 Gers, 34 Hérault, 46 Lot, 48 Lozère, 65 Hautes-Pyrénées, 66 Pyrénées-Orientales, 81 Tarn, 82 Tarn-et-Garonne.

Ce choix permet de comparer de grandes métropoles bien équipées (Toulouse, Montpellier) avec des zones périurbaines, rurales et de montagne moins desservies. Les projections de population de l'INSEE sont disponibles à l'échelle de la région, ce qui garde toutes les données cohérentes.

## Emplacement dans Fabric

Lakehouse **LakeMobilite** > **Files** > **Data Démographiques**

## Fichiers

| Fichier | Contenu | Niveau géographique | Lignes |
|---|---|---|---|
| `filosofi_200m_occitanie.csv` | Habitants, âges, revenus, logement, coordonnées | Carreau de 200 m | 280 177 |
| `projections_population_occitanie.csv` | Population projetée de 2018 à 2070 | Région | 1 |
| `projections_age_occitanie.csv` | Répartition par âge en 2018 et 2070 | Région | 1 |
| `motorisation_occitanie.csv` | Ménages, ménages avec et sans voiture | IRIS (quartier) | 5 450 |
| `contours_communes_occitanie.geojson` | Contours des communes | Commune | 4 446 |

## Détail des fichiers

### 1. `filosofi_200m_occitanie.csv`

Nombre d'habitants, âges et revenus dans chaque carré de 200 m de l'Occitanie.

- **Source** : INSEE, Filosofi 2021, données au carreau de 200 m
- **Colonnes d'origine INSEE** :
  - `idcar_200m` : identifiant du carreau (contient la position de son coin bas-gauche)
  - `lcog_geo` : code(s) commune. Un carreau à cheval sur plusieurs communes a plusieurs codes séparés par une virgule
  - `ind` : nombre d'habitants
  - `ind_0_3` à `ind_80p` : habitants par tranche d'âge, `ind_inc` : âge inconnu
  - `men`, `men_pauv`, `men_1ind`, `men_5ind`, `men_prop`, `men_fmp` : ménages (total, pauvres, une personne, 5 personnes ou plus, propriétaires, familles monoparentales)
  - `ind_snv` : somme des niveaux de vie des habitants
  - `men_surf`, `men_coll`, `men_mais`, `log_av45` à `log_soc` : surface, type de logement, période de construction, logements sociaux
  - `i_est_200` : vaut 1 si les valeurs du carreau sont estimées par l'INSEE (secret statistique)
- **Colonnes ajoutées** :
  - `com_principale` : premier code commune du carreau (règle retenue pour les carreaux sur plusieurs communes)
  - `dep` : code du département
  - `nb_communes` : nombre de communes touchées par le carreau
  - `x_3035`, `y_3035` : centre du carreau, en mètres (système de coordonnées EPSG:3035, celui de l'INSEE)
  - `lon`, `lat` : centre du carreau en longitude / latitude GPS (EPSG:4326), pour croiser avec les arrêts et stations
- **Traitement** : filtre sur les 13 départements d'Occitanie à partir de `lcog_geo`, puis ajout des colonnes ci-dessus
- **Contrôle** : 280 177 carreaux, environ 5 710 000 habitants, aucun doublon, 28 683 carreaux sur plusieurs communes, 232 177 carreaux avec valeurs estimées

| Département | Carreaux | Habitants |
|---|---|---|
| 09 Ariège | 11 919 | 147 290 |
| 11 Aude | 14 889 | 354 184 |
| 12 Aveyron | 25 165 | 262 802 |
| 30 Gard | 26 509 | 737 499 |
| 31 Haute-Garonne | 42 699 | 1 344 442 |
| 32 Gers | 27 318 | 180 900 |
| 34 Hérault | 22 651 | 1 132 438 |
| 46 Lot | 23 679 | 161 274 |
| 48 Lozère | 7 125 | 70 238 |
| 65 Hautes-Pyrénées | 14 261 | 211 034 |
| 66 Pyrénées-Orientales | 10 885 | 471 518 |
| 81 Tarn | 29 392 | 378 532 |
| 82 Tarn-et-Garonne | 23 685 | 257 728 |

### 2. `projections_population_occitanie.csv`

Évolution prévue de la population de la région, année par année, de 2018 à 2070.

- **Source** : INSEE, Omphale 2022, scénario central, onglet `Population_REG`
- **Unité** : **milliers d'habitants** (5885 = 5 885 000 habitants)
- **Colonnes** : `reg`, `libreg`, puis `pop_2018` à `pop_2070`
- **Traitement** : conservation de la ligne Occitanie, noms de colonnes en minuscules
- **À retenir** : 5 885 000 habitants en 2018, 6 266 000 en 2030, 6 496 000 en 2040

### 3. `projections_age_occitanie.csv`

Répartition de la population par tranche d'âge en 2018 et en 2070, avec l'âge moyen.

- **Source** : INSEE, Omphale 2022, scénario central, onglet `Structure_Age_REG`
- **Unités** : `pop_` en milliers, `age_moy_` en années, `age_` en **pourcentage** de la population
- **Colonnes** : `pop_2018`, `age_moy_2018`, `age_0_5_2018` à `age_95_99_2018`, puis les mêmes pour 2070
- **Traitement** : conservation de la ligne Occitanie, colonnes renommées pour Fabric (`[0,5)_2018` devient `age_0_5_2018`)
- **À retenir** : âge moyen de 42,7 ans en 2018 et 48,4 ans en 2070. Les 75 ans et plus passent d'environ 11 % à près de 20 % de la population.

### 4. `motorisation_occitanie.csv`

Nombre de ménages, avec et sans voiture, dans chaque quartier IRIS.

- **Source** : INSEE, Recensement de la population 2022, base Logement par IRIS
- **Colonnes** :
  - `iris` : code du quartier IRIS
  - `com` : code de la commune
  - `dep` : code du département (ajouté)
  - `p22_rp` : nombre de résidences principales, soit le nombre de ménages
  - `p22_rp_voit1p` : ménages disposant d'au moins une voiture
  - `menages_sans_voiture` : `p22_rp` - `p22_rp_voit1p` (ajouté)
  - `part_sans_voiture` : `menages_sans_voiture` / `p22_rp`, entre 0 et 1 (ajouté)
- **Traitement** : filtre sur les 13 départements, conservation des colonnes utiles, ajout des calculs
- **Contrôle** : 5 450 IRIS, environ 2 888 000 ménages, dont 14,6 % sans voiture. Aucun doublon, aucune incohérence. 2 IRIS sans ménage ont une `part_sans_voiture` vide.
- **Pourquoi cette donnée** : un ménage sans voiture est un client naturel de l'autopartage. Elle permet de repérer les quartiers où Citiz trouvera ses utilisateurs. La motorisation des ménages est citée dans le sujet comme donnée pertinente.

### 5. `contours_communes_occitanie.geojson`

Contour de chaque commune, pour dessiner la carte.

- **Source** : geo.api.gouv.fr (service public, à partir des données IGN)
- **Propriétés** : `nom`, `code` (code INSEE), `codeDepartement`, `codeEpci` (intercommunalité), `population`
- **Format** : GeoJSON, longitude / latitude (EPSG:4326)
- **Contrôle** : 4 446 communes, 13 départements, aucun doublon, aucun contour manquant
- **Dans Fabric** : laissé dans **Files** (un GeoJSON ne se charge pas en table). À utiliser dans Power BI ou dans un notebook Python.

## Format et points d'attention

- **Séparateur des CSV** : point-virgule `;`
- **Décimales** : écrites avec un point (`0.9`)
- **Encodage** : UTF-8
- **Codes commune, IRIS et département** : à garder en **texte**, sinon le zéro de l'Ariège disparaît (`09001` deviendrait `9001`) et les croisements ne marchent plus
- **Noms de colonnes** : en minuscules, sans espace ni caractère spécial, compatibles avec les tables Fabric

## Comment croiser ces données

- **Avec les arrêts et stations** : via `lon` / `lat` du centre des carreaux. Par exemple, compter les arrêts à moins de 400 m (environ 5 minutes à pied) de chaque carreau.
- **Avec la motorisation** : via le code commune (`com_principale` et `com`). Un croisement plus fin par IRIS nécessiterait les contours IRIS.
- **Avec les contours** : via le code commune (`com_principale` et `code`).

## Valeurs manquantes

Certaines cases sont **vides et laissées vides volontairement** :
- `log_soc` (logements sociaux) : 6 532 carreaux
- `ind_inc` (âge inconnu) : 865 carreaux
- `part_sans_voiture` : 2 IRIS sans ménage

Une case vide veut dire « non renseigné », pas zéro. Les remplacer par 0 reviendrait à inventer une valeur. Dans Fabric, elles sont lues comme `null` et sont ignorées dans les sommes. Ces colonnes ne servent pas aux indicateurs principaux (population, ménages, âges et motorisation sont complets).

## Limites

- **Niveaux différents** : la population est au carreau de 200 m, la motorisation par IRIS, les projections par région. Les croisements entre ces niveaux sont des approximations.
- **Secret statistique** : plus de 8 carreaux sur 10 ont des valeurs estimées par l'INSEE (`i_est_200` = 1), car l'Occitanie compte beaucoup de zones peu peuplées. Ces carreaux sont à interpréter avec prudence et ne doivent pas permettre d'identifier des foyers.
- **Carreaux sur plusieurs communes** : rattachés à leur premier code commune, ce qui peut légèrement décaler les totaux par commune.
- **Projections** : disponibles seulement à l'échelle de la région. Elles donnent une tendance générale, pas l'évolution quartier par quartier.
- **Dates** : revenus 2021, recensement 2022, projections publiées en 2022.

## Sources

- INSEE, Filosofi 2021, données au carreau de 200 m : https://www.insee.fr/fr/statistiques/8735162
- INSEE, Omphale 2022, projections de population 2018-2070 par département et région : https://www.insee.fr/fr/statistiques/6652134
- INSEE, Recensement de la population 2022, base Logement par IRIS : https://www.insee.fr/fr/statistiques/8647012
- geo.api.gouv.fr, contours des communes d'Occitanie : https://geo.api.gouv.fr/communes?codeRegion=76&format=geojson&geometry=contour
