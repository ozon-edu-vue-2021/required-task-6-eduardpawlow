<template>
  <div id="app">
    <BaseTable />
  </div>
</template>

<script>
import BaseTable from '@/components/BaseTable'
export default {
  name: 'App',
  components: {
    BaseTable,
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
