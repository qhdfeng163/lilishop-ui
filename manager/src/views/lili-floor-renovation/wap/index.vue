<template>
  <div class="wrapper">
    <!-- 拖拽栏 ，展示栏  -->
    <div class="model-view">
      <div class="model-view-menu">
        <draggable
          class="model-view-menu-item"
          :list="modelData"
          :move="handleMove"
          v-bind="{
            group: { name: 'model', pull: 'clone', put: false, animation: 150 },
            sort: false,
            ghostClass: 'ghost',
          }"
        >
          <li
            v-for="(model, index) in modelData"
            v-if="!model.drawer"
            :key="index"
            class="model-item"
          >
            <img src alt />
            <span>{{ model.name }}</span>
          </li>
        </draggable>
      </div>

      <div class="model-view-content">
        <div class="content">
          <div class="wap-title">首页</div>
          <draggable
            class="draggable"
            group="model"
            ghostClass="ghost"
            @add="handleContentlAdd"
            @end="handleContentlEnd"
            v-model="contentData.list"
          >
            <div
              class="list"
              v-for="(element, index) in contentData.list"
              :key="element.key"
            >
              <component
                class="component"
                :class="{ active: selected == index }"
                @click.native="handleComponent(element, index)"
                :is="templates[element.type]"
                :res="element.options"
              ></component>
              <Icon
                v-if="selected == index"
                @click="closeComponent(index)"
                color="#e1251b"
                size="25"
                class="close"
                type="ios-close-circle"
              />
            </div>
          </draggable>
        </div>
      </div>

      <!-- 右侧栏 -->
      <div class="model-config">
        <decorate
          @handleDrawer="handleDrawer"
          v-if="decorateData"
          :res="decorateData"
        ></decorate>
      </div>
    </div>
  </div>
</template>
<script>
import templates from "./template/index";
import Draggable from "vuedraggable";
import { modelData } from "./config";
import decorate from "./decorate";
import * as API_Other from '@/api/other'
export default {
  components: {
    Draggable,
    decorate,
  },
  data() {
    return {
      templates, // 模板类型
      modelData, // 装修模型
      qrcode: "", // 二维码
      selected: 0, // 已选下标
      contentData: { // 总数据
        list: [],
      },
      decorateData: "", // 装修数据
    };
  },
  watch: {
    contentData: {
      handler(val) {
        this.$store.state.styleStore = val;
      },
      deep: true,
    },
  },
  mounted() {
    this.init();
  },

  methods: {
    init() {
      if (!this.$route.query.id) return false;
      API_Other.getHomeData(this.$route.query.id).then(res=>{
        this.contentData = JSON.parse(res.result.pageData)


         this.handleComponent( this.contentData.list[0], 0)
      })
    },
    handleSpinShow() {
      this.$Spin.show();
      setTimeout(() => {
        this.$Spin.hide();
      }, 3000);
    },

    // 中间组件拖动，右侧数据绑定不变
    handleContentlEnd(evt) {
      const { newIndex } = evt;
      this.handleComponent(this.contentData.list[newIndex], newIndex);
    },

    // 关闭楼层装修
    closeComponent(index) {
      this.$nextTick(() => {
        this.decorateData = "";

        console.log(this.contentData.list.length);
        // 如果当前楼层不为一
        if (this.contentData.list.length > 1) {
          // 如果当前最底层 给下一层赋值

          if (index - 1 == -1) {
            this.handleComponent(this.contentData.list[index], index);
          } else {
            // 如果不是最底层给上一层赋值
            this.handleComponent(this.contentData.list[index - 1], index - 1);
          }
          this.contentData.list.splice(index, 1);
        } else {
          this.contentData.list.splice(index, 1);
        }
      });
    },

    // 点击楼层装修
    handleComponent(val, index) {
      console.warn(val)
      this.selected = index;
      this.$set(this, "decorateData", val);
    },

    handleDrawer(val) {
      let newIndex = this.selected;

      this.decorateData = "";

      this.$set(this.contentData.list, newIndex, {
        ...val,
        options: {
          ...val.options,
        },

        // 绑定键值
        model: val.type,
      });
      this.contentData.list = JSON.parse(JSON.stringify(this.contentData.list));

      this.$set(this, "decorateData", this.contentData.list[newIndex]);
    },

    // 封装拖拽参数
    package(val, newIndex) {
      this.contentData.list[newIndex] = "";
      val = JSON.parse(JSON.stringify(val));
      this.$set(this.contentData.list, newIndex, {
        ...val,
        options: {
          ...val.options,
        },

        // 绑定键值
        model: val.type,
      });
    },
    // 拖动
    handleContentlAdd(evt) {
      const { newIndex } = evt;

      this.package(this.contentData.list[newIndex], newIndex);
      this.handleComponent(this.contentData.list[newIndex], newIndex);
    },
    handleMove() {
      return true;
    },
  },
};
</script>
<style scoped lang="scss">
@import "./style.scss";
</style>
