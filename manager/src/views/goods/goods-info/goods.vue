<template>
  <div class="search">
    <Card>
      <Row @keydown.enter.native="handleSearch">
        <Form
          ref="searchForm"
          :model="searchForm"
          inline
          :label-width="70"
          class="search-form"
        >
          <Form-item label="商品名称" prop="goodsName">
            <Input
              type="text"
              v-model="searchForm.goodsName"
              placeholder="请输入商品名称"
              clearable
              style="width: 200px"
            />
          </Form-item>
          <Form-item label="商品编号" prop="sn">
            <Input
              type="text"
              v-model="searchForm.sn"
              placeholder="请输入商品编号"
              clearable
              style="width: 200px"
            />
          </Form-item>
          <Form-item label="状态" prop="status">
            <Select
              v-model="searchForm.marketEnable"
              placeholder="请选择"
              clearable
              style="width: 200px"
            >
              <Option value="UPPER">上架</Option>
              <Option value="DOWN">下架</Option>
            </Select>
          </Form-item>
          <Form-item label="商品类型" prop="status">
            <Select v-model="searchForm.goodsType" placeholder="请选择" clearable style="width: 200px">
              <Option value="PHYSICAL_GOODS">实物商品</Option>
              <Option value="VIRTUAL_GOODS">虚拟商品</Option>
            </Select>
          </Form-item>
          <Button @click="handleSearch" class="search-btn" type="primary" icon="ios-search" >搜索</Button>
        </Form>
      </Row>
      <Table
        :loading="loading"
        border
        :columns="columns"
        :data="data"
        ref="table"
        sortable="custom"
        @on-sort-change="changeSort"
        @on-selection-change="changeSelect"
      >

        <!-- 商品栏目格式化 -->
        <template slot="goodsSlot" slot-scope="{row}">
          <div style="margin: 5px 0px;height: 80px; display: flex;">
            <div style="">
              <img :src="row.original" style="height: 60px;margin-top: 1px;width: 60px">
            </div>

            <div style="margin-left: 13px;">
              <div class="div-zoom">
                <a @click="linkTo(row.id,row.skuId)">{{row.goodsName}}</a>
              </div>
              <Poptip trigger="hover" title="扫码在手机中查看" transfer>
                <div slot="content">
                  <vue-qr :text="wapLinkTo(row.id,row.skuId)"  :margin="0" colorDark="#000" colorLight="#fff" :size="150"></vue-qr>
                </div>
                <img src="../../../assets/qrcode.svg" class="hover-pointer" width="20" height="20" alt="">
              </Poptip>
            </div>
          </div>

        </template>
      </Table>
      <Row type="flex" justify="end" class="page">
        <Page
          :current="searchForm.pageNumber"
          :total="total"
          :page-size="searchForm.pageSize"
          @on-change="changePage"
          @on-page-size-change="changePageSize"
          :page-size-opts="[10, 20, 50]"
          size="small"
          show-total
          show-elevator
          show-sizer
        ></Page>
      </Row>
    </Card>
    <Modal
      :title="modalTitle"
      v-model="modalVisible"
      :mask-closable="false"
      :width="500"
    >
      <Form
        ref="underForm"
        :model="underForm"
        :label-width="100"
      >
        <FormItem label="下架原因" prop="reason">
          <Input v-model="underForm.reason" clearable style="width: 100%" />
        </FormItem>
      </Form>
      <div slot="footer">
        <Button type="text" @click="modalVisible = false">取消</Button>
        <Button type="primary" :loading="submitLoading" @click="lower"
          >提交</Button
        >
      </div>
    </Modal>
  </div>
</template>

