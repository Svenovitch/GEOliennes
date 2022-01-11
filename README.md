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
```
à remplir
```

### Onglet "Vue 2D"
```
à remplir
```

### Onglet "Vue 3D"
```
à remplir
```

## Rapport

### Concept du projet
```
Les projets d’éoliennes font l’objet de nombreux débats. Ils sont souvent mal perçus par la population qui les rejettent presque systématiquement. 
Nous avons remarqué qu’il était parfois difficile de se représenter l’impact qu’ont de tels projets sur notre territoire. Le but de ce géoportail était alors de fournir un support sur lequel représenter les éoliennes dans leur environnement, aussi bien sur une carte 2D qu’en 3 dimensions sur un globe virtuel. 
L’affichage sur la carte 2D permet de visualiser les éoliennes sur divers fonds de plan (carte nationale et imagerie aérienne par exemple). Il est possible de zoomer sur une éolienne projetée en la choisissant dans un menu déroulant et d’afficher la portion de territoire depuis laquelle elle est visible en cliquant sur son icône. Une fonction de recherche par localité est aussi disponible, ainsi qu’un outil de mesure de distance.
L’affichage 3D permet lui de visualiser un globe virtuel sur lequel les éoliennes, les bâtiments et les arbres sont représentés. Une fonction offre la possibilité à l’utilisateur de se projeter au sol, ce qui lui permet de s’immerger dans le paysage.
```