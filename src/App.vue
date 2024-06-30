<template>
  <div id="app">
    <TextCounter />
  </div>
</template>

<script>
import TextCounter from './components/TextCounter.vue';

export default {
  name: 'App',
  components: {
    TextCounter
  }
};
</script>

<style>
html, body, #app {
  height: 100%;
  margin: 0;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #f0f8ff; /* 非常に薄い淡い青色 */
}

#app {
  text-align: center;
  color: #2c3e50;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
