<template>
  <div :data-id="data.name" class="v-process-node">
    <div class="v-process-node-content" :data-id="data.name">
      {{data.alias}}
      <transition name="bounce">
        <div @click="del(data)" v-if="data.style.delBtn" class="v-process-node-del">×</div>
      </transition>
    </div>
  </div>
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
    del (data) {
      this.$confirm(`是否删除${data.alias}`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$emit('del', data)
        })
        .catch(() => {
          this.$message({
            message: '取消删除',
            type: 'info'
          })
        })
    }
  }
}
</script>

<style lang="less">
.v-process-node {
  position: absolute;
  width: 100px;
  height: 50px;
  color: #888;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  z-index: 1000;
  font-size: 13px;
  .v-process-node-content {
    border: 1px solid #888;
    border-radius: 3px;
    position: absolute;
    height: 30px;
    width: 80px;
    line-height: 30px;
    text-align: center;
    top: 50%;
    transform: translateY(-50%) translateX(-50%);
    left: 50%;
  }
  .v-process-node-del {
    position: absolute;
    right: -6px;
    top: -6px;
    height: 15px;
    line-height: 13px;
    width: 15px;
    border-radius: 50%;
    background-color: #f56c6c;
    cursor: pointer;
    color: #fff;
    font-size: 12px;
  }
  .bounce-enter-active {
    animation: bounce-in 0.3s;
  }
  .bounce-leave-active {
    animation: bounce-in 0.3s reverse;
  }
  @keyframes bounce-in {
    0% {
      transform: scale(0);
    }
    50% {
      transform: scale(1.2);
    }
    100% {
      transform: scale(1);
    }
  }
}
</style>
