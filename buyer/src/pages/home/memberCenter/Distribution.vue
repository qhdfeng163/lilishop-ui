<template>
    <div class='wrapper'>
      <card _Title='我的分销'/>
      <!-- 分销申请 -->
      <div v-if="status === 0">
        <Alert type="warning">分销商申请</Alert>
        <Form ref="form" :model="applyForm" :rules="rules">
          <FormItem label="姓名" prop="name">
            <Input v-model="applyForm.name"></Input>
          </FormItem>
          <FormItem label="身份证号" prop="idNumber">
            <Input v-model="applyForm.idNumber"></Input>
          </FormItem>
          <FormItem>
            <Button type="primary" :loading="applyLoading" @click="apply">提交申请</Button>
          </FormItem>
          </Form>
      </div>
      <!-- 分销审核 -->
      <div v-if="status === 1">
        <Alert type="success">
          您提交的信息正在审核
          <template slot="desc">提交认证申请后，工作人员将在三个工作日进行核对完成审核</template>
        </Alert>
      </div>
      <!-- 分销提现、商品、订单 -->
      <div v-if="status === 2">
        <div class="box">
          <div class="mb_20 account-price">
            <span class="subTips">可提现金额：</span>
            <span class="fontsize_48 global_color">￥{{ result.canRebate | unitPrice }}</span>
            <span class="subTips">冻结金额：</span>
            <span class="">￥{{ result.commissionFrozen | unitPrice }}</span>
            <Button type="primary" size="small" class="ml_20" @click="withdrawApplyModal = true">申请提现</Button>
          </div>
        </div>
        <Tabs :value="tabName" @on-click="tabPaneChange">
          <TabPane label="已选商品" name="goodsChecked">
            <Table stripe :columns="goodsColumns" :data="goodsData.records">
              <template slot-scope="{ row }" slot="name">
                <div class="goods-msg" @click="linkTo(`/goodsDetail?skuId=${row.skuId}&goodsId=${row.id}`)"><img style="vertical-align:top;" :src="row.thumbnail" width="60" height="60" alt="">&nbsp; {{row.goodsName}}</div>
              </template>
              <template slot-scope="{ row }" slot="price">
                <span> ￥{{ row.price | unitPrice }}</span>
              </template>
              <template slot-scope="{ row }" slot="commission">
                <span> ￥{{ row.commission | unitPrice }}</span>
              </template>
              <template slot-scope="{ row }" slot="action">
                <Button type="primary" size="small" style="margin-right: 5px" @click="fenxiao(row)">分销商品</Button>
              </template>
            </Table>
            <div class="page-size">
              <Page
                :current="params.pageNumber"
                :total="goodsData.total"
                :page-size="params.pageSize"
                @on-change="changePage"
                size="small"
                show-total
                show-elevator
              ></Page>
            </div>
          </TabPane>
          <TabPane label="未选商品" name="goodsUncheck">
            <Table stripe :columns="goodsColumns" :data="goodsData.records">
              <template slot-scope="{ row }" slot="name">
                <div class="goods-msg" @click="linkTo(`/goodsDetail?skuId=${row.skuId}&goodsId=${row.id}`)"><img style="vertical-align:top;" :src="row.thumbnail" width="60" height="60" alt="">&nbsp; {{row.goodsName}}</div>
              </template>
              <template slot-scope="{ row }" slot="price">
                <span> ￥{{ row.price | unitPrice }}</span>
              </template>
              <template slot-scope="{ row }" slot="commission">
                <span> ￥{{ row.commission | unitPrice }}</span>
              </template>
              <template slot-scope="{ row }" slot="action">
                <Button type="primary" size="small" style="margin-right: 5px" @click="selectGoods(row.id)">选择商品</Button>
              </template>
            </Table>
            <div class="page-size">
              <Page
                :current="params.pageNumber"
                :total="goodsData.total"
                :page-size="params.pageSize"
                @on-change="changePage"
                size="small"
                show-total
                show-elevator
              ></Page>
            </div>
          </TabPane>
          <TabPane label="提现记录" name="log">
            <Table stripe :columns="logColumns" :data="logData.records">
              <template slot-scope="{ row }" slot="time">
                <span>{{row.createTime}}</span>
              </template>
              <template slot-scope="{ row }" slot="price">
                <span> +￥{{ row.price | unitPrice }}</span>
              </template>
              <template slot-scope="{ row }" slot="status">
                <span> {{row.distributionCashStatus == "APPLY" ? "待处理" : row.distributionCashStatus == "PASS" ? "通过" : "拒绝"}}</span>
              </template>
            </Table>
            <div class="page-size">
              <Page
                :current="logParams.pageNumber"
                :total="logData.total"
                :page-size="logParams.pageSize"
                @on-change="changePageLog"
                size="small"
                show-total
                show-elevator
              ></Page>
            </div>
          </TabPane>
        </Tabs>
      </div>
      <!-- 未开放 -->
      <div v-if="status === 3">
        <Alert type="error">
          分销功能暂未开启
          <template slot="desc">提交认证申请后，工作人员将在三个工作日进行核对完成审核</template>
        </Alert>
      </div>

      <Modal v-model="withdrawApplyModal" width="530">
        <p slot="header">
          <Icon type="edit"></Icon>
          <span>提现金额</span>
        </p>
        <div>
          <Input
            v-model="withdrawPrice"
            size="large"
            number
            maxlength="9"
            ><span slot="append">元</span></Input>
        </div>
        <div slot="footer" style="text-align: center">
          <Button type="primary" size="large" @click="withdraw">提现</Button>
        </div>
      </Modal>
      <Modal v-model="qrcodeShow" title="分销商品" width="800">
        <Alert type="warning">
          下载二维码或者复制链接分享商品
        </Alert>
        <div style="width:200px;height:200px;border:1px solid #eee;">
          <vue-qr :text="qrcode" :callback="qrcodeData" :margin="0" colorDark="#000" colorLight="#fff" :size="200"></vue-qr>
          <Button class="download-btn" type="success" @click="downloadQrcode">下载二维码</Button>
        </div>
        <div class="mt_10">商品链接：<Input style="width:400px" v-model="qrcode"></Input></div>
      </Modal>
    </div>
