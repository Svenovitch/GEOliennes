<template>
  <div class="box">
    <textarea v-model="localite" class="textarea" placeholder="Rechercher" rows="1"></textarea>
    <button class="button" @click="getLocationsInfos(apiURL,localite,apiURLEnd)">Rechercher</button>
  </div>
  <select @change="ZoomOnObjects(eolienne)" v-model="eolienne" position="topleft">
    <option v-for="eolienne in eoliennes" :key="eolienne">{{eolienne}}</option>
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
      center: [46.68856, 7.07903],
      lmap:null,
      zoom: 10,
      eoliennes : ["Vue générale", "Gibloux1", "Gibloux2", "Glaney1", "Glaney2", "Vuisternens", "Esserta", "Schwyberg1", "Schwyberg2", "Surpierre-Cheiry", "Sonnaz1", "Sonnaz2"],
      localite : "",
      locations : [
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
      let basemaps = {
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

    LienViewshedPopup(e) {
      var location = e.popup._source._popup._content;
      this.LoadViewsheds(location);
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
        var popup = L.popup({closeOnClick: false, autoClose : false }).setContent(this.locations[i][0]);
        var markers = L.marker([this.locations[i][1], this.locations[i][2]], {icon: iconeoliennes})
          .bindPopup(popup)
          .addTo(this.lmap);
      }
      return
    },

    ZoomOnObjects(eolienne) {
      for(var i = 0; i < this.locations.length; i++) {
        if (eolienne==this.locations[i][0]) {
          this.lmap.setView([this.locations[i][1], this.locations[i][2]], 15); //Changement du zoom
        }
        if (eolienne=='Vue générale') {
          this.lmap.setView(this.center, this.zoom);
        }         
      };
      return 
    },

    LoadViewsheds(eolienne) {
      for(var i = 0; i < this.locations.length; i++) {
        this.lmap.eachLayer(function (layer) {
        });
        if (eolienne==this.locations[i][0]) {
          var viewshed = new L.imageOverlay(require('../../assets/'+eolienne+'.png'), [[46.4344535851,6.62326508105], [47.0140361051,7.38658291045]], {opacity: 0.40})
          this.TestViewsheds(viewshed)
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
  height: 600px;
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