<template>
  <div>
    <template>
      <Row>
        <i-col span="24">
          <Card>
            <p slot="title">商家信息</p>
            <div class="flex flex_align_item">
              <p>店铺名称：{{ bill.storeName }}</p>
              <p>银行开户名：{{ bill.bankAccountName }}</p>
              <p>银行账号：{{ bill.bankAccountNumber }}</p>
              <p>开户行支行名称：{{ bill.bankName }}</p>
              <p>支行联行号：{{ bill.bankCode }}</p>
            </div>
          </Card>
        </i-col>
      </Row>
    </template>
    <template>
      <Row>
        <i-col span="24">
          <Card>
            <p slot="title">账单详细</p>

            <div class="tips-status">
              <span>商品状态</span>

              <span class="theme_color">{{
                bill.billStatus | unixSellerBillStatus
              }}</span>


              <Button
                v-if="bill.billStatus == 'CHECK'"
                size="mini"
                type="primary"
                @click="pass()"
                >付款</Button
              >
            </div>

            <i-table :columns="columns" :data="data" stripe></i-table>
          </Card>
        </i-col>
      </Row>
    </template>
    <template>
      <Tabs active-key="key1" @on-click="clickTabs">
        <Tab-pane label="入账流水" key="key1">
          <Card>
            <Table
              :loading="loading"
              border
              :columns="orderColumns"
              :data="order"
              ref="table"
            ></Table>
            <Row type="flex" justify="end" class="page">
              <Page
                :current="orderParam.pageNumber"
                :total="orderTotal"
                :page-size="orderParam.pageSize"
                @on-change="orderChangePage"
                @on-page-size-change="orderChangePageSize"
                size="small"
                show-total
                show-elevator
              ></Page>
            </Row>
          </Card>
        </Tab-pane>
        <Tab-pane label="退款流水" key="key2">
          <Card>
            <Table
              :loading="loading"
              border
              :columns="refundColumns"
              :data="refund"
              ref="table"
            ></Table>
            <Row type="flex" justify="end" class="page">
              <Page
                :current="refundParam.pageNumber"
                :total="refundTotal"
                :page-size="refundParam.pageSize"
                @on-change="getRefund()"
                @on-page-size-change="getRefund()"
                size="small"
                show-total
                show-elevator
              ></Page>
            </Row>
          </Card>
        </Tab-pane>
      </Tabs>
    </template>
  </div>
