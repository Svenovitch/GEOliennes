<template>
  <div class="box">
    <textarea v-model="localite" class="textarea" placeholder="Rechercher" rows="1"></textarea>
    <button class="button" @click="getLocationsInfos(apiURL,localite,apiURLEnd)">Rechercher</button>
  </div>
  <p>Choisir une éolienne:</p>
  <select @change="ZoomOnObjects(eolienne)" v-model="eolienne" position="topleft"> <!-- Affichage d'un menu déroulant qui appelle une fonction de zoom sur objet lorsqu'on séléctionne une éolienne -->
    <option v-for="eolienne in eoliennes" :key="eolienne">{{eolienne}}</option> <!-- Boucle v-for sur les éoliennes contenues dans la liste des éoliennes pour affichage dans le menu déroulant -->
  </select>
  <div id="l-container"></div>
  <div style="font-weight: bold; text-decoration:underline; text-align: left; background-color: #FFFFFF;border-radius: 4px;"></div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from 'leaflet';
import 'leaflet.polylinemeasure';
import axios from 'axios'
import { toFunction } from 'ol/style/Style';
import Interaction from 'ol/interaction/Interaction';
import { forEachCorner } from 'ol/extent';
import { remove } from 'ol/array';

export default {
  name: "LeafletMapView",
  data() {
    return{
      center: [46.68856, 7.07903], //Centre de la carte par défaut
      lmap:null,
      zoom: 10, //Zoom par défaut
      eoliennes : ["Vue générale", "Gibloux1", "Gibloux2", "Glaney1", "Glaney2", "Vuisternens", "Esserta", "Schwyberg1", "Schwyberg2", "Surpierre-Cheiry", "Sonnaz1", "Sonnaz2"], //Liste des éoliennes pour le menu déroulant (v-for)
      localite : "",
      locations : [ //Array contenant les éoliennes avec leur position en longitude/latitude
        ["Gibloux1", 46.67679, 7.02146],
        ["Gibloux2", 46.66365, 7.00686],
        ["Glaney1", 46.67890, 6.87359],
        ["Glaney2", 46.65247, 6.84562],
        ["Vuisternens", 46.62848, 6.90941],
        ["Esserta", 46.61058, 6.94127],
        ["Schwyberg1", 46.68031, 7.25892],
        ["Schwyberg2", 46.69877, 7.26710],
        ["Surpierre-Cheiry", 46.74431, 6.84192],
        ["Sonnaz1", 46.84741, 7.11054],
        ["Sonnaz2", 46.83535, 7.09383]
      ],
      apiURL:"https://api3.geo.admin.ch/rest/services/api/SearchServer?searchText=",
      apiURLEnd: "&type=locations&geometryFormat=geojson",
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
      let basemaps = { // Définition des fonds de plan WMTS
        'Carte nationale': L.tileLayer('https://wmts100.geo.admin.ch/1.0.0/ch.swisstopo.pixelkarte-farbe/default/current/3857/{z}/{x}/{y}.jpeg', {
        attribution: 'Map data &copy; <a href="https://www.map.geo.admin.ch">Swisstopo</a> '
        }),
        'Swissimage': L.tileLayer('https://wmts100.geo.admin.ch/1.0.0/ch.swisstopo.swissimage/default/current/3857/{z}/{x}/{y}.jpeg', {
        attribution: 'Map data &copy; <a href="https://www.map.geo.admin.ch">Swisstopo</a> '
        }),
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

    setupPolylineMeasure () {
      let polylineMeasure =  L.control.polylineMeasure({position:'topleft', unit:'metres', showBearings:false, clearMeasurementsOnStop: false, showClearControl: true, showUnitControl: false})
          polylineMeasure.addTo (this.lmap);//Ajout du contrôle pour mesurer des distances par polylignes
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
      ).setView(mapcenter, mapzoom); //Définition de la carte avec limites zoom et coordonnées centre
      basemapObject.Swissimage.addTo(initmap); //Fond de plan par défaut à l'ouverture de la carte
      L.control.layers(basemapObject).addTo(initmap); //Contrôle pour choisir le fond de plan
    return initmap
    },

    LienViewshedPopup(e) { //Lien pour que le viewshed s'affiche lorsqu'on affiche la popup d'une éolienne
      var location = e.popup._source._popup._content;
      this.LoadViewsheds(location); //Appel à la fonction qui affiche le viewshed
    },

    AffichageMarkers () {
      //Paramètres pour l'icone des éoliennes      
      var iconeoliennes = L.icon({
        iconUrl: require('../../assets/eolienne.png'), //Lien vers l'icone au format "png"
        iconAnchor: [14, 14],//Centrage de l'icone au milieu de ce dernier
        iconSize: [28, 28]//Taille de l'icone
        });
      for (var i = 0; i < this.locations.length; i++) { //Boucle sur la liste des éoliennes et leur position
        var popup = L.popup({closeOnClick: false, autoClose : false }).setContent(this.locations[i][0]); //Affichage de la popup avec le nom de l'éolienne cliquée
        var markers = L.marker([this.locations[i][1], this.locations[i][2]], {icon: iconeoliennes}) //Affichage de l'icône aux coordonnées de chaque éolienne
          .bindPopup(popup)
          .addTo(this.lmap);
      }
    },

    ZoomOnObjects(eolienne) { 
      for(var i = 0; i < this.locations.length; i++) { //Boucle sur la liste des éoliennes et leur position
        if (eolienne==this.locations[i][0]) { //Si le nom de l'éolienne choisie dans le menu déroulant (v-for) correspond à une éolienne existante...
          this.lmap.setView([this.locations[i][1], this.locations[i][2]], 15); //...Déplacement au coordonnées de l'éolienne concernée et zoom
        }
        if (eolienne=='Vue générale') { //Si "Vue générale" est choisi dans le menu...
          this.lmap.setView(this.center, this.zoom); //...retour à la vue d'origine de la carte avec le zoom par défaut
        }         
      };
    },

    LoadViewsheds(eolienne) {
      for(var i = 0; i < this.locations.length; i++) { //Boucle sur la liste des éoliennes et leur position
        this.lmap.eachLayer(function (layer) {
        });
        if (eolienne==this.locations[i][0]) {
          var viewshed = new L.imageOverlay(require('../../assets/'+eolienne+'.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.40}) //Affichage du viewshed en appelant un fichier .png qui porte le même nom que l'éolienne séléctionnée avec une tansparence
          this.TestViewsheds(viewshed) //Appel de la fonction TestViewsheds
        }
      };
    },

    TestViewsheds (viewshed) {
      var count = 0
      var deleteLayer
      this.lmap.eachLayer(function(layer) {
        if (typeof(layer._url) != "undefined") {
          if (layer._url == viewshed._url) {
            count = 1
            deleteLayer = layer
          }
        }
      });
      if (count == 1) {
        this.lmap.removeLayer(deleteLayer);
      }
      else {
        viewshed.addTo(this.lmap);
      };
    },

    async getLocationsInfos(apiURL,localite,apiURLEnd){

      const response = await axios.get(apiURL+localite+apiURLEnd)
      console.log(response.data.features)

      //response status handling: success & error
      if (response.status == 200){
        // Get & return first location
          var LocaliteLat = response.data.features[0].properties.lat
          var LocaliteLon = response.data.features[0].properties.lon
        this.lmap.setView([LocaliteLat, LocaliteLon], 12)
      }

      if (response.status == 404){
        throw "Didn't find any answers";
      }
      else {
        throw "Ouch an unknown error occurred";
      }
    },
  },

  async mounted() {
    try {
      await this.getLocationsInfos(this.apiURL,this.localite,this.apiURLEnd);
    } catch(e) {
      console.error(e);
    }
  },

  mounted() {
    let basemapObject = this.setupBaseMaps();
    this.lmap = this.setupLeafletMap(this.center,this.zoom,basemapObject);
    this.AffichageMarkers();
    this.ZoomOnObjects();
    L.control.scale ({maxWidth:240, metric:true, imperial:false, position: 'bottomleft'}).addTo(this.lmap);
    this.setupPolylineMeasure();
    this.lmap.on('popupopen', this.LienViewshedPopup);
    this.lmap.on('popupclose', this.LienViewshedPopup);
  },
}
</script>

<style scoped>
#l-container {
  height: 500px;
}
/*Paramètres de style des menus dans leaflet */
.menu {
    line-height: 24px;
    font-family: "Arial";
    padding: 5px 5px;
    background-color: #FFFFFF;
    border-radius: 5px;
    width: 200px;
    border: 1px solid;
}
</style>