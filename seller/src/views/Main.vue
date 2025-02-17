<style lang="scss" scoped>
@import "./main.scss";
</style>

<template>
  <div class="main">
    <div class="sidebar-menu-con menu-bar">
      <div class="logo-con">
        <img src="../assets/logo.png" key="max-logo" />
      </div>
      <shrinkable-menu></shrinkable-menu>
    </div>
    <!-- 顶部标题栏主体 -->
    <div class="main-header-con">
      <div class="main-header">
        <div :class="{'header-avator-con':navType!=4, 'header-avator-con nav4':navType == 4}">
          <!-- 用户头像 -->
          <div class="user-dropdown-menu-con">
            <Row type="flex" justify="end" align="middle" class="user-dropdown-innercon">
              <Dropdown transfer trigger="hover" @on-click="handleClickUserDropdown">
                <div class="dropList">
                  <span class="main-user-name">{{ userInfo.storeName }}</span>
                  <Icon type="md-arrow-dropdown" />
                  <Avatar :src="userInfo.storeLogo" style="background: #fff;margin-left: 10px;"></Avatar>
                </div>
                <DropdownMenu slot="list">
                  <!-- <DropdownItem name="ownSpace">{{ $t('userCenter') }}</DropdownItem> -->
                  <DropdownItem name="changePass">{{ $t('changePass') }}</DropdownItem>
                  <DropdownItem name="loginOut" divided>{{ $t('logout') }}</DropdownItem>
                </DropdownMenu>
              </Dropdown>
            </Row>
          </div>
        </div>
      </div>
      <!-- 已打开的页面标签 -->
      <div class="tags-con">
        <tags-page-opened :pageTagsList="pageTagsList"></tags-page-opened>
      </div>
    </div>
    <div class="single-page-con">
      <div class="single-page">
        <keep-alive :include="cachePage">
          <router-view></router-view>
        </keep-alive>
      </div>
    </div>
    <!-- 全局加载动画 -->
    <circleLoading class="loading-position" v-show="loading" />
  </div>
</template>

<script>
import shrinkableMenu from "./main-components/shrinkable-menu/shrinkable-menu.vue";
import tagsPageOpened from "./main-components/tags-page-opened.vue";
import breadcrumbNav from "./main-components/breadcrumb-nav.vue";
import fullScreen from "./main-components/fullscreen.vue";
import messageTip from "./main-components/message-tip.vue";
import circleLoading from "@/views/my-components/lili/circle-loading.vue";
import Cookies from "js-cookie";
import util from "@/libs/util.js";

export default {
  components: {
    shrinkableMenu,
    tagsPageOpened,
    breadcrumbNav,
    fullScreen,
    messageTip,
    circleLoading,
  },
  data() {
    return {
      sliceNum: 5, // 展示nav数量
      userInfo: "", // 用户信息
      navType: 1, // nav类型
    };
  },
  computed: {
    loading() {
      return this.$store.state.app.loading;
    },
    pageTagsList() {
      return this.$store.state.app.storeOpenedList; // 打开的页面的页面对象
    },
    cachePage() {
      return this.$store.state.app.cachePage;
    },
    lang() {
      return this.$store.state.app.lang;
    },
    mesCount() {
      return 0;
    },
  },
  methods: {
    init() {
      // 菜单
      let pathArr = util.setCurrentPath(this, this.$route.name);
      if (pathArr.length >= 2) {
        this.$store.commit("addOpenSubmenu", pathArr[1].name);
      }

      let userInfo = JSON.parse(Cookies.get("userInfo"));
      this.userInfo = userInfo;
      this.checkTag(this.$route.name);

      let currWidth = document.body.clientWidth;
      if (currWidth <= 1200) {
        this.sliceNum = 2;
      }
    },
    selectNav(name) {
      this.$store.commit("setCurrNav", name);
      this.setStore("currNav", name);
      // 清空所有已打开标签
      // this.$store.commit("clearAllTags");
      if (this.$route.name != "home_index") {
        this.$router.push({
          name: "home_index",
        });
      }
      util.initRouter(this);
    },
    toggleClick() {
      this.shrink = !this.shrink;
    },
    handleLanDropdown(name) {
      this.$i18n.locale = name;
      this.$store.commit("switchLang", name);
    },
    handleClickUserDropdown(name) {
      if (name == "ownSpace") {
        util.openNewPage(this, "personal-enter");
        this.$router.push({
          name: "personal-enter",
        });
      } else if (name == "changePass") {
        util.openNewPage(this, "change_pass");
        this.$router.push({
          name: "change_pass",
        });
      } else if (name == "loginOut") {
        Cookies.set("accessToken", "");
        this.$store.commit("logout", this);
        this.$store.commit("clearOpenedSubmenu");
        this.setStore("accessToken", "");
        this.setStore("refreshToken", "");
        this.$router.push({ path: "/login" });
      }
    },
    checkTag(name) {
      let openpageHasTag = this.pageTagsList.some((item) => {
        if (item.name == name) {
          return true;
        }
      });
      if (!openpageHasTag) {
        //  解决关闭当前标签后再点击回退按钮会退到当前页时没有标签的问题
        util.openNewPage(
          this,
          name,
          this.$route.params || {},
          this.$route.query || {}
        );
      }
    },

    resize() {
      let currWidth = document.body.clientWidth;
      let count = currWidth / 300;
      if (count > 6) {
        this.sliceNum = 6;
      } else {
        this.sliceNum = count;
      }
    },
  },
  watch: {
    $route(to) {
      this.$store.commit("setCurrentPageName", to.name);
      let pathArr = util.setCurrentPath(this, to.name);
      if (pathArr.length > 2) {
        this.$store.commit("addOpenSubmenu", pathArr[1].name);
      }
      this.checkTag(to.name);
      localStorage.currentPageName = to.name;
    },
    lang() {
      util.setCurrentPath(this, this.$route.name); // 在切换语言时用于刷新面包屑
    },
  },
  mounted() {
    this.init();
    let that = this;
    this.resize();
    window.addEventListener("resize", function () {
      that.resize();
    });
  },
  created() {
    // 显示打开的页面的列表
    this.$store.commit("setOpenedList");
  },
};
</script>
