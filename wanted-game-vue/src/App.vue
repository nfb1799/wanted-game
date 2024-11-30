<template>
  <div class="wrapper">x
    <WantedGame />
  </div>
</template>

<script>
import WantedGame from '@/components/game'

export default {
  name: 'App',
  components: {
    WantedGame
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: black;
  height: 100vh;
}
body {
  margin: 0;
}
.wrapper {
  display: flex;
  flex-direction: column;
  height: 100%;
}
</style>
