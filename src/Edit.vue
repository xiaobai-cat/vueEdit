<template>
  <div class="modal">
    <div class="components-lists">
      <h4>组件列表</h4>
      <ShowComponentBox
        v-for="item in componentLists"
        :key="item.component"
        :title="item.title"
        @dragstart="dragstart($event, item.component)"
        @drag="drag"
        ><component :is="item.component"></component
      ></ShowComponentBox>
    </div>
    <div class="edit">
      <ul class="edit-options">
        <li v-for="item in editOptions" :key="item.type" @click="editOptionsMethod(item.type)">{{ item.name }}</li>
      </ul>
      <div
        class="edit-body"
        @click.prevent="componentBlockFocus('edit-body')"
        @dragover.prevent="dragover"
        @drop.stop="drop"
        :class="[isComponentBlockFocusKey === 'edit-body' ? 'componentBlockFocus' : '']"
      >
        <div
          :class="['componentBlock', isComponentBlockFocusKey === item.key ? 'componentBlockFocus' : '']"
          v-for="item in componentsIsEditLists"
          :key="item.key"
          :style="{ top: item.position[0].top + 'px', left: item.position[0].left + 'px' }"
          @click.stop="componentBlockFocus(item)"
          @dragstart.stop="dragstart($event, item)"
          draggable="true"
        >
          <component :is="item.name"></component>
        </div>
      </div>
    </div>
    <div class="component-properties-set">3</div>
  </div>
</template>
<script>
  import ZInput from './components/ZInput.vue';
  import Tag from './components/Tag.vue';
  import ShowComponentBox from './components/ShowComponentBox.vue';
  import { random } from './common/utils.js';
  export default {
    name: 'Edit',
    components: { ZInput, ShowComponentBox, Tag },
    data() {
      return {
        copyComponent: null, //被拖动的组件
        isComponentBlockFocusKey: null, //组件是否被点击
        isFocusComponent: null, //被选中的组件
        componentLists: [
          { title: '输入框', component: 'ZInput' },
          { title: '标签', component: 'Tag' },
        ],
        componentsIsEditLists: [], //拖动到中间的组件列表
        startX: 0,
        startY: 0,
        editOptions: [
          { name: '撤销', type: 'revoke' },
          { name: '重做', type: 'redo' },
          { name: '预览', type: 'preview' },
          { name: '导入', type: 'import' },
          { name: '导出', type: 'export' },
          { name: '置顶', type: 'top' },
          { name: '置底', type: 'bottom' },
          { name: '删除', type: 'delete' },
          { name: '清空', type: 'clear' },
          { name: '关闭', type: 'close' },
        ],
        deleteComponets: {}, //删除的组件
        actionPerformed: [], //已经执行的操作
      };
    },
    methods: {
      dragstart(e, component) {
        this.copyComponent = component;
        //如果把组件看做一个盒子，记录鼠标首次在组件盒子内的位置
        this.startX = e.offsetX;
        this.startY = e.offsetY;
      },
      drag() {},
      dragend() {},
      dragover() {},
      drop(e) {
        let copyComponent = this.copyComponent;
        let optionType = 'add';
        let key = null;
        //如果拖动的是已经新增的组件，只改变top、left值
        if (copyComponent.isAdd) {
          optionType = 'move';
          this.componentsIsEditLists = this.componentsIsEditLists.map((item) => {
            if (item.key === copyComponent.key) {
              //编辑区域看做编辑盒子，使用鼠标在编辑盒子内的位置减去在组件盒子内的位置即使组件的实际位置。
              item.position.unshift({
                top: e.offsetY - this.startY,
                left: e.offsetX - this.startX,
              });
              this.isComponentBlockFocusKey = item.key;
              key = item.key;
            }
            return item;
          });
        } else {
          key = this.copyComponent + random(); //循环渲染组件的key
          let component = {
            name: this.copyComponent, //组件名称
            key: key,
            position: [
              {
                top: e.offsetY,
                left: e.offsetX,
              },
            ],
            isAdd: true, //新增的组件
          };
          this.componentsIsEditLists.push(component);
        }
        this.actionPerformed.unshift({ optionType: optionType, componentKey: key });
      },
      //当组件被点击添加标识
      componentBlockFocus(item) {
        this.isComponentBlockFocusKey = item.key;
        this.isFocusComponent = item;
      },
      editOptionsMethod(optionType) {
        switch (optionType) {
          case 'revoke': //撤销
            this.revoke();
            break;
          case 'redo': //重做
            break;
          case 'preview': //预览
            break;
          case 'import': //导入
            break;
          case 'export': //导出
            break;
          case 'top': //置顶
            break;
          case 'bottom': //置底
            break;
          case 'delete': //删除
            this.delete();
            break;
          case 'clear': //清空
            this.clear();
            break;
          default: //关闭
        }
      },
      //撤销
      revoke() {
        if (this.actionPerformed.length === 0) return;
        let componentsIsEditLists = this.componentsIsEditLists;
        let actionPerformed = this.actionPerformed[0];
        let optionType = actionPerformed.optionType;
        switch (optionType) {
          case 'move':
            for (let i in componentsIsEditLists) {
              let key = componentsIsEditLists[i].key;
              if (key === actionPerformed.componentKey) {
                let position = componentsIsEditLists[i].position;
                if (position.length !== 1) {
                  position.shift();
                  break;
                }
              }
            }
            break;
          case 'add':
            for (let i in componentsIsEditLists) {
              let key = componentsIsEditLists[i].key;
              if (key === actionPerformed.componentKey) {
                this.componentsIsEditLists.splice(i, 1);
                break;
              }
            }
            break;
          case 'clear':
            this.componentsIsEditLists = actionPerformed.component;
            break;
          default:
            //默认回退操作是删除
            componentsIsEditLists.push(actionPerformed.component);
        }

        this.actionPerformed.shift();
      },
      //删除
      delete() {
        let componentsIsEditLists = this.componentsIsEditLists;
        this.actionPerformed.unshift({ optionType: 'delete', component: this.isFocusComponent });
        for (let i in componentsIsEditLists) {
          if (componentsIsEditLists[i].key === this.isComponentBlockFocusKey) {
            componentsIsEditLists.splice(i, 1);
            this.copyComponent = componentsIsEditLists[componentsIsEditLists.length - 1];
            break;
          }
        }

        this.componentsIsEditLists = componentsIsEditLists;
      },
      //清空
      clear() {
        this.actionPerformed.unshift({ optionType: 'clear', component: this.componentsIsEditLists });
        this.componentsIsEditLists = [];
      },
    },
  };
