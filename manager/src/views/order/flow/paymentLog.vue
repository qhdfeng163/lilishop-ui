<template>
  <div class="search">
    <Card>
      <Row @keydown.enter.native="handleSearch">
        <Form ref="searchForm" :model="searchForm" inline :label-width="70" class="search-form">
          <Form-item label="订单号" prop="sn">
            <Input type="text" v-model="searchForm.sn" placeholder="订单/交易号" clearable style="width: 200px" />
          </Form-item>
          <Form-item label="付款状态" prop="orderStatus">
            <Select v-model="searchForm.payStatus" placeholder="请选择" clearable style="width: 200px">
              <Option value="UNPAID">未付款</Option>
              <Option value="PAID">已付款</Option>
            </Select>
          </Form-item>
          <Form-item label="支付方式" prop="orderStatus">
            <Select v-model="searchForm.paymentMethod" placeholder="请选择" clearable style="width: 200px">
              <Option value="WECHAT">微信</Option>
              <Option value="ALIPAY">支付宝</Option>
              <Option value="WALLET">余额</Option>
              <Option value="BANK_TRANSFER">银行转账</Option>
              <Option value="">暂未付款</Option>
            </Select>
          </Form-item>
          <Form-item label="支付时间">
            <DatePicker v-model="searchForm" type="datetimerange" format="yyyy-MM-dd" clearable @on-change="selectDateRange" placeholder="选择起始时间" style="width: 200px"></DatePicker>
          </Form-item>
          <Button @click="handleSearch" type="primary" icon="ios-search" class="search-btn">搜索</Button>
        </Form>
      </Row>
      <Table :loading="loading" border :columns="columns" :data="data" ref="table" sortable="custom" @on-sort-change="changeSort" @on-selection-change="changeSelect"></Table>
      <Row type="flex" justify="end" class="page">
        <Page :current="searchForm.pageNumber" :total="total" :page-size="searchForm.pageSize" @on-change="changePage" @on-page-size-change="changePageSize" :page-size-opts="[10, 20, 50]"
          size="small" show-total show-elevator show-sizer></Page>
      </Row>
    </Card>
  </div>
</template>

<script>
import * as API_Order from "@/api/order";

export default {
  name: "paymentLog",
  components: {},
  data() {
    return {
      loading: true, // 表单加载状态
      drop: false, // 更多搜索项
      dropDownIcon: "ios-arrow-down", // drop图标
      searchForm: {
        // 搜索框初始化对象
        pageNumber: 1, // 当前页数
        pageSize: 10, // 页面大小
        sort: "createTime", // 默认排序字段
        order: "desc", // 默认排序方式
        startDate: "", // 起始时间
        endDate: "", // 终止时间
        sn: "",
        payStatus: "",
      },
      columns: [
        {
          title: "订单/交易编号",
          key: "sn",
          minWidth: 180,
          tooltip: true,
        },
        {
          title: "支付方式",
          key: "paymentMethod",
          width: 120,
          align: "center",
          render: (h, params) => {
            if (params.row.paymentMethod === "WECHAT") {
              return h("div", [h("Tag", {props: {color: "green",},}, "微信"),]);
            } else if (params.row.paymentMethod === "ALIPAY") {
              return h("div", [h("Tag", {props: {color: "blue",},}, "支付宝"),]);
            } else if (params.row.paymentMethod === "WALLET") {
              return h("div", [h("Tag", {props: {color: "geekblue",},}, "余额支付"),]);
            } else if (params.row.paymentMethod === "BANK_TRANSFER") {
              return h("div", [h("Tag", {props: {color: "orange",},}, "银行转帐"),]);
            } else {
              return h("div", [h("Tag", {}, "暂未付款")]);
            }
          },
        },
        {
          title: "第三方流水",
          key: "receivableNo",
          minWidth: 130,
          render: (h, params) => {
            return h("div", [
              h("span", {}, params.row.receivableNo || "暂无流水号"),
            ]);
          },
        },
        {
          title: "客户端",
          key: "clientType",
          width: 130,
          render: (h, params) => {
            if (params.row.clientType === "WECHAT_MP" || params.row.clientType === '小程序') {
              return h("div", [h("span", {}, "小程序")]);
            } else if (params.row.clientType === "APP") {
              return h("div", [h("span", {}, "APP")]);
            } else if (params.row.clientType === "PC") {
              return h("div", [h("span", {}, "PC网页")]);
            } else if (params.row.clientType === "H5" || params.row.clientType === 'wap') {
              return h("div", [h("span", {}, "移动端")]);
            }
          },
        },
        {
          title: "支付时间",
          key: "paymentTime",
          width: 200,
          render: (h, params) => {
            return h("div", [
              h("span", {}, params.row.paymentTime || "暂无支付时间"),
            ]);
          },
        },
        {
          title: "订单金额",
          fixed: "right",
          key: "flowPrice",
          minWidth: 80,
          render: (h, params) => {
            return h(
              "div",
              this.$options.filters.unitPrice(params.row.flowPrice, "￥")
            );
          },
        },
        {
          title: "支付状态",
          key: "payStatus",
          width: 95,
          fixed: "right",
          render: (h, params) => {
            if (params.row.payStatus == "PAID") {
              return h("div", [h("span", {}, "已付款")]);
            } else {
              return h("div", [h("span", {}, "未付款")]);
            }
          },
        },
      ],
      data: [], // 表单数据
      total: 0, // 表单数据总数
    };
  },
  methods: {
    dropDown() {
      if (this.drop) {
        this.dropDownContent = "展开";
        this.dropDownIcon = "ios-arrow-down";
      } else {
        this.dropDownContent = "收起";
        this.dropDownIcon = "ios-arrow-up";
      }
      this.drop = !this.drop;
    },
    init() {
      this.getDataList();
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
    handleReset() {
      this.$refs.searchForm.resetFields();
      this.searchForm.pageNumber = 1;
      this.searchForm.pageSize = 10;
      this.selectDate = null;
      this.searchForm.startDate = "";
      this.searchForm.endDate = "";
      // 重新加载数据
      this.getDataList();
    },
    changeSort(e) {
      this.searchForm.sort = e.key;
      this.searchForm.order = e.order;
      if (e.order === "normal") {
        this.searchForm.order = "";
      }
      this.getDataList();
    },

    changeSelect(e) {
      this.selectList = e;
      this.selectCount = e.length;
    },
    selectDateRange(v) {
      if (v) {
        this.searchForm.startDate = v[0];
        this.searchForm.endDate = v[1];
      }
    },
    getDataList() {
      this.loading = true;
      API_Order.paymentLog(this.searchForm).then((res) => {
        this.loading = false;
        if (res.success) {
          this.data = res.result.records;
          this.total = res.result.total;
        }
      });
      this.total = this.data.length;
      this.loading = false;
    },
  },
  mounted() {
    this.init();
  },
};
</script>
<style lang="scss" scoped>
// 建议引入通用样式 可删除下面样式代码
@import "@/styles/table-common.scss";
</style>
