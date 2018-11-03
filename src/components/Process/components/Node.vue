<template>
  <div  @click="nodeClick" :data-id="data.name" class="v-process-node">{{data.alias}}</div>
</template>

<script>
export default {
  props: {
    data: {
      type: Object,
      default: () => ({
        name: '节点',
        left: 0,
        top: 0
      })
    }
  },
  mounted () {
    this.setPostiton()
  },
  methods: {
    setPostiton (x = this.data.left, y = this.data.top) {
      this.$el.style.left = x + 'px'
      this.$el.style.top = y + 'px'
    },
    nodeClick () {
      // 点击 上浮
      this.$el.style.zIndex++
    }
  }
}
</script>

<style>
.v-process-node {
  width: 100px;
  height: 50px;
  border: 1px solid #ccc;
  line-height: 50px;
  text-align: center;
  /* background-color: teal; */
  opacity: 0.8;
  border-radius: 3px;
  color: #888;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  z-index:9999;
}
</style>
