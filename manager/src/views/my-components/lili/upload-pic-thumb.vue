<template>
  <div>
    <div class="upload-pic-thumb">
      <vuedraggable
        :list="uploadList"
        :disabled="!draggable||!multiple"
        :animation="200"
        class="list-group"
        ghost-class="thumb-ghost"
        @end="onEnd"
      >
        <div class="upload-list" v-for="(item, index) in uploadList" :key="index">
          <div v-if="item.status == 'finished'" style="height:60px;">
            <img :src="item.url" />
            <div class="upload-list-cover">
              <Icon type="ios-eye-outline" @click="handleView(item.url)"></Icon>
              <Icon type="ios-trash-outline" @click="handleRemove(item)"></Icon>
            </div>
          </div>
          <div v-else>
            <Progress v-if="item.showProgress" :percent="item.percentage" hide-info></Progress>
          </div>
        </div>
      </vuedraggable>
      <Upload
        ref="upload"
        :multiple="multiple"
        :show-upload-list="false"
        :on-success="handleSuccess"
        :on-error="handleError"
        :format="['jpg','jpeg','png','gif']"
        :max-size="maxSize*1024"
        :on-format-error="handleFormatError"
        :on-exceeded-size="handleMaxSize"
        :before-upload="handleBeforeUpload"
        type="drag"
        :action="uploadFileUrl"
        :headers="accessToken"
        style="display: inline-block;width:58px;"
      >
        <div style="width: 58px;height:58px;line-height: 58px;">
          <Icon type="md-camera" size="20"></Icon>
        </div>
      </Upload>
    </div>
    <Modal title="图片预览" v-model="viewImage" :styles="{top: '30px'}" draggable>
      <img :src="imgUrl" alt="无效的图片链接" style="width: 100%;margin: 0 auto;display: block;" />
      <div slot="footer">
        <Button @click="viewImage=false">关闭</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
import { uploadFile } from "@/api/index";
import vuedraggable from "vuedraggable";
export default {
  name: "uploadPicThumb",
  components: {
    vuedraggable
  },
  props: {
    value: {
      type: Object
    },
    draggable: {
      type: Boolean,
      default: true
    },
    multiple: {
      type: Boolean,
      default: true
    },
    maxSize: {
      type: Number,
      default: 5
    },
    limit: {
      type: Number,
      default: 10
    }
  },
  data() {
    return {
      accessToken: {}, // 验证token
      uploadFileUrl: uploadFile, // 上传地址
      uploadList: [], // 上传列表
      viewImage: false, // 预览modal
      imgUrl: "" // 图片地址
    };
  },
  methods: {
    onEnd() {
      this.returnValue();
    },
    init() {
      this.setData(this.value, true);
      this.accessToken = {
        accessToken: this.getStore("accessToken")
      };
    },
    handleView(imgUrl) {
      this.imgUrl = imgUrl;
      this.viewImage = true;
    },
    handleRemove(file) {
      this.uploadList = this.uploadList.filter(i => i.url !== file.url);
      this.returnValue();
    },
    handleSuccess(res, file) {
      if (res.success) {


        file.url = res.result;
        // 单张图片处理
        if (!this.multiple && this.uploadList.length > 0) {
          // 删除第一张
          this.uploadList.splice(0, 1);
        }
        this.uploadList.push(file);
        // 返回组件值
        this.returnValue();
      } else {
        this.$Message.error(res.message);
      }
    },
    handleError(error, file, fileList) {
      this.$Message.error(error.toString());
    },
    handleFormatError(file) {
      this.$Notice.warning({
        title: "不支持的文件格式",
        desc:
          "所选文件‘ " +
          file.name +
          " ’格式不正确, 请选择 .jpg .jpeg .png .gif图片格式文件"
      });
    },
    handleMaxSize(file) {
      this.$Notice.warning({
        title: "文件大小过大",
        desc:
          "所选文件‘ " +
          file.name +
          " ’大小过大, 不得超过 " +
          this.maxSize +
          "M."
      });
    },
    handleBeforeUpload() {
      if (this.multiple && this.uploadList.length >= this.limit) {
        this.$Message.warning("最多只能上传" + this.limit + "张图片");
        return false;
      }
      return true;
    },
    returnValue() {
      if (!this.uploadList || this.uploadList.length < 1) {
        if (!this.multiple) {
          this.$emit("input", "");
          this.$emit("on-change", "");
        } else {
          this.$emit("input", []);
          this.$emit("on-change", []);
        }
        return;
      }
      if (!this.multiple) {
        // 单张
        let v = this.uploadList[0].url;
        this.$emit("input", v);
        this.$emit("on-change", v);
      } else {
        let v = [];
        this.uploadList.forEach(e => {
          v.push(e.url);
        });
        this.$emit("input", v);
        this.$emit("on-change", v);
      }
    },
    setData(v, init) {
      if (typeof v == "string") {
        // 单张
        if (this.multiple) {
          this.$Message.warning("多张上传仅支持数组数据类型");
          return;
        }
        if (!v) {
          return;
        }
        this.uploadList = [];
        let item = {
          url: v,
          status: "finished"
        };
        this.uploadList.push(item);
        this.$emit("uploadchange", v);
        this.$emit("on-change", v);
      } else if (typeof v == "object") {
        // 多张
        if (!this.multiple) {
          this.$Message.warning("单张上传仅支持字符串数据类型");
          return;
        }
        this.uploadList = [];
        if (v.length > this.limit) {
          for (let i = 0; i < this.limit; i++) {
            let item = {
              url: v[i],
              status: "finished"
            };
            this.uploadList.push(item);
          }
          this.$emit("on-change", v.slice(0, this.limit));
          if (init) {
            this.$emit("input", v.slice(0, this.limit));
          }
          this.$Message.warning("最多只能上传" + this.limit + "张图片");
        } else {
          v.forEach(e => {
            let item = {
              status: "finished",
              ...e
            };
            this.uploadList.push(item);
          });
          this.$emit("on-change", v);
        }
      }
    }
  },
  watch: {
    value(val) {
      this.setData(val);
    }
  },
  mounted() {
    this.init();
  }
};
</script>

<style lang="scss" scoped>
.upload-pic-thumb{
  display: flex;
}
.upload-list {
  display: inline-block;
  width: 60px;
  height: 60px;
  text-align: center;
  line-height: 60px;
  border: 1px solid transparent;
  border-radius: 4px;
  overflow: hidden;
  background: #fff;
  position: relative;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
  margin-right: 5px;
  vertical-align: middle;
}
.upload-list img {
  width: 100%;
  height: -webkit-fill-available;
}
.upload-list-cover {
  display: none;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.6);
}
.upload-list:hover .upload-list-cover {
  display: block;
}
.upload-list-cover i {
  color: #fff;
  font-size: 20px;
  cursor: pointer;
  margin: 0 2px;
}
.list-group {
  display: inline-block;
}
.thumb-ghost {
  opacity: 0.5;
  background: #c8ebfb;
}
</style>
