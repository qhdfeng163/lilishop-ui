<template>
  <div class="search">
    <Card>
      <Row @keydown.enter.native="handleSearch">
        <Form ref="searchForm" :model="searchForm" inline :label-width="70" class="search-form">
          <Form-item label="会员名称" prop="username">
            <Input type="text" v-model="searchForm.username" placeholder="请输入会员名称" clearable style="width: 200px" />
          </Form-item>

          <Form-item label="会员昵称" prop="nickName">
            <Input type="text" v-model="searchForm.nickName" placeholder="请输入会员昵称" clearable style="width: 200px" />
          </Form-item>

          <Form-item label="联系方式" prop="mobile">
            <Input type="text" v-model="searchForm.mobile" placeholder="请输入会员联系方式" clearable style="width: 200px" />
          </Form-item>
          <Button @click="handleSearch" class="search-btn" type="primary" icon="ios-search">搜索</Button>
        </Form>
      </Row>
      <Row class="operation padding-row">
        <Button @click="addMember" type="primary">添加会员</Button>
      </Row>

      <Table :loading="loading" border :columns="columns" :data="data" ref="table" sortable="custom" @on-sort-change="changeSort" @on-selection-change="changeSelect">
      </Table>
      <Row type="flex" justify="end" class="page">
        <Page :current="searchForm.pageNumber" :total="total" :page-size="searchForm.pageSize" @on-change="changePage" @on-page-size-change="changePageSize" :page-size-opts="[10, 20, 50]" size="small"
          show-total show-elevator show-sizer></Page>
      </Row>
    </Card>

    <!-- 添加用户模态框 -->
    <Modal v-model="addFlag" title="添加用户">
      <Form ref="addMemberForm" :model="addMemberForm" :rules="addRule" :label-width="100">
        <FormItem label="手机号码" prop="mobile" style="width: 90%;">
          <Input v-model="addMemberForm.mobile" maxlength="11" placeholder="请输入手机号码" />
        </FormItem>
        <FormItem label="会员名称" prop="uname" style="width: 90%">
          <Input v-model="addMemberForm.username" maxlength="15" placeholder="请输入会员名称" />
        </FormItem>

        <FormItem label="会员密码" prop="pwd" style="width: 90%">
          <Input type="password" password v-model="addMemberForm.password" maxlength="20" placeholder="请输入会员密码" />
        </FormItem>
      </Form>

      <div slot="footer">
        <Button @click="
            () => {
              addFlag = false;
            }
          ">
          取消
        </Button>
        <Button type="primary" :loading="handleAddLoading" @click="addMemberSubmit">
          确定
        </Button>
      </div>
    </Modal>

    <!-- 修改模态框 -->
    <Modal v-model="descFlag" :title="descTitle" @on-ok="handleSubmitModal" width="500">
      <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="80">
        <FormItem label="头像">
          <img :src="formValidate.face" class="face" />
          <Button type="text" class="upload" @click="
              () => {
                this.picModelFlag = true;
                this.$refs.ossManage.selectImage = true;
              }
            ">修改
          </Button>
          <input type="file" style="display: none" id="file" />
        </FormItem>
        <FormItem label="用户名" prop="name">
          <Input v-model="formValidate.username" style="width: 200px" disabled />
        </FormItem>
        <FormItem label="用户昵称" prop="name">
          <Input v-model="formValidate.nickName" style="width: 200px" />
        </FormItem>
        <FormItem label="性别" prop="sex">
          <RadioGroup type="button" button-style="solid" v-model="formValidate.sex">
            <Radio :label="1">
              <span>男</span>
            </Radio>
            <Radio :label="0">
              <span>女</span>
            </Radio>
          </RadioGroup>
        </FormItem>
        <FormItem label="修改密码" prop="password">
          <Input type="password" style="width: 220px" password v-model="formValidate.newPassword" />
        </FormItem>
        <FormItem label="生日" prop="birthday">
          <DatePicker type="date" format="yyyy-MM-dd" v-model="formValidate.birthday" style="width: 220px"></DatePicker>
        </FormItem>
        <FormItem label="所在地" prop="mail">
          <div class="form-item" v-if="!updateRegion">
            <Input disabled style="width: 250px" :value="formValidate.region" />
            <Button type="text" @click="
                () => {
                  this.updateRegion = !this.updateRegion;
                }
              ">修改
            </Button>
          </div>
          <div class="form-item" v-else>
            <region style="width: 250px" @selected="selectedRegion" />
          </div>
        </FormItem>
      </Form>
    </Modal>
    <Modal width="1200px" v-model="picModelFlag">
      <ossManage @callback="callbackSelected" ref="ossManage" />
    </Modal>
  </div>
