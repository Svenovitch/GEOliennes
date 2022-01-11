# GEOliennes - Géoportail de visualisation d'éoliennes

C'est un géoportail de visualisation d'éoliennes basé sur le framework Vuejs.

Il inclut:
- vuejs: 3.2
    - vue-router
    - axios
- cesium: 1.85
- leaflet: 1.7.1


## Installation

First download the project with Git:

```
git clone https://github.com/thibaud-c/seed-vuejs3.0-carto2D-3D.git your_project_name
```

Then open the folder in your command line, and install the needed dependencies:

```
cd your_project_name
npm install
```

Finally create a file .env at your project root to provide your Cesium ion key:

<sup>_an example of .env is provided in .env.example_</sup>
```js
VUE_APP_CESIUM_ION_TOKEN=YOUR_CESIUM_ION_KEY
```

## Run

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

## Documentation utilisateur

### Général

à remplir


### Onglet "Vue 2D"

à remplir


### Onglet "Vue 3D"

à remplir


## Rapport

### Concept du projet

Les projets d’éoliennes font l’objet de nombreux débats. Ils sont souvent mal perçus par la population qui les rejettent presque systématiquement.

Nous avons remarqué qu’il était parfois difficile de se représenter l’impact qu’ont de tels projets sur notre territoire. Le but de ce géoportail était alors de fournir un support sur lequel représenter les éoliennes dans leur environnement, aussi bien sur une carte 2D qu’en 3 dimensions sur un globe virtuel.

L’affichage sur la carte 2D permet de visualiser les éoliennes sur divers fonds de plan (carte nationale et imagerie aérienne par exemple). Il est possible de zoomer sur une éolienne projetée en la choisissant dans un menu déroulant et d’afficher la portion de territoire depuis laquelle elle est visible en cliquant sur son icône. Une fonction de recherche par localité est aussi disponible, ainsi qu’un outil de mesure de distance.

L’affichage 3D permet lui de visualiser un globe virtuel sur lequel les éoliennes, les bâtiments et les arbres sont représentés. Une fonction offre la possibilité à l’utilisateur de se projeter au sol, ce qui lui permet de s’immerger dans le paysage.

### Données utilisées

#### Leaflet

Premièrement, les données de bases ont été intégrées à la carte brute déjà existante dans le code source. Il s’agit des éléments suivants :
- Cartes de base en WMTS : SWISSIMAGE et Carte nationale repris de Swisstopo et fond de plan de OpenStreetMap.
- Coordonnée du centre de la carte pour affichage lors de la première ouverture.
- Niveau de zoom par défaut à l’ouverture.
- Liste contenant les noms des éoliennes ainsi que leur position en latitude/longitude.
- API REST de geo.admin.ch pour la recherche de lieux par noms.
- Viewsheds en .PNG issus d'analyses de visibilité sur QGIS.

#### Cesium

- Géoservices 3D Tiles swisstopo :
    - swissALTI3D, modèle numérique de terrain ;
    - swissTLM3D, modèle topographique du paysage (bâtiments et végétation).
- Géoservice WMTS swisstopo :
    - swissimage, orthophoto aérienne.
- Modèle 3D « Turbine », par Peter Primini, CC Attribution (sur sketchfab), testé en local puis publié sur le portail Cesium Ion.
- Emplacements des éoliennes approximatifs issus des zones d’intérêts du Plan directeur cantonal de Fribourg.

### Démarche

L’architecture de base du projet est issue du dépôt Github https://github.com/thibaud-c/seed-vuejs3.0-carto2D-3D. Après l’avoir copié sur un dépôt personnel, chacun a pu collaborer facilement grâce aux outils de versioning de Git. À ce stade, la répartition des tâches s’est effectuée principalement par composant et librairie utilisée, puis par fonctionnalité.

#### Page d'accueil

La structuration de la page d’accueil se base sur la librairie Bulma et son outil d’affichage d’éléments par tuiles. Une première partie présente l’équipe et une deuxième décrit sommairement le projet.

#### Leaflet

Premièrement, les données de bases pré-citées ont été intégrées à la carte brute déjà existante dans le code source. À partir de là, différentes fonctions ont été implémentées qui font appel à ces données de base :
- Affichage de markers avec icône pour les éoliennes. Apparition d’une popup avec son nom lors d’un clic.
- Zoom sur objet lié à un menu déroulant codé en VueJS (v-for sur la liste des éoliennes). Lorsqu’une sélection est faite, la vue se déplace aux coordonnées de l’éolienne avec un niveau de zoom défini.
- Affichage du viewsheds sous forme d’une image qui s’affiche par-dessus la carte (L.imageOverlay). Cette dernière apparaît lorsque l’on clique sur le marker. Le calcul des viewsheds a été fait au préalable sur QGIS (plugin “viewshed analysis”) à l’aide du MNS raster de Swisstopo couplé aux positions des éoliennes projetées dans le Canton de Fribourg et leur hauteur fixée à 90 mètres au centre des pales.

Au niveaux des outils ajoutés à l’application, on peut citer la mesure de distance via le dessin de polylignes (librairie l.polylinemeasure) et la recherche de localités ou de lieux codé sous forme de promesse à travers la librairie Axios et en faisant appel à l’API de recherche REST de Swisstopo.

#### Cesium

Les premières modifications apportées au composant Cesium concernent l’initialisation de l’interface. On mentionnera notamment les points suivants :
- Configuration de la clé Cesium Ion ;
- Tri des éléments de l’interface à afficher ;
- Modification des paramètres par défaut dépendants des éléments de l’interface masqués, comme
    - activer les animations,
    - activer le test de profondeur des éléments affichés,
- Modification des données de base du terrain et de son mappage, afin d’avoir un modèle de terrain précis et une visualisation « photoréaliste » avec le placage de la dernière swissimage.
Ensuite, plusieurs éléments 3D ont été ajoutés au viewer :
- Ajout des données des bâtiments et de la végétation de swisstopo, afin de compléter le paysage 3D et d’offrir une visibilité cohérente avec le MNS utilisé pour le calcul des viewsheds ;
- Ajout des modèles 3D d’éoliennes et de cercles concentriques pour le repérage.

La fonction principale du composant a été de créer une commande afin de projeter la position de la caméra à 2 mètres du sol afin de visualiser le paysage et l’impact des éoliennes sur celui-ci à la première personne.

![Vue du Mont Gibloux avec végétation](./images_rapport/gibloux_avec_veget.png)![Vue du Mont Gibloux sans végétation](./images_rapport/gibloux_avec_veget.png)
