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

.no-speechrecognition {
  height: 100vh;
  -webkit-animation: blink 1s infinite;  /* Safari 4+ */
  -moz-animation: blink 1s infinite;  /* Fx 5+ */
  -o-animation: blink 1s infinite;  /* Opera 12+ */
  animation: blink 1s infinite;  /* IE 10+, Fx 29+ */
}

@keyframes blink {
  0%, 49% {
    background-color: #011627;
  }
  50%, 100% {
    background-color: #e50000;
  }
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
