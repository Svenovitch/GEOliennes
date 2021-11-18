<template>
  <div id="l-container"></div>
  <div id="checkboxes"></div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from 'leaflet';

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
    setupLeafletMap (mapcenter,mapzoom) {
      let initmap = L.map("l-container").setView(mapcenter, mapzoom);
      L.tileLayer.wms('https://wms.geo.admin.ch/?SERVICE=WMS&VERSION=1.3.0', {
        layers: 'ch.swisstopo.landeskarte-farbe-10',
        maxZoom: 20,
        attribution: '&copy; Swisstopo | &copy; <a href="https://wms.geo.admin.ch/?SERVICE=WMS&VERSION=1.3.0"></a>'
      }).addTo(initmap);
    return initmap
    },
     
    AffichageMarkers () {
      //Paramètres pour l'icone des éoliennes      
      var locations = [
        ["LOCATION_1", 46.67679,7.02146],
        ["LOCATION_2", 46.66365,7.00686],
        ["LOCATION_3", 46.67890,6.87359],
        ["LOCATION_4", 46.65247,6.84562],
        ["LOCATION_5", 46.62848,6.90941],
        ["LOCATION_6", 46.61058,6.94127],
        ["LOCATION_7", 46.68031,7.25892],
        ["LOCATION_8", 46.69877,7.26710],
        ["LOCATION_9", 46.74431,6.84192],
        ["LOCATION_10", 46.84741,7.11054],
        ["LOCATION_11", 46.83535,7.09383]
      ];
      var iconeoliennes = L.icon({
        iconUrl: require('../../assets/eolienne.png'),
        iconAnchor: [14, 14],//Centrage de l'icone au milieu de ce dernier
        iconSize: [28, 28]//Taille de l'icone
        });
      //Lien vers l'icone au format "png"
      for (var i = 0; i < locations.length; i++) {
        var markers = new L.marker([locations[i][1], locations[i][2]], {icon: iconeoliennes})
          .bindPopup(locations[i][0])
          .addTo(this.lmap);
      }
      return 
    }
  },
  mounted() {
    this.lmap= this.setupLeafletMap(this.center,this.zoom);
    this.AffichageMarkers()
    },
}
</script>

<style scoped>
#l-container {
  height: 500px;
}
</style>