</template>

<script>
import region from "@/views/lili-components/region";
import * as API_Member from "@/api/member.js";
import ossManage from "@/views/sys/oss-manage/ossManage";
import * as RegExp from "@/libs/RegExp.js";

export default {
  name: "member",
  components: {
    region,
    ossManage,
  },
  data() {
    return {
      selectedMember: false, //是否是其他组件调用
      descTitle: "", // modal标题
      descFlag: false, //编辑查看框
      openSearch: true, // 显示搜索
      loading: true, // 表单加载状态
      addFlag: false, // modal显隐控制
      updateRegion: false, // 地区
      addMemberForm: {
        // 添加用户表单
        mobile: "",
        username: "",
        password: "",
      },
      searchForm: {
        // 请求参数
        pageNumber: 1,
        pageSize: 10,
        order: "desc",
        username: "",
        mobile: "",
        disabled: "OPEN",
      },
      picModelFlag: false, // 选择图片
      formValidate: {}, // 表单数据
      addRule: {
        // 验证规则
        mobile: [
          { required: true, message: "请输入手机号码" },
          {
            pattern: RegExp.mobile,
            message: "请输入正确的手机号",
          },
        ],
        uname: [{ required: true, message: "请输入会员名称" }],
        pwd: [{ required: true, message: "请输入密码" }],
      },
      ruleValidate: {}, //修改验证
      submitLoading: false, // 添加或编辑提交状态
      selectList: [], // 多选数据
      selectCount: 0, // 多选计数
      columns: [
        {
          title: "会员名称",
          key: "username",
          tooltip: true,
        },
        {
          title: "会员昵称",
          key: "nickName",
          tooltip: true,
        },
        {
          title: "联系方式",
          width: 130,
          key: "mobile",
          render: (h, params) => {
            if (params.row.mobile == null) {
              return h("div", [h("span", {}, "")]);
            } else {
              return h("div", [h("span", {}, params.row.mobile)]);
            }
          },
        },
        {
          title: "注册时间",
          key: "createTime",
          width: 180,
        },

        {
          title: "积分数量",
          align: "left",
          width: 100,
          render: (h, params) => {
            return h(
              "div",
              {},
              params.row.point == void 0 ? "0" : params.row.point
            );
          },
        },
        {
          title: "操作",
          key: "action",
          align: "center",
          width: 200,
          fixed: "right",
          render: (h, params) => {
            return h(
              "div",
              {
                style: {
                  display: "flex",
                  justifyContent: "center",
                },
              },
              [
                h(
                  "Button",
                  {
                    props: {
                      size: "small",
                      type: params.row.___selected ? "primary" : "",
                    },
                    style: {
                      marginRight: "5px",
                      display: this.selectedMember ? "block" : "none",
                    },
                    on: {
                      click: () => {
                        this.callback(params.row, params.index);
                      },
                    },
                  },
                  params.row.___selected ? "已选择" : "选择"
                ),

                h(
                  "Button",
                  {
                    props: {
                      type: "info",
                      size: "small",
                    },
                    style: {
                      marginRight: "5px",
                      display: this.selectedMember ? "none" : "block",
                    },
                    on: {
                      click: () => {
                        this.detail(params.row);
                      },
                    },
                  },
                  "查看"
                ),
                h(
                  "Button",
                  {
                    props: {
                      type: "info",
                      size: "small",
                      ghost: true,
                    },
                    style: {
                      marginRight: "5px",
                      display: this.selectedMember ? "none" : "block",
                    },
                    on: {
                      click: () => {
                        this.editPerm(params.row);
                      },
                    },
                  },
                  "编辑"
                ),
                h(
                  "Button",
                  {
                    props: {
                      size: "small",
                      type: "error",
                    },
                    style: {
                      marginRight: "5px",
                      display: this.selectedMember ? "none" : "block",
                    },
                    on: {
                      click: () => {
                        this.disabled(params.row);
                      },
                    },
                  },
                  "禁用"
                ),
              ]
            );
          },
        },
      ],
      data: [], // 表单数据
      total: 0, // 表单数据总数
      selectMember: [], //保存选中的用户
    };
  },
  props: {
    // 是否为选中模式
    selectedMember: {
      type: Boolean,
      default: false,
    },
    // 已选择用户数据
    selectedList: {
      type: null,
      default: [],
    },
  },
  watch: {
    selectedList: {
      handler(val) {
        this.$set(this, "selectMember", JSON.parse(JSON.stringify(val)));
        this.init(this.data);
        // 将父级数据与当前组件数据进行匹配

      },
      deep: true,
      immediate: true,
    },
  },
  methods: {
    // 回调给父级
    callback(val, index) {
      this.$set(val, "___selected", !val.___selected);
      console.log(val.___selected);
      console.log(this.selectMember);
      let findUser = this.selectMember.find((item) => {
        return item.id == val.id;
      });
      // 如果没有则添加
      if (!findUser) {
        this.selectMember.push(val);
      } else {
        // 有重复数据就删除
        this.selectMember.map((item, index) => {
          if (item.id == findUser.id) {
            this.selectMember.splice(index, 1);
          }
        });
      }

      this.$emit("callback", val);
    },
    // 初始化信息
    init(data) {
      data.forEach((item) => {
        if (this.selectMember.length != 0) {
          this.selectMember.forEach((member) => {
            if (member.id == item.id) {
              this.$set(item, "___selected", true);
            }
          });
        } else {
          this.$set(item, "___selected", false);
        }
      });
      this.data = data;
    },
    changePage(v) {
      this.searchForm.pageNumber = v;
      // 此处如果是父子级传值的时候需要做一下处理
      //selectedMember

      this.getData();
    },
    changePageSize(v) {
      this.searchForm.pageSize = v;
      this.getData();
    },
    handleSearch() {
      this.searchForm.pageNumber = 1;
      this.searchForm.pageSize = 10;
      this.getData();
    },

    changeSort(e) {
      this.searchForm.sort = e.key;
      this.searchForm.order = e.order;
      if (e.order === "normal") {
        this.searchForm.order = "";
      }
      this.getData();
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
    //查看详情修改
    editPerm(val) {
      this.descTitle = `查看用户 ${val.username}`;
      this.descFlag = true;
      this.updateRegion = false;
      this.getMemberInfo(val.id);
    },
    addMember() {
      this.addFlag = true;
      this.addMemberForm = {
        mobile: "",
        username: "",
        password: "",
      };
    },
    /**
     * 查询查看会员详情
     */
    getMemberInfo(id) {
      API_Member.getMemberInfoData(id).then((res) => {
        if (res.result) {
          this.$set(this, "formValidate", res.result);
        }
      });
    },

    //查询会员列表
    getData() {
      API_Member.getMemberListData(this.searchForm).then((res) => {
        if (res.result.records) {
          this.loading = false;
          this.init(res.result.records);
          this.total = res.result.total;
        }
      });
    },
    // 选中的图片
    callbackSelected(val) {
      this.picModelFlag = false;
      this.formValidate.face = val.url;
    },
    //添加会员提交
    addMemberSubmit() {
      this.addMemberForm.password = this.md5(this.addMemberForm.password);
      this.$refs.addMemberForm.validate((valid) => {
        if (valid) {
          API_Member.addMember(this.addMemberForm).then((res) => {
            if (res.result) {
              this.getData();
              this.$Message.success("添加成功！");
              this.addFlag = false;
            }
          });
        }
      });
    },

    // 选中的地址
    selectedRegion(val) {
      this.region = val[1];
      this.regionId = val[0];
    },
    //查看会员
    detail(row) {
      this.$router.push({ name: "member-detail", query: { id: row.id } });
    },

    //禁用会员
    disabled(v) {
      let params = {
        memberIds: [v.id],
        disabled: false,
      };
      this.$Modal.confirm({
        title: "提示",
        content: "<p>确认禁用此会员？</p>",
        onOk: () => {
          API_Member.updateMemberStatus(params).then((res) => {
            if (res.success) {
              this.$Message.success("禁用成功");
              this.getData();
            } else {
              this.$Message.error(res.message);
            }
          });
        },
      });
    },

    // 提交修改数据
    handleSubmitModal() {
      const { nickName, sex, username, face, newPassword } = this.formValidate;
      let time = new Date(this.formValidate.birthday);
      let birthday =
        time.getFullYear() + "-" + (time.getMonth() + 1) + "-" + time.getDate();
      let submit = {
        regionId: this.formValidate.regionId,
        region: this.formValidate.region,
        nickName,
        username,
        sex,
        birthday,
        face,
      };
      if (this.region != "undefined") {
        submit.regionId = this.regionId;
        submit.region = this.region;
      }
      if (newPassword) {
        submit.password = this.md5(newPassword);
      }
      API_Member.updateMember(submit).then((res) => {
        if (res.result) {
          this.$Message.success("修改成功！");
          this.init();
        }
      });
    },
  },
  mounted() {
    this.getData();
  },
};
</script>
<style lang="scss" scoped>
@import "@/styles/table-common.scss";
/deep/ .ivu-table-wrapper {
  width: 100%;
}
/deep/ .ivu-card {
  width: 100%;
}
.face {
  width: 60px;
  height: 60px;
  border-radius: 50%;
}
</style>
