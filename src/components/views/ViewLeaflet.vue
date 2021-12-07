<template>
  <div id="l-container"></div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from 'leaflet';
import 'leaflet.polylinemeasure';

export default {
  name: "LeafletMapView",
  data() {
    return {
      center: [46.68856, 7.07903],
      lmap:null,
      zoom: 10
    }
  },
  methods: {
    /**
     * Init Leaflet map
     * 
     * @param {number[]} mapcenter center of the map in EPSG:3857
     * @param {number} mapzoom zommlevel
     * @returns {Map} initmap new leaflet map
     */

    setupBaseMaps () {
      let basemaps = {
        'Carte nationale': L.tileLayer('https://wmts100.geo.admin.ch/1.0.0/ch.swisstopo.pixelkarte-farbe/default/current/3857/{z}/{x}/{y}.jpeg'
        ),
        'Swissimage': L.tileLayer('https://wmts100.geo.admin.ch/1.0.0/ch.swisstopo.swissimage/default/current/3857/{z}/{x}/{y}.jpeg'
        ),
        'OSM': L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw', {
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, ' +
            'Imagery <a href="https://www.mapbox.com/">Mapbox</a>',
          id: 'mapbox/streets-v11',
          tileSize: 512,
          zoomOffset: -1
        }),
      };
    return basemaps
    },

    AffichageViewsheds () {
      var ViewshedUrl = require('../../assets/Vuisternens.png'),
        ViewshedBounds = [[46.4354657239999966, 6.6232650810000004], [47.0140361050000024, 7.3865939730000001]];
        L.imageOverlay(ViewshedUrl, ViewshedBounds, {opacity: 0.60}).addTo(this.lmap); 
      return
    },

    MenuZoom () {
      var zoom_to = L.control({position: "topleft"});//Position en haut à gauche du contrôle
      zoom_to.onAdd = function(lmap) {
        var div = L.DomUtil.create("div", "menu");//Fait le lien avec le css
        L.DomEvent.on(div, 'mouseover', L.DomEvent.stopPropagation);//Permet que la souris passe sur la fenêtre et pas sur la map
        L.DomEvent.on(div, 'click', L.DomEvent.stopPropagation);//Permet que l'on clique sur la bouton et pas sur la map
        div.innerHTML = //Définit un division HTML dans le javascript
        '<div style="font-weight: bold; text-decoration:underline; text-align: left; background-color: #FFFFFF;border-radius: 4px;"><p>Eoliennes: <br></div>'+ //Créer le titre et le style 
          '<select id="liste" onchange="zoom(this.value)">'+ //Créer une liste déroulante
            '<option value="Gibloux 1">Gibloux 1</option>'+
            '<option value="Gibloux 2">Gibloux 2</option>'+
            '<option value="Glâney 1">Glâney 1</option>'+
            '<option value="Glâney 2">Glâney 2</option>'+
            '<option value="Vuisternens">Vuisternens</option>'+
            '<option value="Esserta">Esserta</option>'+
            '<option value="Schwyberg 1">Schwyberg 1</option>'+
            '<option value="Schwyberg 2">Schwyberg 2</option>'+
            '<option value="Surpierre-Cheiry">Surpierre-Cheiry</option>'+
            '<option value="Sonnaz 1">Sonnaz 1</option>'+
            '<option value="Sonnaz 2">Sonnaz 2</option>'+
          '</select>'+
        '</p>'
        return div;
      };
      zoom_to.addTo(this.lmap);//Ajoute à la carte
    return
    },

    setupPolylineMeasure () {
      let polylineMeasure =  L.control.polylineMeasure({position:'topleft', unit:'metres', showBearings:false, clearMeasurementsOnStop: false, showClearControl: true, showUnitControl: false})
          polylineMeasure.addTo (this.lmap);
          //Code pour les debugs 
          function debugevent(e) { console.debug(e.type, e, polylineMeasure._currentLine) }

                this.lmap.on('polylinemeasure:toggle', debugevent);
                this.lmap.on('polylinemeasure:start', debugevent);
                this.lmap.on('polylinemeasure:resume', debugevent);
                this.lmap.on('polylinemeasure:finish', debugevent);
                this.lmap.on('polylinemeasure:change', debugevent);
                this.lmap.on('polylinemeasure:clear', debugevent);
                this.lmap.on('polylinemeasure:add', debugevent);
                this.lmap.on('polylinemeasure:insert', debugevent);
                this.lmap.on('polylinemeasure:move', debugevent);
                this.lmap.on('polylinemeasure:remove', debugevent);
    return polylineMeasure
    },           

    setupLeafletMap (mapcenter,mapzoom,basemapObject) {
      let initmap = L.map("l-container", {
        maxZoom: 18,
        minZoom: 7,
        maxBounds:[
            [45.680, 5.130],
            [47.860, 11.420]
            ],
      }
      ).setView(mapcenter, mapzoom);
      basemapObject.Swissimage.addTo(initmap);
      L.control.layers(basemapObject).addTo(initmap);
    return initmap
    },

    AffichageMarkers () {
      //Paramètres pour l'icone des éoliennes      
      var locations = [
        ["Gibloux 1", 46.67679,7.02146],
        ["Gibloux 2", 46.66365,7.00686],
        ["Glâney 1", 46.67890,6.87359],
        ["Glâney 2", 46.65247,6.84562],
        ["Vuisternens", 46.62848,6.90941],
        ["Esserta", 46.61058,6.94127],
        ["Schwyberg 1", 46.68031,7.25892],
        ["Schwyberg 2", 46.69877,7.26710],
        ["Surpierre-Cheiry", 46.74431,6.84192],
        ["Sonnaz 1", 46.84741,7.11054],
        ["Sonnaz 2", 46.83535,7.09383]
      ];
      var iconeoliennes = L.icon({
        iconUrl: require('../../assets/eolienne.png'),
        iconAnchor: [14, 14],//Centrage de l'icone au milieu de ce dernier
        iconSize: [28, 28]//Taille de l'icone
        });
      //Lien vers l'icone au format "png"
      for (var i = 0; i < locations.length; i++) {
        var markers = L.marker([locations[i][1], locations[i][2]], {icon: iconeoliennes})
          .bindPopup(locations[i][0])
          .addTo(this.lmap);
      }
      return markers
    },

    ZoomOnObjects(){
      var test = document.getElementById('liste').value //Récupere la valeur dans la liste 
      if (test == "Gibloux 1") { //Création de la condition
        var map_zoom = this.lmap.setView([locations[0][1], locations[0][2]], 10); //Changement du zoom
        }
      if (test == "Gibloux 2") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[1][1], locations[1][2]], 10);//Changement du zoom
        }
      if (test == "Glâney 1") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[2][1], locations[2][2]], 10);//Changement du zoom
        }
      if (test == "Glâney 2") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[3][1], locations[3][2]], 10);//Changement du zoom
        }
      if (test == "Vuisternens") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[4][1], locations[4][2]], 10);//Changement du zoom
        }
      if (test == "Esserta") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[5][1], locations[5][2]], 10);//Changement du zoom
        }
      if (test == "Schwyberg 1") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[6][1], locations[6][2]], 10);//Changement du zoom
        }
      if (test == "Schwyberg 2") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[7][1], locations[7][2]], 10);//Changement du zoom
        }
      if (test == "Surpierre-Cheiry") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[8][1], locations[8][2]], 10);//Changement du zoom
        }
      if (test == "Sonnaz 1") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[9][1], locations[9][2]], 10);//Changement du zoom
        }
      if (test == "Sonnaz 2") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[10][1], locations[10][2]], 10);//Changement du zoom
        }
      return
    },
  
  },
  mounted() {
    let basemapObject = this.setupBaseMaps();
    this.lmap = this.setupLeafletMap(this.center,this.zoom,basemapObject);
    this.AffichageMarkers();
    L.control.scale ({maxWidth:240, metric:true, imperial:false, position: 'bottomleft'}).addTo(this.lmap);
    this.AffichageViewsheds();
    this.setupPolylineMeasure();
    this.MenuZoom ();
    this.ZoomOnObjects();
    },
}
</script>

<style scoped>
#l-container {
  height: 600px;
}
/*Paramètres de style des menus dans leaflet */
.menu {
    line-height: 24px;
    font-family: "Arial";
    padding: 5px 5px;
    background-color: #FFFFFF ;
    border-radius: 5px;
    width: 200px;
    border: 1px solid;
}
</style>