<script>
import { getGoodsListData, upGoods, lowGoods } from "@/api/goods";
export default {
  name: "goods",
  components: {},
  data() {
    return {
      id: "", //要操作的id
      loading: true, // 表单加载状态
      modalType: 0, // 添加或编辑标识
      modalVisible: false, // 添加或编辑显示
      modalTitle: "", // 添加或编辑标题
      searchForm: {
        // 搜索框初始化对象
        pageNumber: 1, // 当前页数
        pageSize: 10, // 页面大小
        sort: "create_time", // 默认排序字段
        order: "desc", // 默认排序方式
      },
      underForm: { // 下架原因
        reason: "",
      },
      submitLoading: false, // 添加或编辑提交状态
      selectList: [], // 多选数据
      selectCount: 0, // 多选计数
      columns: [
        {
          title: "商品名称",
          key: "goodsName",
          minWidth: 180,
          slot: "goodsSlot",
        },
        {
          title: "商品编号",
          key: "sn",
          minWidth: 150,
          tooltip: true
        },
        {
          title: "成本价",
          key: "price",
          width: 130,
          render: (h, params) => {
            return h(
              "div",
              this.$options.filters.unitPrice(params.row.cost, "￥")
            );
          },
        },
        {
          title: "价格",
          key: "price",
          width: 130,
          render: (h, params) => {
            return h(
              "div",
              this.$options.filters.unitPrice(params.row.price, "￥")
            );
          },
        },
        {
          title: "商品类型",
          key: "goodsType",
          width: 130,
          render: (h, params) => {
            if (params.row.goodsType === 'PHYSICAL_GOODS') {
              return h("Tag", {props: {color: "green",},}, "实物商品");
            } else if (params.row.goodsType === 'VIRTUAL_GOODS') {
              return h("Tag", {props: {color: "volcano",},}, "虚拟商品");
            } else {
              return h("Tag", {props: {color: "geekblue",},}, "电子卡券");
            }
          },
        },
        {
          title: "状态",
          key: "marketEnable",
          width: 100,
          render: (h, params) => {
            if (params.row.marketEnable == "DOWN") {
              return h("Tag", {props: {color: "volcano"},},"下架");
            } else if (params.row.marketEnable == "UPPER") {
              return h("Tag", {props: {color: "green",},},"上架");
            }
          },
        },
        {
          title: "审核状态",
          key: "isAuth",
          width: 130,
          render: (h, params) => {
            if (params.row.isAuth == "TOBEAUDITED") {
              return h("Tag", {props: {color: "volcano",},},"待审核");
            } else if (params.row.isAuth == "PASS") {
              return h("Tag", {props: {color: "green"},},"通过");
            } else if (params.row.isAuth == "REFUSE") {
              return h("Tag", {props: {color: "red",},},"拒绝");
            }
          },
        },
        {
          title: "店铺名称",
          key: "storeName",
          minWidth: 100,
          tooltip: true
        },
        {
          title: "操作",
          key: "action",
          align: "center",
          fixed: "right",
          width: 150,
          render: (h, params) => {
            if (params.row.marketEnable == "DOWN") {
              return h("div", [
                h(
                  "Button",
                  {
                    props: {
                      type: "info",
                      size: "small",
                    },
                    style: {
                      marginRight: "5px",
                    },
                    on: {
                      click: () => {
                        this.upper(params.row);
                      },
                    },
                  },
                  "上架"
                ), h(
                  "Button",
                  {
                    props: {
                      size: "small",
                    },
                    on: {
                      click: () => {
                        this.showDetail(params.row);
                      },
                    },
                  },
                  "查看"
                )
              ]);
            } else {
              return h("div", [
                h(
                  "Button",
                  {
                    props: {
                      type: "error",
                      size: "small",
                    },
                    style: {
                      marginRight: "5px",
                    },
                    on: {
                      click: () => {
                        this.edit(params.row);
                      },
                    },
                  },
                  "下架"
                ), h(
                  "Button",
                  {
                    props: {
                      size: "small",
                    },
                    on: {
                      click: () => {
                        this.showDetail(params.row);
                      },
                    },
                  },
                  "查看"
                )
              ]);
            }
          },
        },
      ],
      data: [], // 表单数据
      total: 0, // 表单数据总数
    };
  },
  methods: {
    init() {
      this.getDataList();
    },
    changePage(v) {
      this.searchForm.pageNumber = v;
      this.getDataList();
      this.clearSelectAll();
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
    changeSort(e) {
      this.searchForm.sort = e.key;
      this.searchForm.order = e.order;
      if (e.order === "normal") {
        this.searchForm.order = "";
      }
      this.getDataList();
    },
    clearSelectAll() {
      this.$refs.table.selectAll(false);
    },
    changeSelect(e) {
      this.selectList = e;
      this.selectCount = e.length;
    },
    getDataList() {
      this.loading = true;
      // 带多条件搜索参数获取表单数据
      getGoodsListData(this.searchForm).then((res) => {
        this.loading = false;
        if (res.records) {
          this.data = res.records;
          this.total = res.total;
        }
      });
    },
    edit(v) {
      this.id = v.id;
      if (v.underMessage != "{}") {
        this.underForm.reason = v.underMessage;
      }

      this.modalType = 1;
      this.modalTitle = "下架操作";
      this.modalVisible = true;
    },
    lower() {
      lowGoods(this.id, this.underForm).then((res) => {
        this.$Modal.remove();
        if (res.success) {
          this.$Message.success("操作成功");
          this.modalVisible = false;
          this.getDataList();
        }
      });
    },
    upper(v) {
      this.$Modal.confirm({
        title: "确认上架",
        content: "您确认要上架 " + v.goodsName + " ?",
        loading: true,
        onOk: () => {
          upGoods(v.id).then((res) => {
            this.$Modal.remove();
            if (res.success) {
              this.$Message.success("上架成功");
              this.getDataList();
            }
          });
        },
      });
    },

    //查看商品详情
    showDetail(v) {
      let id = v.id;
      this.$router.push({
        name: "goods-detail",
        query: { id: id },
      });
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
