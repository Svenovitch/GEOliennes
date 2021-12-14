<template>
  <div id="l-container"></div>
  <div style="font-weight: bold; text-decoration:underline; text-align: left; background-color: #FFFFFF;border-radius: 4px;"></div>
  <select v-model="eolienne">
    <option v-for="eolienne in eoliennes" :key="eolienne">{{eolienne}}</option>
    <option v-on:change="ZoomOnObjects()"></option>
  </select>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from 'leaflet';
import 'leaflet.polylinemeasure';
import { toFunction } from 'ol/style/Style';
import Interaction from 'ol/interaction/Interaction';

export default {
  name: "LeafletMapView",
  data() {
    return{
    eoliennes : ["Gibloux 1", "Gibloux 2", "Glâney 1", "Glâney 2", "Vuisternens", "Esserta", "Schwyberg 1", "Schwyberg 2", "Surpierre-Cheiry", "Sonnaz 1", "Sonnaz 2"],
    locations : [
      ["Gibloux 1", 46.67679, 7.02146],
      ["Gibloux 2", 46.66365, 7.00686],
      ["Glâney 1", 46.67890, 6.87359],
      ["Glâney 2", 46.65247, 6.84562],
      ["Vuisternens", 46.62848, 6.90941],
      ["Esserta", 46.61058, 6.94127],
      ["Schwyberg 1", 46.68031, 7.25892],
      ["Schwyberg 2", 46.69877, 7.26710],
      ["Surpierre-Cheiry", 46.74431, 6.84192],
      ["Sonnaz 1", 46.84741, 7.11054],
      ["Sonnaz 2", 46.83535, 7.09383]
    ],
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
      let viewsheds = {
        'Esserta' : L.imageOverlay(require('../../assets/Esserta.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Gibloux1' : L.imageOverlay(require('../../assets/Gibloux1.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Gibloux2' : L.imageOverlay(require('../../assets/Gibloux2.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Glaney1' : L.imageOverlay(require('../../assets/Glaney1.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Glaney2' : L.imageOverlay(require('../../assets/Glaney2.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Surpierre-Cheiry' : L.imageOverlay(require('../../assets/Surpierre-Cheiry.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Sonnaz1' : L.imageOverlay(require('../../assets/Sonnaz1.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Sonnaz2' : L.imageOverlay(require('../../assets/Sonnaz2.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Schwyberg1' : L.imageOverlay(require('../../assets/Schwyberg1.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Schwyberg2' : L.imageOverlay(require('../../assets/Schwyberg2.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
        'Vuisternens' : L.imageOverlay(require('../../assets/Vuisternens.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.60}).addTo(this.lmap),
      };
      L.control.layers(viewsheds).addTo(this.lmap);
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
      var iconeoliennes = L.icon({
        iconUrl: require('../../assets/eolienne.png'),
        iconAnchor: [14, 14],//Centrage de l'icone au milieu de ce dernier
        iconSize: [28, 28]//Taille de l'icone
        });
      //Lien vers l'icone au format "png"
      for (var i = 0; i < this.locations.length; i++) {
        var markers = L.marker([this.locations[i][1], this.locations[i][2]], {icon: iconeoliennes})
          .bindPopup(this.locations[i][0])
          .addTo(this.lmap);
      }
      return
    },

    ZoomOnObjects(){
      //var test = document.getElementById('liste').value //Récupere la valeur dans la liste 
      if (this.locations[0] == "Générale") { //Création de la condition
        var map_zoom = this.lmap.setView(this.center, this.zoom); //Changement du zoom
        }
      if (this.locations.eolienne == "Gibloux 1") { //Création de la condition
        var map_zoom = this.lmap.setView([locations.lat, locations.long], 15); //Changement du zoom
        }
      if (test == "Gibloux 2") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[1][1], locations[1][2]], 15);//Changement du zoom
        }
      if (test == "Glâney 1") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[2][1], locations[2][2]], 15);//Changement du zoom
        }
      if (test == "Glâney 2") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[3][1], locations[3][2]], 15);//Changement du zoom
        }
      if (test == "Vuisternens") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[4][1], locations[4][2]], 15);//Changement du zoom
        }
      if (test == "Esserta") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[5][1], locations[5][2]], 15);//Changement du zoom
        }
      if (test == "Schwyberg 1") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[6][1], locations[6][2]], 15);//Changement du zoom
        }
      if (test == "Schwyberg 2") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[7][1], locations[7][2]], 15);//Changement du zoom
        }
      if (test == "Surpierre-Cheiry") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[8][1], locations[8][2]], 15);//Changement du zoom
        }
      if (test == "Sonnaz 1") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[9][1], locations[9][2]], 15);//Changement du zoom
        }
      if (test == "Sonnaz 2") {//Création de la condition
        var map_zoom = this.lmap.setView([locations[10][1], locations[10][2]], 15);//Changement du zoom
        }
      return
    },
  
  },
  mounted() {
    let basemapObject = this.setupBaseMaps();
    this.lmap = this.setupLeafletMap(this.center,this.zoom,basemapObject);
    this.AffichageMarkers();
    this.AffichageViewsheds();
    L.control.scale ({maxWidth:240, metric:true, imperial:false, position: 'bottomleft'}).addTo(this.lmap);
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