<template>
  <div class="v-process-platform-container">
    <div class="v-process-platform">
      <Node
      @dblclick.native="nodeDbclick(item)"
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
          @click="lineClick(item)"
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
    return {}
  },
  props: {
    nodeList: {
      type: Array,
      default: () => []
    },
    linkList: {
      type: Array,
      default: () => []
    },
    drag: {
      type: Boolean,
      default: true
    },
    link: {
      type: Boolean,
      default: false
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
        const getPosition = el.__vue__.$parent.getPosition
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
          const tabWidth = document.querySelector('.v-process-tools').offsetWidth
          cursorX = e.pageX - container.offsetParent.offsetLeft + container.scrollLeft - tabWidth
          cursorY = e.pageY - container.offsetParent.offsetTop + container.scrollTop
          // cursorX = e.pageX + container.scrollLeft
          // cursorY = e.pageY + container.scrollTop
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
          // const toName = e.target.textContent
          const toName = e.target.dataset.id
          // console.log(el.dataset.id)
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

          const toList = linkList.filter(link => {
            if (!link.to) {
              return false
            }
            return link.to.name === toName
          }).map(item => item.from.name)
          const isTo = toList.some(toNode => {
            return data.name === toNode
          })

          // 如果是来自节点，就不挂载
          if (isfrom || isTo) {
            linkList.pop()
            e.stopPropagation()
            removeEventListener('mousemove', move)
            removeEventListener('mouseup', up)
            return
          }
          // 判断是否有已经连过线，如果是就不挂

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
            linkItem.to = { ...nodeToBeLink }
            const position = getPosition(linkItem.to, linkItem.from)
            linkItem.position = { ...position }
            linkItem.lineStyle = { ...linkItem.lineStyle, dash: false }
            nodeList[0].flag = false
            // 再次预防 to 和 from 一样
            if (linkItem.to.name === linkItem.from.name) {
              linkList.pop()
            }
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
            let toNode = {}
            let fromNode = {}
            // if (link.to.top <= link.from.top) {
            if (node.name === link.to.name) {
              link.to = { ...node }
              toNode = node
              const obj = this.getPosition(toNode, link.from)
              link.position = { ...obj }
            }
            if (node.name === link.from.name) {
              link.from = { ...node }
              fromNode = node
              const obj = this.getPosition(link.to, fromNode)
              link.position = { ...obj }
            }
          })
        })
      },
      // immediate:true,
      deep: true
    }
  },
  methods: {
    getPosition (toNode, fromNode) {
      let startX, startY, endX, endY
      // 比例 2
      const tan = fromNode.width / fromNode.height
      // left 差
      const Dleft = fromNode.left - toNode.left
      // top 差
      const Dtop = fromNode.top - toNode.top

      const currentTan = Math.abs(Dleft / Dtop)
      // 用top计算位置 left相对中心点不动
      if (currentTan > tan) {
        const y = Math.abs(0.5 * toNode.width * Dtop / Dleft)
        if (Dtop >= 0 && Dleft >= 0) {
          startX = fromNode.left
          startY = fromNode.top + 0.5 * fromNode.height - y
          endX = toNode.left + toNode.width
          endY = toNode.top + 0.5 * toNode.height + y
        }
        if (Dtop > 0 && Dleft < 0) {
          startX = fromNode.left + fromNode.width
          startY = fromNode.top + 0.5 * fromNode.height - y
          endX = toNode.left
          endY = toNode.top + 0.5 * toNode.height + y
        }
        if (Dtop < 0 && Dleft > 0) {
          startX = fromNode.left
          startY = fromNode.top + 0.5 * fromNode.height + y
          endX = toNode.left + toNode.width
          endY = toNode.top + 0.5 * toNode.height - y
        }
        if (Dtop <= 0 && Dleft <= 0) {
          startX = fromNode.left + fromNode.width
          startY = fromNode.top + 0.5 * fromNode.height + y
          endX = toNode.left
          endY = toNode.top + 0.5 * toNode.height - y
        }
      } else {
        const x = Math.abs(0.5 * toNode.height * Dleft / Dtop)
        if (Dtop >= 0 && Dleft >= 0) {
          startX = fromNode.left + 0.5 * fromNode.width - x
          startY = fromNode.top
          endX = toNode.left + 0.5 * toNode.width + x
          endY = toNode.top + toNode.height
        }
        if (Dtop > 0 && Dleft < 0) {
          startX = fromNode.left + 0.5 * fromNode.width + x
          startY = startY = fromNode.top
          endX = toNode.left + 0.5 * toNode.width - x
          endY = toNode.top + toNode.height
        }
        if (Dtop < 0 && Dleft > 0) {
          startX = fromNode.left + 0.5 * fromNode.width - x
          startY = fromNode.top + fromNode.height
          endX = toNode.left + 0.5 * toNode.width + x
          endY = toNode.top
        }
        if (Dtop <= 0 && Dleft <= 0) {
          startX = fromNode.left + 0.5 * fromNode.width + x
          startY = fromNode.top + fromNode.height
          endX = toNode.left + 0.5 * toNode.width - x
          endY = toNode.top
        }
      }
      return {
        startX,
        startY,
        endX,
        endY
      }
    },
    nodeDbclick (item) {
      console.log(item)
    },
    lineClick (item) {
      console.log(item)
    }
  }
}
</script>

<style lang="less">
.v-process-platform-container {
  border:1px solid #ccc;
  border-left: none;
  position: relative;
  overflow: auto;
  // width: 100%;
  height: 100%;
  // top: 0px;
  // left: 0px;
  // z-index:9999;
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
