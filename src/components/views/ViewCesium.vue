<template>
  <button class="button" v-on:click="groundView">Vue au sol</button>
  <div id="cesium-container"></div>
</template>

<script>
import "cesium/Build/Cesium/Widgets/widgets.css";
import * as Cesium from 'cesium';

export default {
  name: "CesiumGlobeView",
  data() {
    return{
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
      dataUrl : [
        "https://vectortiles0.geo.admin.ch/3d-tiles/ch.swisstopo.swisstlm3d.3d/20190313/tileset.json",
        "https://vectortiles0.geo.admin.ch/3d-tiles/ch.swisstopo.vegetation.3d/20190313/tileset.json"
      ],
    }
  },
  methods: {
    // charge les données externes
    getExternalData(){
      for(var i = 0; i < this.dataUrl.length; i++) {
        var newLayer = new Cesium.Cesium3DTileset({
          url: this.dataUrl[i]
        });
        this.viewer.scene.primitives.add(newLayer);
      }
    },

    // modification de la vue depuis le sol
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
            heading: Cesium.Math.toRadians(0.0),
            pitch: Cesium.Math.toRadians(10.0),
            roll: 0.0,
          },
        });
      });
    },

    // affiche les éoliennes
    displayEolienne(){
      var viewer = this.viewer
      var uri
      Cesium.IonResource.fromAssetId(752612) // fichier GLB publié sur cesium ion
      .then(function (resource) {
        uri = resource
      });
      for (var i = 0; i < this.locations.length; i++) {
        let position = Cesium.Cartographic.fromDegrees(this.locations[i][2],this.locations[i][1])
        let location = Cesium.Cartesian3.fromDegrees(this.locations[i][2],this.locations[i][1])
        Cesium.sampleTerrainMostDetailed(this.viewer.terrainProvider, [position])
        .then(function(samples) {
          let groundHeight = samples[0].height;
          // ajoute les éoliennes
          viewer.entities.add({
            position: Cesium.Cartesian3.fromRadians(position.longitude,position.latitude, groundHeight),
            model : {
              uri: uri,
              scale : 15,
              color : Cesium.Color.WHITE,
            },
          });
          // ajoute les cercles au sol
          viewer.entities.add({
            position: location,
            ellipse: {
              semiMinorAxis: 90,
              semiMajorAxis: 90,
              material: Cesium.Color.AQUA,
            },
          });
        });
      };
    },

    // init Cesium globe
    setupCesiumGlobe () {
      // etendue de la Suisse, pour les données de base
      let rectangle = Cesium.Rectangle.fromDegrees(
        5.013926957923385,
        45.35600133779394,
        11.477436312994008,
        48.27502358353741
      );
      let viewer = new Cesium.Viewer('cesium-container', {
        baseLayerPicker: false,
        terrainProvider: new Cesium.CesiumTerrainProvider({
          // mnt de la suisse comme terrain
          url: "//3d.geo.admin.ch/1.0.0/ch.swisstopo.terrain.3d/default/20160115/4326/"
        }),
        imageryProvider: new Cesium.UrlTemplateImageryProvider({
          // swissimage pour mapper le mnt
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
    var extent = Cesium.Rectangle.fromDegrees(6.80980,46.59838,7.26709,46.86392);
    Cesium.Camera.DEFAULT_VIEW_RECTANGLE = extent;
    Cesium.Camera.DEFAULT_VIEW_FACTOR = 0;

    // init viewer
    this.viewer = this.setupCesiumGlobe();

    // désactive le test de profondeur de l'affichage (transparence du MNT)
    this.viewer.scene.globe.depthTestAgainstTerrain = true;
    
    // ajoute les données externes
    this.getExternalData()

    // ajoute les éoliennes
    this.displayEolienne();
  },
};
</script>

<style scoped>
#cesium-container {
  height: 600px;
}
</style>