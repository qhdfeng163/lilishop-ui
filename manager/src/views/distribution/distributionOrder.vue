<template>
  <div class="search">
    <Card>
      <Row v-show="openSearch" @keydown.enter.native="handleSearch">
        <Form ref="searchForm" :model="searchForm" inline :label-width="70" class="search-form">
          <Form-item label="订单编号" prop="orderSn">
            <Input
              type="text"
              v-model="searchForm.orderSn"
              placeholder="请输入订单编号"
              clearable
              style="width: 200px"
            />
          </Form-item>
          <Form-item label="分销商" prop="distributionName">
            <Input
              type="text"
              v-model="searchForm.distributionName"
              placeholder="请输入分销商名称"
              clearable
              style="width: 200px"
            />
          </Form-item>
          <Form-item label="店铺名称">
            <Select v-model="searchForm.shopId" placeholder="请选择" @on-query-change="searchChange" filterable
                    clearable style="width: 150px">
              <Option v-for="item in shopList" :value="item.id" :key="item.id">{{ item.storeName }}</Option>
            </Select>
          </Form-item>
          <Form-item label="订单时间">
            <DatePicker type="daterange" v-model="timeRange" format="yyyy-MM-dd" placeholder="选择时间"
                        style="width: 210px"></DatePicker>
          </Form-item>
          <Button @click="handleSearch" type="primary" icon="ios-search" class="search-btn">搜索</Button>
        </Form>
      </Row>
      <Table :loading="loading" border :columns="columns" :data="data" ref="table" sortable="custom"></Table>
      <Row type="flex" justify="end" class="page">
        <Page :current="searchForm.pageNumber" :total="total" :page-size="searchForm.pageSize"
          @on-change="changePage" @on-page-size-change="changePageSize" :page-size-opts="[10,20,50]"
          size="small" show-total show-elevator show-sizer></Page>
      </Row>
    </Card>
  </div>
</template>

<script>
  import {
    getDistributionOrder
  } from "@/api/distribution";
  import {orderStatusList} from './dataJson'
  import {getShopListData} from '@/api/shops'

  export default {
    name: "distributionOrder",
    components: {},
    data() {
      return {
        timeRange: [], // 范围时间
        orderStatusList, // 订单状态列表
        shopList: [], // 店铺列表
        distributionId: this.$route.query.id, // 分销id
        openSearch: true, // 显示搜索
        openTip: true, // 显示提示
        loading: true, // 表单加载状态
        searchForm: { // 搜索框初始化对象
          pageNumber: 1, // 当前页数
          pageSize: 10, // 页面大小
        },
        columns: [
          {
            title: "订单编号",
            key: "orderSn",
            minWidth: 120,
            tooltip: true
          },

          {
            title: "实付金额",
            key: "orderPrice",
            width: 130,
            sortable: false,
            render: (h, params) => {
              if(params.row.orderPrice == null){
                return h("div", this.$options.filters.unitPrice(0, '￥'));
              }else{
                return h("div", this.$options.filters.unitPrice(params.row.orderPrice, '￥'));
              }

            }
          },
          {
            title: "退款金额",
            key: "returnMoney",
            width: 130,
            sortable: false,
            render: (h, params) => {
              if(params.row.orderPrice == null){
                return h("div", this.$options.filters.unitPrice(0, '￥'));
              }else{
                return h("div", this.$options.filters.unitPrice(params.row.returnMoney, '￥'));
              }
            }
          },
          {
            title: "商品名称",
            key: "goodsName",
            minWidth: 120,
            tooltip: true
          },
          {
            title: "分销商",
            key: "distributionName",
            minWidth: 120,
            tooltip: true
          },
          {
            title: "店铺名称",
            key: "storeName",
            minWidth: 120,
          },
          {
            title: "状态",
            key: "distributionOrderStatus",
            width: 100,
            sortable: false,
            render: (h, params) => {
              if (params.row.distributionOrderStatus == 'COMPLETE_CASH') {
                return h('div', '提现完成')
              } else if (params.row.distributionOrderStatus == 'WAIT_BILL') {
                return h('div', '待结算')
              } else if (params.row.distributionOrderStatus == 'WAIT_CASH') {
                return h('div', '待提现')
              }
            }
          },

          {
            title: "佣金金额",
            key: "rebateGrade",
            width: 120,
            sortable: false,
            render: (h, params) => {
              if(params.row.rebateGrade == null){
                return h("div", this.$options.filters.unitPrice(0, '￥'));
              }else{
                return h("div", this.$options.filters.unitPrice(params.row.rebateGrade, '￥'));
              }

            }
          },
          {
            title: "创建时间",
            key: "createTime",
            width: 180,
            sortable: false,
          }
        ],
        data: [], // 表单数据
        total: 0 // 表单数据总数
      };
    },
    methods: {
      init() {
        this.getDataList();
        this.getShopList()
      },
      changePage(v) {
        this.searchForm.pageNumber = v;
        this.getDataList();
      },
      changePageSize(v) {
        this.searchForm.pageSize = v;
        this.getDataList();
      },
      handleSearch() {
        this.searchForm.pageNumber = 1;
        this.searchForm.pageSize = 10;
        this.getDataList();
      },

      getDataList() {
        this.searchForm.distributionId = this.distributionId;
        this.loading = true;
        if (this.timeRange && this.timeRange[0]) {
          let startTime = this.timeRange[0]
          let endTime = this.timeRange[1]
          this.searchForm.startTime = this.$options.filters.unixToDate(startTime / 1000)
          this.searchForm.endTime = this.$options.filters.unixToDate(endTime / 1000)
        }
        console.log(this.searchForm)
        // 带多条件搜索参数获取表单数据 请自行修改接口
        getDistributionOrder(this.searchForm).then(res => {
          this.loading = false;
          if (res.success) {
            this.data = res.result.records;

            this.total = res.result.total;
          }
        });
        this.total = this.data.length;
        this.loading = false;
      },
      getShopList(val) { // 获取店铺列表
        const params = {
          pageNumber: 1,
          pageSize: 10,
          storeName: ''
        }
        if (val) {
          params.storeName = val;
        } else {
          params.storeName = ''
        }

        getShopListData(params).then(res => {
          this.shopList = res.result.records
        })
      },
      searchChange(val) { // 店铺搜索，键盘点击回调
        this.getShopList(val)
      }
    },
    mounted() {
      this.init();
    },
    watch: {
      $route(e) {
        this.distributionId = e.query.id ? e.query.id : undefined;
        this.getDataList();
      }
    }
  };
</script>
<style lang="scss" >
  @import "@/styles/table-common.scss";
</style>

