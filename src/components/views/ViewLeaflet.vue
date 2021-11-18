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