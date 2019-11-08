<template lang="html">

  <section>
    <div id="map" class="map"></div>
    <div id="mouse-position"></div>
    <form>
      <label>Projection </label>
      <select id="projection">
        <option value="EPSG:3857">EPSG:3857</option>
        <option value="EPSG:4326">EPSG:4326</option>
      </select>
      <label>Precision </label>
      <input id="precision" type="number" min="0" max="12" value="4"/>
    </form>
  </section> 

</template>

<script lang="js">
import 'ol/ol.css';
import Map from 'ol/Map';
import View from 'ol/View';
import TileLayer from 'ol/layer/Tile';
import {OSM} from 'ol/source';
import Feature from 'ol/Feature';
import Polygon from 'ol/geom/Polygon';
// import MultiPolygon from 'ol/geom/MultiPolygon';
import VectorSource from 'ol/source/Vector';
import VectorLayer from 'ol/layer/Vector';
import {Stroke, Style} from 'ol/style';
import MousePosition from 'ol/control/MousePosition';
import {createStringXY} from 'ol/coordinate';
import WFS from "ol/format/WFS";
import GML from "ol/format/GML";
import TileWMS from "ol/source/TileWMS";

import $ from "jquery";
export default  {
  name: 'Map',
  props: {
    data: {
      type: Object,
      default: null
    },
  },
  data () {
    return {
      map: null,
      view: null,
      endpoint: "http://localhost:8080/geoserver/gis_example",
      featureNS: "gis_example",
      featureType: "wfs_polygon",
      projection: "EPSG:3857",
      projectionFrom: "EPSG:4326"
    }
  },
  computed: {
    layers: function () {
      return this.map.getLayers();
    }
  },
  watch: { 
    data: function(newVal, oldVal) {
      console.log('Prop changed: ', newVal, ' | was: ', oldVal)
      if (!newVal) return;

      // let polygonList = [];
      // for (var i = 0; i < newVal.length; i += 1) {
      //     var polygon = new Polygon([newVal[i]]);
      //     polygonList.push(polygon);
      // }
      // var multiPolygon = new MultiPolygon(polygonList);
      // var feature = new Feature(multiPolygon);

      var polygon = new Polygon(newVal['coord']);
      var feature = new Feature(polygon);
      if (newVal['epsg'] === this.projectionFrom) {
        feature.getGeometry().transform(this.projectionFrom, this.projection);
      }

      var vectorSource = new VectorSource();
      vectorSource.addFeature(feature);

      var vectorLayer = new VectorLayer({
        source: vectorSource,
        style: new Style({
          stroke: new Stroke({
            color: 'rgba(0, 0, 255, 1.0)',
            width: 2
          })
        })
      });

      this.map.addLayer(vectorLayer);
      const coord = feature.getGeometry().getCoordinates();
      this.view.animate({
        center: coord[0][0],
        duration: 2000,
        zoom: 8
      });

      this.insertPolygonToGeoserver(vectorLayer.getSource().getFeatures());

    }
  },
  mounted () {
    this.view = new View({
      center: [0, 0],
      zoom: 0
    });
    this.map = window.map = new Map({
      layers: [
        new TileLayer({
          source: new OSM()
        }),

        new TileLayer({
          source: new TileWMS({
            url: this.endpoint + "/wms",
            params: { LAYERS: this.featureNS + ":" + this.featureType, TILED: true },
            serverType: "geoserver",
            projection: this.projection,
            transition: 0
          })
        })
      ],
      target: 'map',
      view: this.view
    });

    var mousePositionControl = new MousePosition({
      coordinateFormat: createStringXY(4),
      projection: this.projection,
      // comment the following two lines to have the mouse position
      // be placed within the map.
      className: 'custom-mouse-position',
      target: document.getElementById('mouse-position'),
      undefinedHTML: '&nbsp;'
    });

    this.map.addControl(mousePositionControl);

    var projectionSelect = document.getElementById('projection');
    projectionSelect.addEventListener('change', function(event) {
      mousePositionControl.setProjection(event.target.value);
    });

    var precisionInput = document.getElementById('precision');
    precisionInput.addEventListener('change', function(event) {
      var format = createStringXY(event.target.valueAsNumber);
      mousePositionControl.setCoordinateFormat(format);
    });

  },
  methods: {
    insertPolygonToGeoserver(features) {
      console.log(features);
      var wfst = new WFS();
      var gml = new GML({
        featureNS: this.featureNS,
        featureType: this.featureType,
        srsName: this.projection
      });
      var node = wfst.writeTransaction(features, null, null, gml);
      var str = new XMLSerializer().serializeToString(node);
      console.log(str);

      $.ajax({
        url: this.endpoint + "/wfs",
        data: str,
        type: "POST",
        dataType: "xml",
        contentType: "text/xml",
        success: function(data) {
          var result = wfst.readTransactionResponse(data);
          console.log(result);
        },
        error: function(e) {
          var errorMsg = e ? e.status + " " + e.statusText : "";
          console.log(
            "Error saving this feature to GeoServer.<br><br>" + errorMsg
          );
        },
        context: this
      });
    } 
  }
}


</script>

<style scoped lang="scss">
@import "~ol/ol.css";
.map {
  height: 400px;
  width: 100%;
}

section {
  height: 500px;
}
</style>
