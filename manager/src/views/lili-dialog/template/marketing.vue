<template>
  <div class="wrapper">
    <div class="list">

      <div class="list-item" @click="clickPromotion(item,index)" v-for="(item,index) in Object.keys(promotionList)" :key="index" :class="{active:selectedIndex == index}">
        {{ typeOption[item].title}}
      </div>

      <!-- <div class="list-item" >暂无活动</div> -->
    </div>
    <div class="content">
      <div v-if="showPromotionList">
        <!-- <div class="search-views">
          <Input v-model="value11" disabled class="search">
          <span slot="prepend">店铺名称</span>
          </Input>
          <Button type="primary">选择</Button>

        </div> -->

        <div class="tables">

          <Table height="350" border tooltip :loading="loading" :columns="activeColumns" :data="showPromotionList"></Table>

          <Page @on-change="(val) => {params.pageNumber = val; } " :current="params.pageNumber" :page-size="params.pageSize" class="page" :total="totals" size="small" show-elevator />

        </div>

      </div>

    </div>
  </div>
</template>
<script>
import {
  getAllPromotion,
  getPromotionSeckill,
  getPromotionGoods,
} from "@/api/promotion";

export default {
  data() {
    return {
      totals: "", // 总数
      loading: true, //表格请求数据为true
      promotionList: "", // 活动列表
      selectedIndex: 0, //左侧菜单选择
      promotions: "", //选中的活动key
      index: 999, // 已选下标
      params: { // 请求参数
        pageNumber: 1,
        pageSize: 10,
      },
      pintuanColumns: [ // 表头
        {
          title: "活动标题",
          key: "title",
          tooltip: true,
          width: 250,
        },
        {
          title: "商品名称",
          key: "goodsName",
          tooltip: true,
        },
        {
          title: "店铺名称",
          key: "storeName",
          tooltip: true,
        },
        {
          title: "开始时间",
          key: "startTime",
          tooltip: true,
        },
        {
          title: "结束时间",
          key: "endTime",
          tooltip: true,
        },
        {
          title: "操作",
          key: "action",
          fixed: "right",
          width: 100,
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: this.index == params.index ? "primary" : "",
                    size: "small",
                  },
                  on: {
                    click: () => {
                      this.selectedPromotion(params);
                    },
                  },
                },
                this.index == params.index ? "已选" : "选择"
              ),
            ]);
          },
        },
      ],
      seckillColumns: [
        {
          title: "商品名称",
          key: "goodsName",
          tooltip: true,
          width: 200,
        },
        {
          title: "店铺名称",
          key: "storeName",
          tooltip: true,
        },
        {
          title: "活动时间",
          key: "timeLine",
          tooltip: true,
          render: (h, params) => {
            return h("div", {}, `${params.row.timeLine}点`);
          },
        },

        {
          title: "原价",
          key: "originalPrice",
          tooltip: true,
          render: (h, params) => {
            return h(
              "div",
              {},
              this.$options.filters.unitPrice(params.row.originalPrice)
            );
          },
        },
        {
          title: "现价",
          key: "price",
          tooltip: true,
          render: (h, params) => {
            return h(
              "div",
              {
                style: {},
              },
              this.$options.filters.unitPrice(params.row.price, '￥')
            );
          },
        },
        {
          title: "状态",
          key: "promotionApplyStatus",
          tooltip: true,
          render: (h, params) => {
            return h(
              "div",
              {
                style: {},
              },
              params.row.promotionApplyStatus == "APPLY"
                ? "申请"
                : params.row.promotionApplyStatus == "PASS"
                ? "通过"
                : "拒绝"
            );
          },
        },

        {
          title: "操作",
          key: "action",
          width: 100,
          fixed: "right",
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: this.index == params.index ? "primary" : "",
                    size: "small",
                  },
                  on: {
                    click: () => {
                      this.selectedPromotion(params);
                    },
                  },
                },
                this.index == params.index ? "已选" : "选择"
              ),
            ]);
          },
        },
      ],

      activeColumns: [], // 活动表头

      columns: [
        {
          title: "活动标题",
          key: "title",
          tooltip: true,
          width: 200,
        },
        {
          title: "商品名称",
          key: "goodsName",
          tooltip: true,
        },
        {
          title: "活动开始时间",
          key: "startTime",
          tooltip: true,
        },
        {
          title: "活动结束时间",
          key: "endTime",
          tooltip: true,
        },
        {
          title: "操作",
          key: "action",
          fixed: "right",
          width: 100,
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: this.index == params.index ? "primary" : "",
                    size: "small",
                  },
                  on: {
                    click: () => {
                      this.selectedPromotion(params);
                    },
                  },
                },
                this.index == params.index ? "已选" : "选择"
              ),
            ]);
          },
        },
      ],

      promotionData: "", //商品集合

      showPromotionList: [], //显示当前促销的商品
      typeOption: { // 活动选项
        FULL_DISCOUNT: {
          title: "满减",
          methodsed: () => {
            this.showPromotionList = [];
            this.activeColumns = this.pintuanColumns;
            this.params.promotionType = "FULL_DISCOUNT";

            this.sortGoods("FULL_DISCOUNT");
          },
        },
        PINTUAN: {
          title: "拼团",
          methodsed: (id) => {
            this.showPromotionList = [];
            this.activeColumns = this.pintuanColumns;
            this.params.promotionType = "PINTUAN";
            this.sortGoods("PINTUAN");
          },
        },
        SECKILL: {
          title: "秒杀",
          methodsed: () => {
            this.showPromotionList = [];
            this.activeColumns = this.seckillColumns;
            this.params.seckillId = this.promotionList["SECKILL"].id;
            delete this.params.promotionType;

            getPromotionSeckill(this.params).then((res) => {
              delete this.params.seckillId;
              this.showPromotionList = this.showPromotionList.concat(
                res.result.records
              );

              this.totals = res.result.total;
              console.log(this.totals);
            });
          },
        },
        COUPON: {
          title: "优惠券",
          methodsed: () => {
            this.showPromotionList = [];
            this.params.promotionType = "COUPON";
            this.sortGoods("COUPON");
          },
        },
        POINTS_GOODS: {
          title: "积分商品",
          methodsed: () => {
            this.showPromotionList = [];
            this.params.promotionType = "POINTS_GOODS";
            this.sortGoods("POINTS_GOODS");
          },
        },
      },
    };
  },
  mounted() {
    this.init();
  },
  watch: {
    params: {
      handler() {
        this.index = 999;
        this.typeOption[this.promotions] &&
          this.typeOption[this.promotions].methodsed();
      },
      deep: true,
    },
  },

  methods: {
    sortGoods(type) {
      this.loading = true;
      this.params.pageNumber - 1;
      getPromotionGoods(this.promotionList[type].id, this.params).then(
        (res) => {
          this.loading = false;
          if (res.result) {
            this.$nextTick(() => {
              this.showPromotionList = this.showPromotionList.concat(
                res.result.records
              );
              console.log(this.totals);
              this.totals = res.result.total;
            });
          }
        }
      );
    },
    // 选择活动
    selectedPromotion(val) {
      console.log(val);
      val.row.___type = "marketing";
      val.row.___promotion = this.promotions;
      this.$emit("selected", [val.row]);
      console.log([val.row]);

      this.index = val.index;
    },
    // 获取所有营销的活动
    async init() {
      let res = await getAllPromotion();
      if (res.success) {
        this.loading = false;
        this.getPromotion(res);
        this.clickPromotion(this.typeOption[Object.keys(res.result)[0]], 0);
      } else {
        this.loading = false;
      }
    },
    getPromotion(res) {
      this.promotionList = res.result;

      if (Object.keys(res.result).length) {
        this.typeOption[Object.keys(res.result)[0]].methodsed(
          this.promotionList[Object.keys(res.result)[0]].id
        );
      }
    },

    // 点击某个活动查询活动列表
    clickPromotion(val, i) {
      this.promotions = val;
      this.selectedIndex = i;
      this.params.pageNumber = 1;
      this.typeOption[val] &&
        this.typeOption[val].methodsed(this.promotionList[val].id);
    },
  },
};
</script>
<style lang="scss" scoped>
img {
  max-width: 100% !important;
}
.search {
  width: 300px;
}
.page {
  margin-top: 2vh;
  text-align: right;
}
.time {
  font-size: 12px;
}
.tables {
  height: 400px;
  margin-top: 20px;
  overflow: auto;
  width: 100%;
}
/deep/ .ivu-table-wrapper {
  width: 100%;
}
.list {
  margin: 0 1.5%;
  height: 400px;
  overflow: auto;
  > .list-item {
    padding: 10px;
    transition: 0.35s;
    cursor: pointer;
  }
  .list-item:hover {
    background: #ededed;
  }
}
.list {
  flex: 1;
  width: auto;
}
.content {
  overflow: hidden;
  flex: 4;
}
.active {
  background: #ededed;
}
.wrapper {
  overflow: hidden;
}
.search-views {
  display: flex;
  > * {
    margin: 0 4px;
  }
}
</style>
