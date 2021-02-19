<template>
  <div class="box" draggable="true" @dragstart="dragstart" @drag="drag" @dragend="dragend">
    <span v-if="title">{{ title }}</span>
    <slot></slot>
  </div>
</template>
<script>
  export default {
    name: 'ShowComponentBox',
    props: ['title', 'componet'],
    methods: {
      dragstart() {
        this.$emit('dragstart', event);
      },
      drag() {
        this.$emit('drag', event);
      },
      dragend() {
        this.$emit('dragend', event);
      },
    },
  };
</script>
<style lang="scss" scoped>
  .box {
    position: relative;
    width: 100%;
    padding: 50px 10px 20px;
    margin-bottom: 15px;
    border: 2px solid #ebeef5;
    cursor: move;
    > * {
      pointer-events: none;
    }

    span {
      position: absolute;
      top: 0;
      left: 0;
      height: 30px;
      padding-right: 5px;
      padding-left: 5px;
      color: #fff;
      line-height: 30px;
      background-color: #409eff;
    }
  }
</style>
