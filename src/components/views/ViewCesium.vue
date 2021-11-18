<template>
  <div id="cesium-container"></div>
</template>

<script>
import "cesium/Build/Cesium/Widgets/widgets.css";
import * as Cesium from 'cesium';

export default {
  name: "CesiumGlobeView",
  data() {
    return{
      center: [7.07903, 46.68856],
      defaultheight:100000.,
      viewer:null
    }
  },
  methods: {
    /**
     * Fly to position 
     * 
     * @param {number[]} globecenter position to fly to
     * @param {number} globeheight altitude
     * @param {Viewer} viewer cesium viewer
     */
    flytodirection(globecenter,globeheight,viewer){
      viewer.camera.flyTo({
        destination : Cesium.Cartesian3.fromDegrees(globecenter[0], globecenter[1], globeheight)
      });
    },
    /**
     * Init Cesium globe
     * 
     * @returns {Viewer} viewer from cesium
     */
    setupCesiumGlobe () {
      let viewer = new Cesium.Viewer('cesium-container', {
        terrainProvider: new Cesium.createWorldTerrain()
      });
      viewer.scene.primitives.add(Cesium.createOsmBuildings());
      return viewer;
    }
  },
  mounted() {
    // add cesium ion token to the app
    Cesium.Ion.defaultAccessToken = process.env.VUE_APP_CESIUM_ION_TOKEN;
    
    this.viewer = this.setupCesiumGlobe();
    this.flytodirection(this.center,this.defaultheight,this.viewer)
  },
};
</script>

<style scoped>
#cesium-container {
  height: 500px;
}
</style>