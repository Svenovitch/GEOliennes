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

    // defini la source swissbuilding
     getCesiumTileset(){
      return new Cesium.Cesium3DTileset({
        url: "https://vectortiles0.geo.admin.ch/3d-tiles/ch.swisstopo.swisstlm3d.3d/20190313/tileset.json"
      });
    },
    /**
     * Init Cesium globe
     * 
     * @returns {Viewer} viewer from cesium
     */
    setupCesiumGlobe () {
      // Etendue de la Suisse
//      let rectangle = Cesium.Rectangle.fromDegrees(
//        5.013926957923385,
//        45.35600133779394,
//        11.477436312994008,
//        48.27502358353741
//      ),
      let viewer = new Cesium.Viewer('cesium-container', {
        baseLayerPicker: false,
        terrainProvider: new Cesium.CesiumTerrainProvider({
          url: "//3d.geo.admin.ch/1.0.0/ch.swisstopo.terrain.3d/default/20160115/4326/"
        }),
        imageryProvider: new Cesium.UrlTemplateImageryProvider({
          // Swissimage
          url: "//wmts20.geo.admin.ch/1.0.0/ch.swisstopo.swissimage-product/default/current/4326/{z}/{x}/{y}.jpeg",
          minimumLevel: 8,
          maximumLevel: 17,
          tilingScheme: new Cesium.GeographicTilingScheme({
            numberOfLevelZeroTilesX: 2,
            numberOfLevelZeroTilesY: 1
          }),
//          rectangle: rectangle
        }),
        fullscreenButton: false,
        homeButton: false,
        sceneModePicker: false,
        selectionIndicator: false,
        timeline: false,
        animation: false,
        geocoder: true,
        navigationInstructionsInitiallyVisible: false,
        navigationHelpButton: false,
        scene3DOnly: true
      });
    }
  },

  mounted() {
    // add cesium ion token to the app
    Cesium.Ion.defaultAccessToken = process.env.VUE_APP_CESIUM_ION_TOKEN;
    
    this.viewer = this.setupCesiumGlobe();
    this.flytodirection(this.center,this.defaultheight,this.viewer)  
    
    // Ajoute Swissbuilding
    this.viewer.scene.primitives.add(getCesiumTileset());
  },
};
</script>

<style scoped>
#cesium-container {
  height: 500px;
}
</style>