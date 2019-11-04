<template lang="html">
  <section class="file">
    <Map :coordinates="formatted"/>
    <label class="text-reader">
      <input @change="loadTextFromFile" type="file" />
    </label>
  </section>
</template>

<script lang="js">
import Map from "./Map.vue"
export default  {
  name: 'File',
  components: {
    Map
  },
  props: [],
  data () {
    return {
        text: "",
        formatted: []
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
        this.formatted = formatted;
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
