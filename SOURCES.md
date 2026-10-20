# Sources de données

Toutes les données utilisées dans le projet, avec leur lien, leur millésime et leur licence.
Chaque membre de l'équipe complète la section de sa partie.

> **Licences** : la Licence Ouverte (Etalab) demande seulement de citer la source.
> L'**ODbL** demande de citer la source **et** de repartager sous la même licence
> toute base de données dérivée qu'on publierait.

## Affluence (usage réel et générateurs de déplacements)

Explications détaillées des fichiers : voir [`AFFLUENCE.md`](AFFLUENCE.md).

### Flux temps réel (GBFS), enregistrés toutes les 5 minutes

| Donnée | Lien du flux | Page | Licence |
|---|---|---|---|
| Autopartage Citiz Occitanie (GBFS 3.0) | `https://backend.citiz.fr/public/provider/8/gbfs/3.0/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/citiz-autopartage-occitanie) | Licence Ouverte 2.0 |
| Vélos VélôToulouse | `https://api.cyclocity.fr/contracts/toulouse/gbfs/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/velotoulouse-velo-en-libre-service-au-standard-gbfs) | ODbL |
| Scooters YEGO Toulouse (GBFS 2.2) | `https://services.rideyego.com/gbfs/2-2/toulouse/fr/gbfs` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/yego-gbfs-toulouse) | Non spécifiée |
| Vélos Vélomagg Montpellier | `https://gbfs.theta.fifteen.eu/gbfs/montpellier/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/disponibilite-en-temps-reel-des-velos-en-libre-service-velomagg-de-montpellier) | ODbL |
| Vélos NemoVélo Nîmes Métropole | `https://api.gbfs.ecovelo.mobi/nemovelo/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/velos-a-assistance-electrique-en-libre-service-nimes-metropole-nemovelo) | _à vérifier sur la page_ |
| Vélos TLP Mobilités Tarbes-Lourdes | `https://api.gbfs.ecovelo.mobi/tlpmobilites/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/vls-tlp-mobilites) | _à vérifier sur la page_ |
| Vélos et trottinettes Pony Perpignan | `https://proxy.transport.data.gouv.fr/resource/pony-perpignan-gbfs/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/velos-et-trottinettes-pony-perpignan) | _à vérifier sur la page_ |
| Vélos CYCLOlibre Carcassonne | `https://api.gbfs.ecovelo.mobi/cyclolibre/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/cyclolibre-les-velos-electriques-en-libre-service-par-carcassonne) | _à vérifier sur la page_ |
| Vélos Vélo'c Grand Auch | `https://api.gbfs.ecovelo.mobi/auchveloc/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/vls-auch-veloc) | _à vérifier sur la page_ |
| Vélos Vélo d'Aqui Argelès-sur-Mer | `https://api.gbfs.ecovelo.mobi/velodaqui/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/velos-a-assistance-electrique-en-libre-service-argeles-sur-mer-velo-daqui-1) | _à vérifier sur la page_ |
| Vélos VélopartaG Gévaudan | `https://www.mobility-parc.net/gbfs/v3/GEVAUDAN/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/gevaudan-velo-en-libre-service) | _à vérifier sur la page_ |
| Vélos et trottinettes Dott (national, filtré Occitanie) | `https://gbfs.api.ridedott.com/public/v2/countries/fr/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/tier-dott-gbfs-france) | _à vérifier sur la page_ |
| Autopartage Getaround (national, filtré Occitanie) | `https://fr.getaround.com/gbfs/manifest?country_code=FR` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/flotte-getaround-en-libre-service-france) | Non spécifiée |
| Autopartage Clem (national, filtré Occitanie) | `https://gbfs.clem.mobi/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/gbfs-autopartage-publique-clem) | Non spécifiée |
| Autopartage Citiz Développement (national, filtré Occitanie) | `https://backend.citiz.fr/public/provider/12/gbfs/3.0/gbfs.json` | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/citiz-autopartage-developpement) | Licence Ouverte 2.0 |

Période d'enregistrement : du **20/10/2026 à 9h56** au _à compléter_, pour les 15 services.
(Une collecte de test sur 4 services, le 19/10 à partir de 17h04, n'est pas utilisée.)

### Fichiers

| Donnée | Producteur | Millésime | Lien | Licence |
|---|---|---|---|---|
| Géolocalisation des établissements SIRENE | INSEE | sept. 2026 | [data.gouv.fr](https://www.data.gouv.fr/datasets/geolocalisation-des-etablissements-du-repertoire-sirene-pour-les-etudes-statistiques) | Licence Ouverte 2.0 |
| Équipements géolocalisés (BPE) et liste des types d'équipements | INSEE | 2025 | [insee.fr](https://www.insee.fr/fr/statistiques/8217525) | Licence Ouverte |
| Fréquentation annuelle des gares voyageurs | SNCF Gares & Connexions | 2015-2024 | [data.gouv.fr](https://www.data.gouv.fr/datasets/frequentation-en-gares) | ODbL |
| Position des gares voyageurs | SNCF | 2026 | [transport.data.gouv.fr](https://transport.data.gouv.fr/datasets/gares-de-voyageurs-1) | ODbL |
| Trajets domicile-travail (base flux de mobilité) | INSEE, recensement | 2022 | [insee.fr](https://www.insee.fr/fr/statistiques/8582949) | Licence Ouverte |

## Démographie

Explications détaillées des fichiers : voir le fichier de la partie démographie.

| Donnée | Producteur | Millésime | Lien | Licence |
|---|---|---|---|---|
| Population, âges et revenus au carreau de 200 m (Filosofi) | INSEE | 2021 | [insee.fr](https://www.insee.fr/fr/statistiques/8735162) | Licence Ouverte |
| Projections de population 2018-2070 par département (Omphale, scénario central) | INSEE | 2022 | [insee.fr](https://www.insee.fr/fr/statistiques/6652134) | Licence Ouverte |
| Logement par IRIS : ménages et ménages avec voiture | INSEE, recensement | 2022 | [insee.fr](https://www.insee.fr/fr/statistiques/8647012) | Licence Ouverte |
| Contours des communes | geo.api.gouv.fr (à partir des données IGN) | — | [geo.api.gouv.fr](https://geo.api.gouv.fr/departements/31/communes?format=geojson&geometry=contour) | Licence Ouverte |

## Réseaux de transport

_à compléter. Pistes : GTFS Tisséo, GTFS liO Occitanie, GTFS TaM Montpellier, arrêts de transport en France, réseau piéton OpenStreetMap._

| Donnée | Producteur | Millésime | Lien | Licence |
|---|---|---|---|---|
| | | | | |