</script>
<style lang="scss" scoped>
  $menuW: 300px;
  $commonPad: 15px;
  .modal {
    position: relative;
    height: 100%;
    padding-right: $menuW;
    padding-left: $menuW;
    background-color: #fff;
    box-shadow: 0px 0px 6px 0 #a9a9a9;
    border-radius: 4px;
  }

  .components-lists,
  .component-properties-set {
    position: absolute;
    top: 0;
    width: $menuW;
    height: 100%;
    padding: $commonPad;
    border-width: 1px;
    border-color: #ccc;
    overflow-y: auto;
  }
  .components-lists {
    left: 0;
    border-right-style: solid;
  }
  .component-properties-set {
    right: 0;
    border-left-style: solid;
  }
  .edit {
    height: 100%;
    background-color: antiquewhite;
    border: 1px solid rgba(0, 0, 0, 0);
    .edit-body {
      position: relative;
      width: 1000px;
      height: 700px;
      margin-left: auto;
      margin-right: auto;
      background-color: #fff;
    }
  }
  .edit-body,
  .componentBlock {
    border: 1px dashed rgba(0, 0, 0, 0);
  }
  .componentBlock {
    position: absolute;
    cursor: move;
    z-index: 10;
    padding: 3px;

    > * {
      pointer-events: none;
    }
  }
  .componentBlockFocus {
    border-color: #409eff;
  }
  .edit-options {
    width: 500px;
    margin: 20px auto;
    display: flex;
    justify-content: space-between;
    li {
      height: 50px;
      background-color: rgba(0, 0, 0, 0.5);
      color: #fff;
      text-align: center;
      line-height: 50px;
      flex-grow: 1;
      cursor: pointer;
      &:not(:last-child) {
        border-right: 1px solid #fff;
      }
      &:hover {
        background-color: #fff;
        color: #409eff;
      }
    }
  }
</style>
