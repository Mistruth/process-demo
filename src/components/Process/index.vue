<template>
  <div class="v-process-container">
    <Tools></Tools>
    <Platform></Platform>
  </div>
</template>

<script>
import Tools from './components/Tools'
import Platform from './components/Platform.vue'
export default {
  components: {
    Tools,
    Platform
  }
}
</script>

<style lang="less">
.v-process-container {
  position: relative;
  width: 100%;
  height: 100%;
}
</style>
