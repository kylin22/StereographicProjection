<template>
  <div id="app">
    <RealProjectiveLine/>
  </div>
</template>

<script setup lang="ts">
  import RealProjectiveLine from "~/src/components/RealProjectiveLine.vue";
  useHead({
    title: "Stereographic Projection",
    meta: [
      { name: "description", content: "An interactive visualisation of stereographic projection and the Riemann sphere." }
    ],
  })
</script>

<style lang="scss">
  html, body, #__nuxt, #__layout {
    font-family: "Courier New", Courier, monospace;
    margin: 0px;
    min-height: 100% !important;
    height: 100%;
    user-select: none;
    caret-color: transparent;
  }
  
  #app {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
  }
</style>
