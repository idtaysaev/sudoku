<template>
  <div id="app">
    <Sudoku />
  </div>
</template>

<script>
import Sudoku from './components/Sudoku.vue'

export default {
  name: 'App',
  components: {
    Sudoku
  }
}
</script>

<style lang="scss" scoped>
#app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
