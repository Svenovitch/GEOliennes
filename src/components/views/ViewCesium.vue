<template>
  <button v-on:click="groundView">Vue au sol</button>
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
      defaultheight:10000.,
      viewer:null,
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

    // defini la source SwissBuilding
    getSwissBuilding(){
      return new Cesium.Cesium3DTileset({
        url: "https://vectortiles0.geo.admin.ch/3d-tiles/ch.swisstopo.swisstlm3d.3d/20190313/tileset.json"
      });
    },
    // defini la source SwissTLM
    getSwissTLM(){
      return new Cesium.Cesium3DTileset({
        url: "https://vectortiles0.geo.admin.ch/3d-tiles/ch.swisstopo.vegetation.3d/20190313/tileset.json"
      });
    },

    groundView(){
      var viewer = this.viewer
      let posCam = viewer.scene.camera.positionCartographic
      let latCam = posCam.latitude
      let longCam = posCam.longitude
      Cesium.sampleTerrainMostDetailed(viewer.terrainProvider, [posCam])
      .then(function(samples) {
        let groundHeight = samples[0].height;
        var destHeight = groundHeight + 2;
        viewer.camera.flyTo({
          destination : Cesium.Cartesian3.fromRadians(longCam, latCam, destHeight),
          orientation: {
            heading: Cesium.Math.toRadians(90.0),
            pitch: Cesium.Math.toRadians(0.0),
            roll: 0.0,
          },
        });
      });
    },

    // Affiche les éoliennes
    displayEolienne(){
      var viewer = this.viewer
      var uri
      var promise = Cesium.IonResource.fromAssetId(752612)
      .then(function (resource) {
        uri = resource
      });

      for (var i = 0; i < this.locations.length; i++) {
        let position = Cesium.Cartographic.fromDegrees(this.locations[i][2],this.locations[i][1])
        let location = Cesium.Cartesian3.fromDegrees(this.locations[i][2],this.locations[i][1])
        Cesium.sampleTerrainMostDetailed(this.viewer.terrainProvider, [position])
        .then(function(samples) {
          let groundHeight = samples[0].height;
          viewer.entities.add({
            position: Cesium.Cartesian3.fromRadians(position.longitude,position.latitude, groundHeight),
            model : {
              uri: uri,
              //gltf : '../../assets/scene.glb',
              //heightReference : Cesium.HeightReference.CLAMP_TO_GROUND,
              scale : 15,
              color : Cesium.Color.WHITE,
            },
//            ellipse: {
//              semiMinorAxis: 5,
//              semiMajorAxis: 5,
//              height: groundHeight,
//              extrudedHeight: groundHeight + 90,
//              material: Cesium.Color.AQUA,
//            },
          });
          viewer.entities.add({
            position: location,
            ellipse: {
              semiMinorAxis: 30,
              semiMajorAxis: 30,
              material: Cesium.Color.AQUA,
            },
          });
        });
      };
    },

    /**
     * Init Cesium globe
     * 
     * @returns {Viewer} viewer from cesium
     */
    setupCesiumGlobe () {
      // Etendue de la Suisse
      let rectangle = Cesium.Rectangle.fromDegrees(
        5.013926957923385,
        45.35600133779394,
        11.477436312994008,
        48.27502358353741
      );
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
          rectangle: rectangle
        }),
        fullscreenButton: false,
        sceneModePicker: false,
        selectionIndicator: false,
        timeline: false,
        animation: false,
        shouldAnimate: true,
        geocoder: true,
        navigationInstructionsInitiallyVisible: false,
        navigationHelpButton: false,
        scene3DOnly: true
      });
      return viewer
    }
  },

  mounted() {
    // add cesium ion token to the app
    Cesium.Ion.defaultAccessToken = process.env.VUE_APP_CESIUM_ION_TOKEN;

    // homeview sur canton Fribourg
    var extent = Cesium.Rectangle.fromDegrees(6.72826,46.42755,7.39027,47.01382);
    Cesium.Camera.DEFAULT_VIEW_RECTANGLE = extent;
    Cesium.Camera.DEFAULT_VIEW_FACTOR = 0;

    this.viewer = this.setupCesiumGlobe();
    // Transparence du MNT, décommenter pour faire le calcul de profondeur
    this.viewer.scene.globe.depthTestAgainstTerrain = true;
    
    // Ajoute Swissbuilding et SwissTLM
    this.viewer.scene.primitives.add(this.getSwissBuilding());
    this.viewer.scene.primitives.add(this.getSwissTLM());

    // Ajoute les éoliennes
    this.displayEolienne();

  },
};
</script>

<style scoped>
#cesium-container {
  height: 500px;
}
</style>