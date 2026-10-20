# <u>Données transport – Haute-Garonne (31)</u>

Page de départ pour tout l'Occitanie : https://transport.data.gouv.fr/datasets/region/76

## <u>Transport public (GTFS)</u>

Source	Contenu	Où la trouver :

Tisséo	Bus, métro, tram, téléphérique de Toulouse et agglo	Page de départ, chercher « Réseau urbain Tisséo »

liO Occitanie	Bus interurbains de toute la région	Page de départ, chercher « Réseau interurbain liO Occitanie »

TaM	Réseau urbain de Montpellier	Page de départ (deux versions : prendre la plus récente)

Tango	Réseau de Nîmes	Page de départ

Sankéo	Réseau de Perpignan	Page de départ

Alliance	Réseau d'Auch	Page de départ

Autres petits réseaux	Béziers, Albi, Tarbes, Castres, Montauban, Rodez, etc.	Page de départ avec filtre GTFS

Arrêts de transport en France	Agrégat national des arrêts de tous les réseaux (non dédupliqué)	https://www.data.gouv.fr/datasets/arrets-de-transport-en-france

## <u>Autopartage (GBFS)</u>

Source	Contenu	Lien :

Citiz Occitanie	Stations Citiz (Toulouse, Albi, Cahors, Tarbes, Millau, Castelnaudary…)	https://transport.data.gouv.fr/datasets?type=vehicles-sharing&q=citiz

Getaround	Voitures de particuliers, sans stations fixes (GBFS v3)	https://transport.data.gouv.fr/datasets?type=vehicles-sharing&q=getaround
Stations d'autopartage Toulouse Métropole	Places réservées à l'autopartage	https://www.data.gouv.fr/en/datasets/stations-d-auto-partage-tm/

Source d'origine des stations TM : https://data.toulouse-metropole.fr/explore/dataset/stations-d-auto-partage/

## <u>Vélos et trottinettes (GBFS)</u>

Source	Contenu	Où la trouver :

VélôToulouse	Stations de vélos en libre-service	Page de départ

YEGO	Scooters en free-floating à Toulouse	Page de départ

Vélomagg	Vélos en libre-service de Montpellier	Page de départ

Catalogue GBFS mondial	Pour trouver d'autres opérateurs (filtre FR)	https://github.com/MobilityData/gbfs/blob/master/systems.csv

## <u>Covoiturage et contexte (optionnel)</u>

Source	Contenu	Où la trouver :

Lieux de covoiturage Haute-Garonne	Points d'ancrage intermodaux	Page de départ

Lieux de covoiturage Région Occitanie	Idem, à l'échelle régionale	Page de départ

ZFE Toulouse Métropole	Zone à faibles émissions	Page de départ

## <u>Base de calcul</u>

Source	Contenu	Lien :

OSM Midi-Pyrénées	Réseau piéton pour les isochrones 5 min (.osm.pbf)	https://download.geofabrik.de/europe/france.html

OSM Languedoc-Roussillon	Idem pour l'est de l'Occitanie	https://download.geofabrik.de/europe/france.html

API Géo (communes)	Limites communales, remplace ADMIN EXPRESS	https://geo.api.gouv.fr/communes?codeRegion=76&format=geojson&geometry=contour

ADMIN EXPRESS (IGN)	Limites officielles (prendre GeoPackage, Lambert 93)	https://geoservices.ign.fr/adminexpress

## <u>Population (pour calculer l'accès par habitant)</u>

Source	Contenu	Lien :

INSEE carreaux 200 m (Filosofi)	Population par carreau de 200 m	https://www.insee.fr/fr/statistiques/8735162
INSEE projections Omphale	Population projetée 2018-2070	https://www.insee.fr/fr/statistiques/6652134

## <u>Formats</u>
GTFS : garder le .zip
GBFS : utiliser l'URL JSON (gbfs.json, puis station_information.json)
Points et zones : GeoJSON
OSM : .osm.pbf
Projections : importer en EPSG:4326, reprojeter en EPSG:2154 pour les distances
