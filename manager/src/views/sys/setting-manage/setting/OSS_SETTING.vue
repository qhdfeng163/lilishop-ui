<template>
  <div class="layout">
    <Form ref="formValidate" :label-width="150" label-position="right" :model="formValidate" :rules="ruleValidate">
      <FormItem label="endPoint" prop="endPoint">
        <Input v-model="formValidate.endPoint" />
      </FormItem>
      <FormItem label="bucketName" class="label-item" prop="bucketName">
        <Input v-model="formValidate.bucketName" />
      </FormItem>
      <FormItem label="picLocation" prop="bucketName">
        <Input v-model="formValidate.picLocation" />
      </FormItem>
      <FormItem label="accessKeyId" prop="accessKeyId">
        <Input v-model="formValidate.accessKeyId" />
      </FormItem>
      <FormItem label="accessKeySecret" prop="accessKeySecret">
        <Input v-model="formValidate.accessKeySecret" />
      </FormItem>
      <div class="label-btns">
        <Button type="primary" @click="submit('formValidate')">保存</Button>

      </div>
    </Form>
  </div>
</template>
<script>
import { setSetting } from "@/api/index";
import { handleSubmit } from "./validate";
export default {
  data() {
    return {
      ruleValidate: {}, // 验证规则
      formValidate: { // 表单数据
        accessKeyId: "",
        accessKeySecret: "",
        bucketName: "",
        picLocation: "",
        endPoint: "",
      },
    };
  },
  props: ["res", "type"],
  created() {
    this.init();
  },
  methods: {
    submit(name) {
      let that = this;
       if( handleSubmit(that, name )){
        this.setupSetting()
      }
    },

    setupSetting() {
      setSetting(this.type, this.formValidate).then((res) => {
        if (res.success) {
          this.$Message.success("保存成功!");
        } else {
          this.$Message.error("保存失败!");
        }
      });
    },
    // 实例化数据
    init() {
      this.res = JSON.parse(this.res);

      this.$set(this, "formValidate", { ...this.res });
      Object.keys(this.formValidate).forEach((item) => {
        this.ruleValidate[item] = [
          {
            required: true,
            message: "请填写必填项",
            trigger: "blur",
          },
        ];
      });
    },
  },
};
</script>

<style lang="scss" scoped>
@import "./style.scss";
.label-item {
  display: flex;
}
/deep/ .ivu-input {
  width: 300px !important;
  margin: 0 10px;
}
.ivu-input-wrapper {
  width: 300px;
  margin-right: 10px;
}
</style>
