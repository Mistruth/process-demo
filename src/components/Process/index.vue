<template>
  <div class="v-process-container">
    <Tools
      @onDrag="setDrag"
      @onLink="setLink"
    ></Tools>
    <Platform
      :key="1"
      v-if="componentSwitch"
      :drag="drag"
      :link="link"
      :node-list="nodeList"
      :link-list="linkList"
      @delNode="delNode"
      @dblclickLine="dblclickLine"
    ></Platform>
    <Platform
      :key="2"
      v-if="!componentSwitch"
      :drag="drag"
      :link="link"
      :node-list="nodeList"
      :link-list="linkList"
      @delNode="delNode"
      @dblclickLine="dblclickLine"
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
          lastIndex: 1,
          style: {
            delBtn: false
          }
        },
        {
          name: '节点2',
          left: 200,
          top: 300,
          width: 100,
          height: 50,
          alias: '节点2',
          lastIndex: 2,
          style: {
            delBtn: false
          }
        },
        {
          name: '节点3',
          left: 400,
          top: 100,
          width: 100,
          height: 50,
          alias: '节点3',
          lastIndex: 3,
          style: {
            delBtn: false
          }
        },
        {
          name: '节点4',
          left: 600,
          top: 150,
          width: 100,
          height: 50,
          alias: '节点4',
          lastIndex: 4,
          style: {
            delBtn: false
          }
        }
      ]
    },
    linkList: {
      type: Array,
      default: () => [
        {
          name: 'link1',
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
      link: false,
      componentSwitch: true
    }
  },
  methods: {
    setDrag (drag) {
      this.drag = true
      this.link = false
      this.componentSwitch = !this.componentSwitch
    },
    setLink (link) {
      this.link = true
      this.drag = false
      this.nodeList.forEach(item => {
        item.style.delBtn = false
      })
      this.componentSwitch = !this.componentSwitch
    },
    delNode (data) {
      const indexNode = this.nodeList.findIndex(item => item.name === data.name)
      let arr = []
      this.linkList.forEach((item, index) => {
        if (item.from.name === data.name || item.to.name === data.name) {
          arr.push(index)
        }
      })
      this.linkList = this.deleteIndex(this.linkList, arr)
      this.nodeList.splice(indexNode, 1)
      this.componentSwitch = !this.componentSwitch
    },
    deleteIndex (array, indexArray) {
      var indexLength = indexArray.length
      indexArray.sort(function (a, b) {
        if (a - b > 0) {
          return 1
        } else if (a - b < 0) {
          return -1
        } else {
          return 0
        }
      })
      var deltimes = 0
      for (var i = 0; i < indexLength; i++) {
        array.splice((indexArray[i] - deltimes), 1)
        deltimes++
      }
      return array
    },
    dblclickLine (data) {
      if (this.link) {
        return
      }
      const index = this.linkList.findIndex(item => item.name === data.name)
      this.linkList.splice(index, 1)
      this.componentSwitch = !this.componentSwitch
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
