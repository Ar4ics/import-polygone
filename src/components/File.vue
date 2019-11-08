<template lang="html">
  <section class="file">
    <Map :data="data"/>
    <div class="text-reader">
      Загрузка файла с проекционными координатами
      <input @change="loadTextFromFile" type="file" />
    </div>
    <div class="text-reader">
      Загрузка xml-файла с WGS-84
      <input @change="loadTextFromFileXML" type="file" />
    </div>
  </section>
</template>

<script lang="js">
import Map from "./Map.vue"
import $ from "jquery";

export default  {
  name: 'File',
  components: {
    Map
  },
  props: [],
  data () {
    return {
        text: "",
        data: null
    }
  },
  
  computed: {

  },
  mounted () {

  },
  methods: {
    loadTextFromFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();

      reader.onload = e => {
        console.log(e);
        this.text = window.text = e.target.result;

        const formatted = this.text.split(/->[0-9]+/).map(
          e => e.trim().split("\n").map(
            e => e.split(" ").slice(0, 2).map(
              e => parseFloat(e)
              )
            )
          );
        formatted.splice(0, 1);
        const data = {coord: formatted, epsg: 'EPSG:3857'};
        this.data = data;
      }
      reader.readAsText(file);
    },

    loadTextFromFileXML(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();

      reader.onload = e => {
        console.log(e);
        this.text = window.text = e.target.result;

        const xml = $($.parseXML(this.text));
        window.xml = xml;
        let points = [];
        xml.find('Point[coordinateReferenceSystemId="GCS_WGS_1984"]').each((_, item) => {
          const point = $(item).attr('location').split(" ").map(e => parseFloat(e));
          points.push(point);
          //points.push([point[1], point[0]]);
        });
        points.push(points[0]);
        console.log(points);
        const data = {coord: [points], epsg: 'EPSG:4326'};
        this.data = data;
      }
      reader.readAsText(file);
    }
  }
}
</script>

<style scoped lang="scss">
#file {
  height: 100%;
  width: 100%;
}
</style>
