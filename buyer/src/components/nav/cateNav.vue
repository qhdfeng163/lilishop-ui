<template>
  <div class="cate-nav">
    <div class="nav-con">
      <div class="all-categories hover-pointer" @mouseenter="showFirstList = true" @mouseleave="showFirstList = false">全部商品分类</div>
      <ul class="nav-item" v-if="showNavBar">
        <li
          class="hover-color"
          v-for="(item, index) in navList.list"
          :key="index"
          @click="linkTo(item.url)"
        >
          {{ item.name }}
        </li>
      </ul>
    </div>
    <!-- 全部商品分类 -->
    <div class="cate-list" v-show="showAlways || showFirstList" @mouseenter="showFirstList = true" @mouseleave="showFirstList = false">
      <!-- 第一级分类 -->
      <div class="nav-side" @mouseleave="panel = false">
        <ul>
          <li v-for="(item, index) in cateList" :key="index" @mouseenter="showDetail(index)" >
            <span class="nav-side-item" @click="goGoodsList(item.id)">{{item.name}}</span>
            <span v-for="(second, secIndex) in item.children" :key="secIndex">
              <span v-if="secIndex < 2" > / </span>
              <span @click="goGoodsList(second.id, second.parentId)" class="nav-side-item" v-if="secIndex < 2">{{second.name}}</span>
            </span>
          </li>
        </ul>
      </div>
      <!-- 展开分类 -->
      <div
        class="detail-item-panel"
        v-show="panel"
        @mouseenter="panel = true"
        @mouseleave="panel = false"
      >
        <div class="nav-detail-item">
          <template v-for="(item, index) in panelData">
            <span @click="goGoodsList(item.id, item.parentId)" v-if="index < 8" :key="index">{{ item.name }}<Icon type="ios-arrow-forward" /></span>
          </template>
        </div>
        <ul>
          <li
            v-for="(items, index) in panelData"
            :key="index"
            class="detail-item-row"
          >
            <span class="detail-item-title" @click="goGoodsList(items.id,items.parentId)">
              {{ items.name }} <Icon type="ios-arrow-forward" />
              <span class="glyphicon glyphicon-menu-right"></span>
            </span>
            <div>
              <span v-for="(item, subIndex) in items.children" @click="goGoodsList(item.id,items.id,items.parentId)"
              :key="subIndex" class="detail-item">{{ item.name }}</span>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { getCategory } from '@/api/goods';
import storage from '@/plugins/storage.js'
export default {
  name: 'GoodsListNav',
  props: {
    showAlways: { // 总是显示下拉分类
      default: false,
      type: Boolean
    },
    showNavBar: { // 显示全部商品分类右侧导航条
      default: true,
      type: Boolean
    }
  },
  data () {
    return {
      cateList: [], // 分类数据
      panel: false, // 二级分类展示
      panelData: [], // 二级分类数据
      showFirstList: false // 始终展示一级列表
    }
  },
  computed: {
    navList () { // 导航列表
      return JSON.parse(storage.getItem('navList')) || []
    }
  },
  methods: {
    getCate () { // 获取分类数据
      getCategory(0).then(res => {
        if (res.success) {
          this.cateList = res.result;
          // 过期时间
          var expirationTime = new Date().setHours(new Date().getHours() + 1);
          // 存放过期时间
          localStorage.setItem('category_expiration_time', expirationTime);
          // 存放分类信息
          localStorage.setItem('category', JSON.stringify(res.result))
        }
      });
    },
    showDetail (index) { // 展示全部分类
      this.panel = true
      this.panelData = this.cateList[index].children
    },
    goGoodsList (id, secondId, firstId) { // 分类共有三级，传全部分类过去
      const arr = [firstId, secondId, id]
      if (!arr[1]) {
        arr.splice(0, 2)
      }
      if (!arr[0]) {
        arr.shift()
      }
      let routerUrl = this.$router.resolve({
        path: '/goodsList',
        query: {categoryId: arr.toString()}
      })
      window.open(routerUrl.href, '_blank')
    }
  },
  mounted () {
    if (localStorage.getItem('category') && localStorage.getItem('category_expiration_time')) {
      // 如果缓存过期，则获取最新的信息
      if (new Date() > localStorage.getItem('category_expiration_time')) {
        this.getCate();
        return;
      }
      this.cateList = JSON.parse(localStorage.getItem('category'))
    } else {
      this.getCate()
    }
  }
};
</script>

<style scoped lang="scss">
.cate-nav{
  width: 1200px;
  position: relative;
  margin: 0 auto;
}
/** 商品分类 */
.nav-con {
  width: 1200px;
  height: 40px;
  // background: #eee;
  margin: 0 auto;
  display: flex;
  .all-categories {
    width: 200px;
    line-height: 40px;
    color: #fff;
    background-color: $theme_color;
    text-align: center;
    font-size: 16px;
  }
  .nav-item {
    width: 1000px;
    height: 40px;
    line-height: 40px;
    overflow: hidden;
    list-style: none;
    background-color: #eee;
    display: flex;
    li {
      font-size: 16px;
      font-weight: bold;
      margin-left: 20px;
      color: rgb(89, 88, 88);
      font-size: 15px;
      &:hover {
        color: $theme_color;
      }
    }
  }
}
.cate-list{
  margin: 0 auto;
  position: absolute;
  z-index: 1000;
}
.nav-item li {
  float: left;
  font-size: 16px;
  font-weight: bold;
  margin-left: 30px;
}
.nav-item a {
  text-decoration: none;
  color: #555555;
}
.nav-item a:hover {
  color: $theme_color;
}

.nav-side {
  width: 200px;
  float: left;
  padding: 0px;
  color: #fff;
  background-color: #6e6568;
  height: 335px;
  overflow: hidden;
}
.nav-side ul {
  width: 100%;
  padding: 0px;
  padding-top: 5px;
  list-style: none;
}
.nav-side li {
  padding: 7.5px 0;
  padding-left: 12px;
  font-size: 13px;
  line-height: 18px;
}
.nav-side li:hover {
  background: #999395;
}
.nav-side-item:hover {
  cursor: pointer;
  color: $theme_color;
}

/*显示商品详细信息*/
.detail-item-panel {
  width: 815px;
  min-height: 340px;
  background-color: #fff;
  box-shadow: 0px 0px 15px #ccc;
  position: absolute;
  top: 0;
  left: 200px;
  z-index: 1000;
  padding: 15px;
}
.nav-detail-item {
  margin-top: 5px;
  margin-bottom: 15px;
  cursor: pointer;
  color: #eee;
}
.nav-detail-item span {
  padding: 6px;
  padding-left: 12px;
  margin-right: 15px;
  font-size: 12px;
  background-color: #6e6568;
}
.nav-detail-item span:hover {
  background-color: $theme_color;
}
.detail-item-panel li {
  line-height: 30px;
  // margin-left: 40px;
}
.detail-item-title {
  font-weight: bold;
  font-size: 12px;
  cursor: pointer;
  color: #555555;
  padding-right: 10px;
  width: 81px;
  text-align: right;
}
.detail-item-title:hover {
  color: $theme_color;
}
.detail-item-row  {
  display: flex;
  >div{flex: 1;}
}
.detail-item {
  font-size: 12px;
  padding-left: 8px;
  padding-right: 8px;
  cursor: pointer;
  border-left: 1px solid #ccc;
  &:first-child{
    border: none;
    padding-left: 0;
  }
}
.detail-item:hover {
  color: $theme_color;
}
</style>
