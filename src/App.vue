<template>
    <RouletteView/>
</template>

<script lang="ts">
import {defineComponent} from "vue";
import HelloWorld from "./components/HelloWorld.vue";
import ReloadPWA from "./components/ReloadPWA.vue";
import RouletteView from "./views/RouletteView.vue"

export default defineComponent({
  name: "App",
  components: {
    HelloWorld,
    ReloadPWA,
    RouletteView
  },
});
</script>

<style>
</style>
