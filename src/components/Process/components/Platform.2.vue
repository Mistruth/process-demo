<template>
  <div class="v-process-platform-container">
    <div class="v-process-platform">
      <Node
      v-for="(item,index) in nodeList"
      :key="index"
      :name="item.name"
      v-drag.cursor="drag"
      :data="item"
      v-link="link"></Node>
      <svg class="v-process-svg"  xmlns="http://www.w3.org/2000/svg">
        <defs>
          <marker id="arrow" markerUnits="strokeWidth" markerWidth="12" markerHeight="12" viewBox="0 0 12 12" refX="6" refY="6" orient="auto">
            <path xmlns="http://www.w3.org/2000/svg" d="M2,2 L10,6 L2,10 L6,6 L2,2" class="v-process-arrow" />
          </marker>
        </defs>
        <line
          v-for="(item,index) in linkList" :key="'line'+index"
          :class="['v-process-line-default',{'v-process-linking':item.lineStyle.dash}]"
          :x1="item.position.startX"
          :y1="item.position.startY"
          :x2="item.position.endX"
          :y2="item.position.endY"
          />
      </svg>
    </div>
  </div>
</template>

<script>
import Node from './Node.vue'
export default {
  components: {
    Node
  },
  data () {
    return {
      drag: true,
      link: false
    }
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
          height: 50
        },
        {
          name: '节点2',
          left: 200,
          top: 300,
          width: 100,
          height: 50
        },
        {
          name: '节点3',
          left: 400,
          top: 100,
          width: 100,
          height: 50
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
            startX: 150,
            startY: 225,
            endX: 250,
            endY: 325
          },
          lineStyle: {
            dash: false
          }
        }
      ]
    }
  },
  directives: {
    drag: {
      bind (el, binding, vnode) {
        const index = vnode.data.key
        // const name = vnode.data.name
        el.style.position = 'absolute'
        var offsetX = 0
        var offsetY = 0

        function down (e) {
          if (!binding.value) {
            return
          }
          e.stopPropagation()
          offsetX = e.pageX - el.offsetLeft
          offsetY = e.pageY - el.offsetTop
          if (binding.modifiers.cursor) el.style.cursor = 'move'
          addEventListener('mousemove', move)
          addEventListener('mouseup', up)
        }
        function move (e) {
          // 获取nodeList
          const nodeList = el.__vue__.$parent.nodeList
          // const linkList = el.__vue__.$parent.linkList
          const maxLeft = el.offsetParent.offsetWidth - el.offsetWidth
          const maxTop = el.offsetParent.offsetHeight - el.offsetHeight
          const minLeft = 0
          const minTop = 0
          const x = e.pageX - offsetX
          const y = e.pageY - offsetY
          if (x < maxLeft && x > minLeft) {
            el.style.left = x + 'px'
          } else {
            el.style.left = x > maxLeft ? maxLeft + 'px' : minLeft + 'px'
          }
          if (y < maxTop && y > minTop) {
            el.style.top = y + 'px'
          } else {
            el.style.top = y > maxTop ? maxTop + 'px' : minTop + 'px'
          }
          // 实现data的双向绑定
          nodeList[index].left = el.offsetLeft
          nodeList[index].top = el.offsetTop
        }
        function up (e) {
          e.stopPropagation()
          removeEventListener('mousemove', move)
          removeEventListener('mouseup', up)
        }

        el.addEventListener('mousedown', down)
      }
    },
    link: {
      bind (el, binding, vnode) {
        if (!binding.value) {
          return
        }
        const index = vnode.data.key
        const nodeList = el.__vue__.$parent.nodeList
        const linkList = el.__vue__.$parent.linkList
        const data = nodeList[index]
        let cursorX, cursorY
        let linkItem = {}
        function down (e) {
          e.stopPropagation()
          let startX, startY, endX, endY
          startX = endX = data.left + data.width / 2
          startY = endY = data.top + data.height / 2
          linkItem = {
            from: {
              ...data
            },
            position: {
              startX,
              startY,
              endX,
              endY
            },
            lineStyle: {
              dash: true
            }
          }
          linkList.push(linkItem)
          addEventListener('mousemove', move)
          addEventListener('mouseup', up)
        }
        function move (e) {
          const container = document.querySelector('.v-process-platform-container')
          cursorX = e.pageX - container.offsetParent.offsetLeft + container.scrollLeft
          cursorY = e.pageY - container.offsetParent.offsetTop + container.scrollTop
          linkItem.position.endX = cursorX
          linkItem.position.endY = cursorY
        }
        function up (e) {
          const index = vnode.data.key
          const nodeList = el.__vue__.$parent.nodeList
          const linkList = el.__vue__.$parent.linkList
          const data = nodeList[index]
          let isEnd = false
          // 目标节点名称
          const toName = e.target.textContent
          // 拿到该节点所有的fromNode
          const nodeToBeLink = nodeList.find(node => node.name === toName)
          // 来自节点的列表
          const fromList = linkList
            .filter(link => {
              if (!link.to) {
                return false
              }
              return link.to.name === data.name
            })
            .map(item => item.from.name)
          // console.log(fromList)
          // 是否是来自的节点
          const isfrom = fromList.some(fromNode => {
            return toName === fromNode
          })
          // 如果是来自节点，就不挂载
          if (isfrom) {
            linkList.pop()
            e.stopPropagation()
            removeEventListener('mousemove', move)
            removeEventListener('mouseup', up)
            return
          }
          // 是否是其他节点
          isEnd = nodeList.some(node => {
            return (
              cursorX > node.left &&
              cursorX < node.left + node.width &&
              cursorY > node.top &&
              cursorY < node.top + node.height &&
              node.name !== data.name
            )
          })
          // 是其他节点
          if (isEnd) {
            // 设置 line 结束的地方 和 开始的地方
            linkItem.position.endX = nodeToBeLink.left + nodeToBeLink.width / 2
            // if (data.top < nodeToBeLink.top) {
            linkItem.position.endY = nodeToBeLink.top + nodeToBeLink.height / 2
            linkItem.position.startY = data.top + data.height / 2
            // } else {
            //   linkItem.position.endY = nodeToBeLink.top + nodeToBeLink.height
            //   linkItem.position.startY = data.top
            // }
            linkItem.to = { ...nodeToBeLink }
            linkItem.lineStyle = { ...linkItem.lineStyle, dash: false }
          } else {
            linkList.pop()
          }
          e.stopPropagation()
          removeEventListener('mousemove', move)
          removeEventListener('mouseup', up)
        }
        el.addEventListener('mousedown', down)
      }
    }
  },
  watch: {
    nodeList: {
      handler (val) {
        if (!this.drag) return
        val.forEach(node => {
          this.linkList.forEach(link => {
            // if (link.to.top <= link.from.top) {
            if (node.name === link.to.name) {
              link.to = { ...node }
              link.position.endX = node.left + node.width / 2
              link.position.endY = node.top + node.height / 2
            }
            if (node.name === link.from.name) {
              link.from = { ...node }
              link.position.startX = node.left + node.width / 2
              link.position.startY = node.top + node.height / 2
            }
            // } else {
            //   if (node.name === link.to.name) {
            //     link.to = { ...node }
            //     link.position.endX = node.left + node.width / 2
            //     link.position.endY = node.top
            //   }
            //   if (node.name === link.from.name) {
            //     link.from = { ...node }
            //     link.position.startX = node.left + node.width / 2
            //     link.position.startY = node.top + node.height
            //   }
            // }
          })
        })
      },
      // immediate:true,
      deep: true
    }
  }
}
</script>

<style lang="less">
.v-process-platform-container {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0px;
  left: 0px;
  overflow: auto;
  z-index:9999;
  .v-process-platform {
    position: absolute;
    top:0px;
    left:0px;
    width: 1500px;
    height: 2000px;
  }
}
.v-process-svg {
  position: absolute;
  height: 100%;
  width: 100%;
}
.v-process-line-default {
  stroke: #909399;
  stroke-width: 2px;
  fill: none;
  marker-end: url(#arrow);
  cursor: pointer;
}
.v-process-linking {
  stroke-dasharray:5,5;
  // stroke: #F56C6C;
}
.v-process-arrow {
  fill:#909399;
}
</style>
