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
        minZoom: 8,
        maxBounds:[
            [45.680, 5.130],
            [47.860, 11.420]
            ],
        attribution: '&copy; <a href="https://map.geo.admin.ch">Swisstopo</a>',
      }).addTo(initmap);
      //Paramètres pour l'icone des éoliennes
      var iconeoliennes = L.Icon.extend({
      options: {
        shadowUrl: null,
        iconAnchor: new L.Point(14, 14),//Centrage de l'icone au milieu de ce dernier
        iconSize: new L.Point(28, 28),//Taille de l'icone
        }
      });
      //Lien vers l'icone au format "png"
      var infoiconeoliennes = new iconeoliennes({iconUrl: 'eolienne.png'});
      var marker = L.marker([46.67679, 7.02146],{icon: infoiconeoliennes}).addTo(initmap); //Ajoute un marker
      return initmap
    }
  },
  mounted() {
    this.lmap= this.setupLeafletMap(this.center,this.zoom);
  }
};
</script>

<style scoped>
#l-container {
  height: 500px;
}
</style>