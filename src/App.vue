<template>
  <div id="app">
    <GCodeLooper />
  </div>
</template>

<script>
import GCodeLooper from './components/GCodeLooper.vue';

export default {
  name: 'App',
  components: {
    GCodeLooper
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
