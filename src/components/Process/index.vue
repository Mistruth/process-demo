<template>
  <div class="v-process-container">
    <Tools
      @onDrag="setDrag"
      @onLink="setLink"
    ></Tools>
    <Platform
      :key="1"
      v-if="drag"
      :drag="drag"
      :link="link"
      :node-list="nodeList"
      :link-list="linkList"
    ></Platform>
    <Platform
      v-else
      :key="2"
      :drag="drag"
      :link="link"
      :node-list="nodeList"
      :link-list="linkList"
    ></Platform>
  </div>
</template>

<script>
import Tools from './components/Tools'
import Platform from './components/Platform.vue'
export default {
  components: {
    Tools,
    Platform
  },
  props: {
    nodeList: {
      type: Array,
      default: () => [
        {
          name: '节点1',
          left: 100,
          top: 200,
          width: 100,
          height: 50,
          flag: true,
          alias: '节点1',
          index: 1
        },
        {
          name: '节点2',
          left: 200,
          top: 300,
          width: 100,
          height: 50,
          alias: '节点2',
          index: 2
        },
        {
          name: '节点3',
          left: 400,
          top: 100,
          width: 100,
          height: 50,
          alias: '节点3',
          index: 3
        },
        {
          name: '节点4',
          left: 600,
          top: 150,
          width: 100,
          height: 50,
          alias: '节点4',
          index: 4
        }
      ]
    },
    linkList: {
      type: Array,
      default: () => [
        {
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
            startX: 175,
            startY: 250,
            endX: 225,
            endY: 300
          },
          lineStyle: {
            dash: false
          }
        }
      ]
    }
  },
  data () {
    return {
      drag: true,
      link: false
    }
  },
  methods: {
    setDrag (drag) {
      this.drag = drag
      this.link = !drag
    },
    setLink (link) {
      this.link = link
      this.drag = !link
    }
  }
}
</script>

<style lang="less">
.v-process-container {
  position: relative;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
}
</style>
