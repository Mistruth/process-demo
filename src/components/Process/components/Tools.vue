<template>
  <div class="v-process-tools" :style="{width:width}">
    <el-tabs v-model="activeName">
      <el-tab-pane label="添加节点" name="first">
        <div class="nodeTypes-container">
          <div @click="setDrag" :class="['v-process-node-template','node-type-1',{'press-btn-bgc':select}]" data-type="1">选择器</div>
          <div @click="setLink" :class="['v-process-node-template','node-type-2',{'press-btn-bgc':link}]" data-type="2">连线</div>
          <div v-drag class="v-process-node-template node-type-3" data-type="3">开始节点</div>
          <div v-drag class="v-process-node-template node-type-4" data-type="4">结束节点</div>
          <div v-drag class="v-process-node-template node-type-5" data-type="5">判断节点</div>
          <div v-drag class="v-process-node-template node-type-6" data-type="6">任务节点</div>
          <div v-drag class="v-process-node-template node-type-7" data-type="7">Join节点</div>
          <div v-drag class="v-process-node-template node-type-8" data-type="8">Fork节点</div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="节点设置" name="second">配置管理</el-tab-pane>
      <el-tab-pane label="流程设置" name="third">角色管理</el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  props: {
    width: {
      type: String,
      default: '270px'
    }
  },
  data () {
    return {
      activeName: 'first',
      select: true,
      link: false
    }
  },
  directives: {
    drag: {
      bind (el, binding, vnode) {
        let oldX, oldY
        var offsetX = 0
        var offsetY = 0
        function down (e) {
          e.stopPropagation()
          oldX = el.offsetLeft
          oldY = el.offsetTop
          offsetX = e.pageX - el.offsetLeft
          offsetY = e.pageY - el.offsetTop
          if (binding.modifiers.cursor) el.style.cursor = 'move'
          el.style.backgroundColor = 'rgb(55, 145, 235)'
          el.style.zIndex = 10001
          addEventListener('mousemove', move)
          addEventListener('mouseup', up)
        }
        function move (e) {
          const x = e.pageX - offsetX
          const y = e.pageY - offsetY
          el.style.left = x + 'px'
          el.style.top = y + 'px'
        }
        function up (e) {
          let newX,
            newY,
            Dx,
            Dy,
            containerWidth,
            selfWidth,
            selfHeight,
            containerHeight,
            container,
            toolWidth,
            order
          order = el.dataset.type
          newX = el.offsetLeft
          newY = el.offsetTop
          selfWidth = el.offsetWidth
          selfHeight = el.offsetHeight
          Dx = order % 2 === 0 ? newX - oldX + selfWidth + 15 : newX - oldX
          Dy = newY - oldY
          container = document.querySelector('.v-process-platform-container')
          toolWidth = document.querySelector('.v-process-tools').offsetWidth
          containerWidth = container.offsetWidth
          containerHeight = container.offsetHeight
          const minDx = toolWidth
          const maxDx = toolWidth + containerWidth - selfWidth
          const minDy = -oldY
          const maxDy = containerHeight + minDy - selfHeight
          if (Dx > minDx && Dx < maxDx && Dy > minDy && Dy < maxDy) {
            const nodeList = vnode.context.$parent.nodeList
            const length = nodeList.length
            const index = nodeList[length - 1].index + 1
            const newNode = {
              name: 'node' + Math.random(),
              alias: `节点${index}`,
              left: newX - toolWidth,
              top: newY,
              width: 100,
              height: 50,
              index
            }
            nodeList.push(newNode)
          }
          e.stopPropagation()
          el.style.left = oldX + 'px'
          el.style.top = oldY + 'px'
          el.style.backgroundColor = '#409EFF'
          el.style.zIndex = 10000
          removeEventListener('mousemove', move)
          removeEventListener('mouseup', up)
        }
        el.addEventListener('mousedown', down)
      }
    }
  },
  methods: {
    setDrag () {
      this.select = true
      this.link = false
      this.$emit('onDrag', this.select)
    },
    setLink () {
      this.select = false
      this.link = true
      this.$emit('onLink', this.link)
    }
  }
}
</script>

<style lang="less">
.v-process-tools {
  height: 100%;
  float: left;
  padding: 5px 10px;
  border: 1px solid #ccc;
  border-radius: 3px;
  box-sizing: border-box;
  .el-tabs__content {
    overflow: visible;
    position: static;
  }
}
.press-btn-bgc {
  background-color: rgb(3, 121, 240)!important;
}
.nodeTypes-container {
  // position: relative;
  padding: 0px 10px;
  height: 600px;
  .v-process-node-template {
    position: absolute;
    color: #fff;
    font-size: 12px;
    text-align: center;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    height: 32px;
    line-height: 32px;
    background-color: #409eff;
    width: 100px;
    cursor: pointer;
    border-radius: 3px;
    z-index: 10000;
    border: 1px solid transparent;
    &:hover {
      border-color: #eee;
    }
  }
  .node-type-1 {
    left: 20px;
    top: 60px;
  }
  .node-type-2 {
    left: 145px;
    top: 60px;
  }
  .node-type-3 {
    left: 20px;
    top: 100px;
  }
  .node-type-4 {
    left: 145px;
    top: 100px;
  }
  .node-type-5 {
    left: 20px;
    top: 140px;
  }
  .node-type-6 {
    left: 145px;
    top: 140px;
  }
  .node-type-7 {
    left: 20px;
    top: 180px;
  }
  .node-type-8 {
    left: 145px;
    top: 180px;
  }
}
</style>
