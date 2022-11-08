<template>
    <RouletteView/>
</template>

<script lang="ts">
import {defineComponent} from "vue";
import RouletteView from "./views/RouletteView.vue"

export default defineComponent({
  name: "App",
  components: {
    RouletteView
  },
});
</script>

<style>
</style>
