<template>
  <div class="search">
    <Card>
      <Row @keydown.enter.native="handleSearch" >
        <Form ref="searchForm" :model="searchForm" inline :label-width="70" class="search-form">
          <Input class="search-input" v-model="searchForm.memberName">
            <span slot="prepend">会员名称</span>
          </Input>
          <Input class="search-input" v-model="searchForm.sn">
            <span slot="prepend">编号</span>
          </Input>
          <Form-item label="状态" style="margin-left: -20px">
            <Select v-model="searchForm.distributionCashStatus" style="width:150px;">
                <Option v-for="item in cashStatusList" :value="item.value" :key="item.value">{{ item.label }}</Option>
            </Select>
          </Form-item>
          <Form-item style="margin-left:-35px;" class="br">
            <Button @click="handleSearch" type="primary" icon="ios-search">搜索</Button>
          </Form-item>
        </Form>
      </Row>
      <Table :loading="loading" border :columns="columns" :data="data" ref="table" sortable="custom" @on-sort-change="changeSort" @on-selection-change="changeSelect"></Table>
      <Row type="flex" justify="end" class="page padding-row">
        <Page :current="searchForm.pageNumber" :total="total" :page-size="searchForm.pageSize" @on-change="changePage" @on-page-size-change="changePageSize" :page-size-opts="[10,20,50]" size="small" show-total show-elevator show-sizer></Page>
      </Row>
    </Card>
    <Modal :title="modalTitle" v-model="modalVisible" :mask-closable='false' :width="500">
      <Form ref="form" :model="form" :label-width="100" :rules="formValidate" >
      <FormItem label="编号">
          <Input disabled v-model="form.sn" clearable style="width:100%"/>
        </FormItem>
        <FormItem label="会员名称">
          <Input disabled v-model="form.distributionName" clearable style="width:100%"/>
        </FormItem>
        <FormItem label="金额">
          <Input disabled v-model="form.price" clearable style="width:100%"/>
        </FormItem>
        <FormItem label="是否通过" prop="result" v-if="handleStatus =='edit'">
             <RadioGroup v-model="result" type="button" button-style="solid">
                 <Radio :key=0 :label=0>通过</Radio>
                 <Radio :key=-1 :label=-1>拒绝</Radio>
             </RadioGroup>
        </FormItem>
      </Form>
      <div slot="footer" v-if="handleStatus == 'edit'">
        <Button type="text" @click="modalVisible=false">取消</Button>
        <Button type="primary" :loading="submitLoading" @click="handleSubmit">提交</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
import {
        getDistributionCash,
        auditDistributionCash
    } from "@/api/distribution";
// import { parse } from 'date-fns';
import {cashStatusList} from './dataJson'
export default {
  name: "distributionCash",
  components: {
  },
  data() {
    return {
      cashStatusList, // 状态列表
      loading: true, // 表单加载状态
      modalVisible: false, // 添加或编辑显示
      modalTitle: "", // 添加或编辑标题
      result: -1, // 是否通过
      searchForm: { // 搜索框初始化对象
        pageNumber: 1, // 当前页数
        pageSize: 10, // 页面大小
        sort: "createTime", // 默认排序字段
        order: "desc", // 默认排序方式
      },
      handleStatus:'edit',// 判断是编辑还是查看
      form: { // 添加或编辑表单对象初始化数据
        sn: "",
        memberName: "",
        price: "",
      },
      auditForm: { // 编辑表单
        result: -1
      },
      submitLoading: false, // 添加或编辑提交状态
      selectList: [], // 多选数据
      selectCount: 0, // 多选计数
      columns: [
        {
          title: "编号",
          key: "sn",
          minWidth: 200
        },
        {
          title: "会员名称",
          key: "distributionName",
          minWidth: 120
        },
        {
          title: "申请金额",
          key: "price",
          minWidth: 90,
           render: (h, params) => {
              if (params.row.price) {
                  return h("div", this.$options.filters.unitPrice(params.row.price,'￥'));
                }
              }
        },
        {
          title: "申请时间",
          key: "createTime",
          minWidth: 130
        },
        {
          title: "处理时间",
          key: "payTime",
          minWidth: 130
        },
        {
          title: "状态",
          key: "distributionCashStatus",
          minWidth: 100,
          render: (h, params) => {
              if (params.row.distributionCashStatus == 'APPLY') {
                   return h("div", "待处理");
              }
              if (params.row.distributionCashStatus == 'PASS') {
                   return h("div", "通过");
              }
              if (params.row.distributionCashStatus == 'REFUSE') {
                   return h("div", "审核拒绝");
              }
            },
         },
        {
          title: "操作",
          key: "action",
          align: "center",
          fixed: "right",
          width: 130,
          render: (h, params) => {
            if(params.row.distributionCashStatus != 'APPLY'){
                return h("div", [
                  h(
                    "Button",
                    {
                      props: {
                        type: "success",
                        size: "small",
                      },
                      style: {
                        marginRight: "5px"
                      },
                      on: {
                        click: () => {
                          this.view(params.row);
                        }
                      }
                    },
                    "查看"
                  ),

                ]);
            }else {
                return h("div", [
                  h(
                    "Button",
                    {
                      props: {
                        type: "primary",
                        size: "small",
                      },
                      style: {
                        marginRight: "5px"
                      },
                      on: {
                        click: () => {
                          this.edit(params.row);
                        }
                      }
                    },
                    "审核"
                  ),

                ]);
            }
          }
        }
      ],
      data: [], // 表单数据
      total: 0 // 表单数据总数
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
      // 带多条件搜索参数获取表单数据 请自行修改接口
      getDistributionCash(this.searchForm).then(res => {
        this.loading = false;
        if (res.success) {
          this.data = res.result.records;
          this.total = res.result.total;
        }
      });
      this.total = this.data.length;
      this.loading = false;
    },
    handleSubmit() {
      let result = "拒绝"
      if(this.result == 0){
        result = "通过"
      }
      this.$refs.form.validate(valid => {
        if (valid) {
          this.$Modal.confirm({
            title: "确认审核",
            content: "您确认要审核"+result+"么?",
            loading: true,
            onOk: () => {
                this.auditForm.result = this.result;
                auditDistributionCash(this.form.id,this.auditForm).then(res => {
                    if (res.success) {
                      this.$Modal.remove();
                      this.$Message.success("审核成功");
                      this.getDataList();
                      this.modalVisible = false;
                    }
                });
            }
          })
        }
      });
    },
    edit(v) {
      this.modalTitle = "审核";
      this.handleStatus = 'edit';
      this.$refs.form.resetFields();
      // 转换null为""
      for (let attr in v) {
        if (v[attr] === null) {
          v[attr] = "";
        }
      }
      let str = JSON.stringify(v);
      let data = JSON.parse(str);
      this.form = data;
      this.modalVisible = true;
    },
    view(v){
      this.modalTitle = "查看";
      this.handleStatus = 'view';
      this.$refs.form.resetFields();
      // 转换null为""
      for (let attr in v) {
        if (v[attr] === null) {
          v[attr] = "";
        }
      }
      let str = JSON.stringify(v);
      let data = JSON.parse(str);
      this.form = data;
      this.modalVisible = true;

    }
  },
  mounted() {
    this.init();
  }
};
</script>
<style lang="scss">
  @import "@/styles/table-common.scss";
</style>