</template>
<script>
import * as filters from "@/utils/filters";
import * as API_Shop from "@/api/shops";
export default {
  name: "bill-detail",
  data() {
    return {
      columns: [ // 表头
        {
          title: "项目",
          key: "name",
          width: 250,
        },
        {
          title: "值",
          key: "value",
        },
      ],
      data: [ // 数据
        {
          name: "计算中",
          value: 0,
        },
        {
          name: "计算中",
          value: 0,
        },
        {
          name: "计算中",
          value: 0,
        },
        {
          name: "计算中",
          value: 0,
        },
        {
          name: "计算中",
          value: 0,
        },
        {
          name: "计算中",
          value: 0,
        },

        {
          name: "计算中",
          value: 0,
        },
        {
          name: "计算中",
          value: 0,
        },
      ],
      id: "", // 账单id 
      bill: {}, // 账单详情
      order: [], // 订单列表
      orderParam: { // 请求参数
        pageNumber: 1, // 当前页数
        pageSize: 10, // 页面大小
        sort: "id", // 默认排序字段
        order: "desc", // 默认排序方式
        flowType: "PAY",
        startDate: null,
        endDate: null,
      },
      orderColumns: [ // 订单表头
        {
          title: "入账时间",
          key: "createTime",
          minWidth: 120,
          tooltip: true
        },
        {
          title: "订单编号",
          key: "sn",
          minWidth: 120,
          tooltip: true
        },
        {
          title: "订单金额",
          key: "finalPrice",
          width: 120,
          render: (h, params) => {
            return h(
              "div",
              this.$options.filters.unitPrice(params.row.finalPrice, "￥")
            );
          },
        },
        {
          title: "平台分佣",
          key: "commissionPrice",
          width: 120,
          render: (h, params) => {
            return h(
              "div",
              this.$options.filters.unitPrice(params.row.commissionPrice, "￥")
            );
          },
        },
        {
          title: "平台优惠券",
          key: "siteCouponPrice",
          minWidth: 120,
          render: (h, params) => {
            if(params.row.siteCouponPrice == null){
              return h(
                "div",
                "-"
              );
            }else{
              return h(
                "div",
                this.$options.filters.unitPrice(params.row.siteCouponPrice, "￥")
              );
            }

          },
        },
        {
          title: "分销金额",
          key: "distributionRebate",
          width: 120,
          render: (h, params) => {
            if(params.row.distributionRebate == null){
              return h(
                "div",
                "-"
              );
            }else{
              return h(
                "div",
                this.$options.filters.unitPrice(params.row.distributionRebate, "￥")
              );
            }

          },
        },
        {
          title: "应结金额",
          key: "billPrice",
          width: 120,
          render: (h, params) => {
            return h(
              "div",
              this.$options.filters.unitPrice(params.row.billPrice, "￥")
            );
          },
        },

      ],
      refund: [], // 退款单
      refundParam: { // 请求参数
        flowTypeEnum: "PAY",
        pageNumber: 1, // 当前页数
        pageSize: 10, // 页面大小
        sort: "id", // 默认排序字段
        order: "desc", // 默认排序方式
        flowType: "REFUND",
        startDate: null,
        endDate: null,
      },
      refundColumns: [ // 退款单表头
        {
          title: "退款时间",
          key: "createTime",
          minWidth: 120,
          tooltip: true
        },
        {
          title: "退款流水编号",
          key: "sn",
          minWidth: 120,
          tooltip: true        
        },
        {
          title: "订单编号",
          key: "sn",
          minWidth: 120,
          tooltip: true        
        },
        {
          title: "退款金额",
          key: "finalPrice",
          width: 120,
          render: (h, params) => {
            return h(
              "div",
              this.$options.filters.unitPrice(params.row.finalPrice, "￥")
            );
          },
        },
        {
          title: "退还佣金",
          key: "commissionPrice",
          minWidth: 120,
          render: (h, params) => {
            return h(
              "div",
              this.$options.filters.unitPrice(params.row.commissionPrice, "￥")
            );
          },
        },

        {
          title: "退还平台优惠券",
          key: "siteCouponCommission",
          minWidth: 110
        },
        {
          title: "退还分销",
          key: "distributionRebate",
          minWidth: 120,
          render: (h, params) => {
            if(params.row.distributionRebate == null){
              return h(
                "div",
                "-"
              );
            }else{
              return h(
                "div",
                this.$options.filters.unitPrice(params.row.distributionRebate, "￥")
              );
            }

          },
        },

        {
          title: "合计金额",
          key: "billPrice",
          minWidth: 120,
          render: (h, params) => {
            if(params.row.billPrice == null){
              return h(
                "div",
                "-"
              );
            }else{
              return h(
                "div",
                this.$options.filters.unitPrice(params.row.billPrice, "￥")
              );
            }

          },
        },


      ],
      orderTotal: 0, // 订单总数
      refundTotal: 0, // 退款单总数
    };
  },
  methods: {
    //订单页数发生变化
    orderChangePage(v) {
      this.orderParam.pageNumber = v;
      this.getOrder();
    },
    //订单每页条数变化
    orderChangePageSize(v) {
      this.orderParam.pageSize = v;
      this.getOrder();
    },
    //退款单页数发生变化
    refundOrderChangePage(v) {
      this.refundParam.pageNumber = v;
      this.getRefund()
    },
    //退款单每页条数变化
    refundOrderChangePageSize(v) {
      this.refundParam.pageSize = v;
      tthis.getRefund()
    },
    clickTabs(index) {
      if (index == 1) {
        this.orderParam.flowType = "REFUND";
        this.getRefund();
      } else {
        this.orderParam.flowType = "PAY";
        this.getOrder();
      }
    },


    pass() {
      API_Shop.pay(this.id).then((res) => {
        if (res.success) {
          this.$Message.success(res.message);
          this.init();
        }
      });
    },

    init() {
      this.id = this.$route.query.id;
      this.getDetail();
    },
    getDetail() {
      API_Shop.getBuyBillDetail(this.id).then((res) => {
        if (res.success) {
          this.bill = res.result;
          //初始化表哥
          this.initTable();
          //初始化订单信息
          this.orderParam.startDate = this.bill.startTime;
          this.orderParam.endDate = this.bill.endTime;
          this.refundParam.startDate = this.bill.startTime;
          this.refundParam.endDate = this.bill.endTime;
          this.getOrder();
        }
      });
    },
    initTable() {
      let bill = this.bill;
      this.data[0].name = "结算单号";
      this.data[0].value = bill.sn;

      this.data[1].name = "起止日期";
      this.data[1].value = bill.startTime + "~" + bill.endTime;

      this.data[2].name = "出帐日期";
      this.data[2].value = bill.createTime;

      this.data[3].name = "当前状态";
      this.data[3].value = filters.unixSellerBillStatus(bill.billStatus);

      this.data[4].name = "当前店铺";
      this.data[4].value = bill.storeName;

      this.data[5].name = "平台打款时间";
      this.data[5].value = bill.payTime === null ? "未付款" : bill.payTime;

      this.data[6].name = "结算金额";
      this.data[6].value = filters.unitPrice(bill.billPrice?bill.billPrice:0, "¥");

      this.data[7].name = "结算详细";
      this.data[7].value =
        "最终结算金额(" +
        filters.unitPrice(bill.billPrice?bill.billPrice:0, "¥") +
        ") = 订单付款总金额(" +
        filters.unitPrice(bill.orderPrice?bill.orderPrice:0, "¥") +
        ") - 退单金额(" +
        filters.unitPrice(bill.refundPrice?bill.refundPrice:0, "¥") +
        ")" +
        "- 平台收取佣金(" +
        filters.unitPrice(bill.commissionPrice?bill.commissionPrice:0, "¥") +
        ") + 退单产生退还佣金金额(" +
        filters.unitPrice(bill.refundCommissionPrice?bill.refundCommissionPrice:0, "¥") +
        ") - 分销返现支出(" +
        filters.unitPrice(bill.distributionCommission?bill.distributionCommission:0, "¥") +
        ") + 退单分销返现返还(" +
        filters.unitPrice(bill.distributionRefundCommission?bill.distributionRefundCommission:0, "¥") +
        ") - 平台优惠券支出(" +
        filters.unitPrice(bill.siteCouponCommission?bill.siteCouponCommission:0, "¥") +
        ") + 退单平台优惠券返还(" +
        filters.unitPrice(bill.siteCouponRefundCommission?bill.siteCouponRefundCommission:0, "¥") +
        ")";
    },
    getOrder() {
      API_Shop.getStoreFlow(this.id, this.orderParam).then((res) => {
        if (res.result) {
          this.order = res.result.records;
          this.orderTotal = res.result.total;
        }
      });
      this.orderTotal = this.order.length;
    },
    getRefund() {
      API_Shop.getStoreFlow(this.id, this.orderParam).then((res) => {
        this.loading = false;
        if (res.result) {
          this.refund = res.result.records;
          this.refundTotal = res.result.total;

          this.$set(this, "refund", res.result.records);
          console.log();
        }
      });
      this.refundTotal = this.refund.length;
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<style scoped lang="scss">
.flex {
  justify-content: space-between;
  flex-wrap: wrap;
  > p {
    width: 50%;
    margin: 15px 0;
  }
}
.tips-status {
  padding: 18px;
  > span {
    font-weight: bold;
    margin-right: 8px;
  }
  > span:nth-of-type(2) {
    color: $theme_color;
  }
}
</style>