</template>

<script>
import {distribution, applyDistribution, distCash, distCashHistory, getDistOrderList, getDistGoodsList, selectDistGoods} from '@/api/member.js'
import { IDCard } from '@/plugins/RegExp.js';
import vueQr from 'vue-qr';
export default {
  name: 'Distribution',
  components: { vueQr },
  data () {
    return {
      status: 0, // 申请状态，0为未申请 1 申请中 2 申请完成 3 功能暂未开启
      applyForm: {}, // 申请表单
      rules: { // 验证规则
        name: [{required: true, message: '请输入真实姓名'}],
        idNumber: [
          {required: true, message: '请输入身份证号'},
          {pattern: IDCard, message: '请输入正确的身份证号'}
        ]
      },
      tabName: 'goodsChecked', // 当前所在tab
      result: {}, // 审核结果
      applyLoading: false, // 申请加载状态
      goodsLoading: false, // 列表加载状态
      withdrawApplyModal: false, // 提现表单显隐
      withdrawPrice: 0, // 提现金额
      goodsData: {}, // 商品数据
      logData: {}, // 日志数据
      goodsColumns: [ // 商品表头
        {title: '商品名称', slot: 'name', width: 400},
        {title: '商品价格', slot: 'price'},
        {title: '佣金', slot: 'commission'},
        {title: '操作', slot: 'action', width: 120}
      ],
      logColumns: [ // 日志表头
        {title: '申请时间', slot: 'time'},
        {title: '提现金额', slot: 'price'},
        {title: '提现状态', slot: 'status'}
      ],
      params: { // 商品请求参数
        pageNumber: 1,
        pageSize: 20,
        checked: true
      },
      orderParams: { // 订单商品请求参数
        pageNumber: 1,
        pageSize: 20
      },
      logParams: { // 日志参数
        pageNumber: 1,
        pageSize: 20
      },
      qrcode: '', // 二维码
      qrcodeShow: false, // 显示二维码
      base64Img: '', // base64编码
      goodsNameCurr: ''// 当前分销商品名称
    }
  },
  mounted () {
    this.distribution()
  },
  methods: {
    apply () { // 申请成为分销商
      this.$refs.form.validate(valid => {
        if (valid) {
          this.applyLoading = true
          applyDistribution(this.form).then(res => {
            this.applyLoading = false
            if (res.success) {
              this.$Message.success('申请已提交，请等待管理员审核')
              this.status = 1;
            }
          })
        }
      })
    },
    withdraw () { // 申请提现
      distCash({price: this.withdrawPrice}).then(res => {
        this.withdrawApplyModal = false
        this.price = 0;
        if (res.success) {
          this.$Message.success('提现成功')
          this.distribution()
        } else {
          this.$Message.error(res.message)
        }
      })
    },
    qrcodeData (data64) { // 二维码base64地址
      this.base64Img = data64
    },
    downloadQrcode () { // 下载二维码
      let a = document.createElement('a'); // 生成一个a元素
      let event = new MouseEvent('click'); // 创建一个单击事件
      a.download = this.goodsNameCurr || 'photo'
      a.href = this.base64Img; // 将生成的URL设置为a.href属性
      a.dispatchEvent(event); // 触发a的单击事件
    },
    tabPaneChange  (tab) { // tab栏切换
      if (tab === 'goodsChecked') {
        this.params.checked = true;
        this.params.pageNUmber = 1
        this.getGoodsData()
      } else if (tab === 'goodsUncheck') {
        this.params.checked = false
        this.getGoodsData()
      } else if (tab === 'order') {
        this.orderParams.pageNumber = 1
        this.getOrderData()
      } else if (tab === 'log') {
        this.logParams.pageNumber = 1
        this.getLog()
      }
    },
    changePage (val) { // 修改页码
      this.params.pageNumber = val;
      this.getGoodsData()
    },
    changePageLog (val) { // 修改页码 日志
      this.logParams.pageNumber = val;
      this.getLog()
    },
    selectGoods (id) { // 选择商品
      selectDistGoods(id).then(res => {
        if (res.success) {
          this.$Message.success('分销商品选择成功，请在已选商品中查看')
          this.getGoodsData()
        }
      })
    },
    fenxiao (row) { // 分销商品
      this.qrcode = `${location.origin}/goodsDetail?skuId=${row.skuId}&goodsId=${row.id}&distributionId=${this.result.id}`
      this.goodsNameCurr = row.goodsName
      this.qrcodeShow = true;
    },
    getGoodsData () { // 商品数据
      getDistGoodsList(this.params).then(res => {
        if (res.success) this.goodsData = res.result
      })
    },
    getOrderData () { // 订单数据
      getDistOrderList(this.orderParams).then(res => {
        if (res.success) this.goodsData = res.result
      })
    },
    getLog () { // 提现历史
      distCashHistory(this.logParams).then(res => {
        if (res.success) this.goodsData = res.result
      })
    },
    distribution () { // 获取分销商信息
      distribution().then((res) => {
        if (res.result) {
          this.result = res.result;
          let type = res.result.distributionStatus;

          if (type === 'PASS') {
            this.status = 2;
            this.getGoodsData()
          } else if (type === 'RETREAT' || type === 'REFUSE') {
            this.status = 0;
          } else {
            this.status = 1;
          }
        } else if (!res.data.success && res.data.code === 22000) {
          this.status = 3;
        } else {
          // 没有资格申请 先去实名认证
          this.status = 0
        }
      });
    }
  }
}
</script>

<style scoped lang='scss'>
  .afList{
    display: flex;

  }
.codeImg{

  display: flex;
  >img{
    width: 150px;
    height: 150px;
  }
}
.mb_40{margin-bottom: 40px;}
.box {
  margin: 20px 0;
}
.page-size {
  margin: 15px 0px;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}
.account-price {
  font-weight: bold;
}
.subTips {
  @include sub_color($light_sub_color);
}
.account-btns {
  margin: 10px 0;
}
.ivu-btn {
  margin: 0 4px;
}
.fontsize_48 {
  font-size: 48px;
}
.goods-msg{
  display: flex;
  align-items: center;
  padding: 3px;
  &:hover{
    color: $theme_color;
    cursor: pointer;
  }
}
.download-btn {
  position: relative;
  top: -200px;
  left: 200px;
}
</style>
