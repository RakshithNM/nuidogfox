<template>
  <DogFox msg="SPEECH RECOGNITION DOG AND FOX" />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import DogFox from './components/DogFox.vue'

export default defineComponent({
  name: 'App',
  components: {
    DogFox
  }
})
</script>

<style>
body {
  margin: 0;
  background: #011627;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: orangered;
  margin-top: 60px;
}
</style>
