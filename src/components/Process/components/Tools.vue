<template>
  <div class="v-process-tools" :style="{width:width}">

  </div>
</template>

<script>
export default {
  // 宽度
  props: {
    width: {
      type: String,
      default: '150px'
    }
  }
}
</script>

<style lang="less">
.v-process-tools {
  position: absolute;
  height: 100%;
  background-color: skyblue;
  z-index:998;
  top:0px;
  left:-150px;
}
</style>
