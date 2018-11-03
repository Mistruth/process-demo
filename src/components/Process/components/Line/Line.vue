<template>
  <line
    class="line"
    :x1="data.position.startX"
    :y1="data.position.startY"
    :x2="data.position.endX"
    :y2="data.position.endY"
    style="stroke: #000; stroke-width: 2px; fill: none;marker-end: url(#arrow);" />
</template>

<script>
export default {
  props: {
    data: {
      type: Object,
      default: () => ({
        to: {
          name: '节点2',
          left: 100,
          top: 200,
          width: 100,
          height: 50
        },
        from: {
          name: '节点1',
          left: 200,
          top: 300,
          width: 100,
          height: 50
        },
        position: {
          startX: 150,
          startY: 250,
          endX: 250,
          endY: 300
        }
      })
    }
  }
}
</script>

<style lang="less">
</style>
