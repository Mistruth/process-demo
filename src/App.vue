<template>
  <div id="app">
    <div class="container">
      <Process></Process>
    </div>
  </div>
</template>

<script>
import Process from '@/components/Process/index.vue'
export default {
  components: {
    Process
  }
}
</script>

<style lang="less">
* {
  padding:0;
  margin:0;
  list-style: none;
}
html,body,#app {
  height: 100%;
  overflow: hidden;
}
.container {
  width: 1300px;
  height: 80%;
  margin: 50px auto;
}
</style>
