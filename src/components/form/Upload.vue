<template>
  <div>
    <el-upload v-if="multiple"
               :action="baseUrl + url"
               list-type="picture"
               :on-success="handleUpload"
               :on-preview="handlePictureCardPreview"
               :on-remove="handleRemove"
               ref="multiUpload"
               :file-list="fileList"
    >
      <i class="el-icon-plus"></i>
    </el-upload>
    <el-upload ref="singleUpload" v-else
               :style="avatarStyle"
               class="logo-uploader"
               :action="baseUrl + url"
               :show-file-list="false"
               :on-success="handleSuccess">
      <div @mouseover="showBtn=true" @mouseout="showBtn=false">
        <i @click.stop="removeSingle" v-show="dialogImageUrl && showBtn" class="el-icon-close remove-btn"></i>
        <img v-if="dialogImageUrl" :src="dialogImageUrl" :style="avatarStyle">
        <i v-else class="el-icon-plus logo-uploader-icon" :style="avatarStyle"></i>
      </div>
    </el-upload>
    <v-dialog v-model="show" max-width="500">
      <img width="500px" :src="dialogImageUrl" alt="">
    </v-dialog>
  </div>
</template>

<script>
  import {Upload} from 'element-ui';
  import config from '../../config'


  export default {
    name: "vUpload",
    components: {
      elUpload: Upload
    },
    props: {
      url: {
        type: String
      },
      value: {},
      multiple: {
        type: Boolean,
        default: true
      },
      picWidth: {
        type: Number,
        default: 150
      },
      picHeight: {
        type: Number,
        default: 150
      }
    },
    data() {
      return {
        showBtn: false,
        show: false,
        dialogImageUrl: "",
        baseUrl: config.api,
        avatarStyle: {
          width: this.picWidth + 'px',
          height: this.picHeight + 'px',
          'line-height': this.picHeight + 'px'
        },
        fileList:[]
      }
    },
    mounted(){
      if (!this.value || this.value.length <= 0) {
        return;
      }
      if (this.multiple) {
        this.fileList = this.value.map(f => {
          return {response: f, url:f}
        });
      } else {
        this.dialogImageUrl = this.value;
      }
    },
    methods: {
      async handleUpload(params) {
        const file = params.file;
        const fileType = file.type;
        const isImage = fileType.indexOf('image') !== -1;

        const isLt2M = file.size < 2 * 1024 * 1024;

        if (!isLt2M) {
          this.$message.error('上传图片或视频大小不能超过 2MB!');
          return;
        }

        if (!isImage) {
          this.$message.error('请选择图片!');
          return;
        }
        // 准备文件名称
        let filename = file.name;
        let uploadUrl = this.url;
        // 准备上传表单
        const formData = new FormData();
        // 判断上传到哪里
        if (this.needSignature) {
          // 上传到阿里，进行签名，把接收到的url作为签名接口地址
          const resp = await this.$http.loadData(this.url);
          // 判断接口返回的签名时间是否超时
          if (resp.expire < new Date().getTime()) {
            this.$message.error('请求超时!');
            return;
          }
          // 修改文件名为随机文件名
          filename = fileUtil.getFileName(file.name, resp.dir);
          // 修改上传文件路径
          uploadUrl = resp.host;
          // 准备请求参数
          formData.append("key", filename);
          formData.append("policy", resp.policy);
          formData.append("OSSAccessKeyId", resp.accessId);
          formData.append("success_action_status", '200');
          formData.append("signature", resp.signature);
        }
        formData.append("file", file);

        // this.$http.post(uploadUrl, formData, {headers: {'Content-Type': 'multipart/form-data'}})
        //   .then(resp => {
        //     console.log(resp);
        //     if (resp.status == '200') {
        //       // 获取文件路径
        //       const fileUrl = resp.data || uploadUrl + "/" + filename;
        //       // 判断是多文件还是单文件
        //       if (!this.multiple) {
        //         // 单文件，直接获取地址即可；
        //         this.dialogImageUrl = fileUrl;
        //         this.$emit("input", fileUrl)
        //       } else {
        //         // 多文件，放到集合中
        //         const files = this.$refs.multiUpload.uploadFiles;
        //         files[files.length - 1].response = fileUrl;
        //         this.fileList = files;
        //         this.$emit("input", files.map(f => f.response))
        //       }
        //     } else {
        //       this.$message.error('上传失败');
        //     }
        //   }).catch(function (err) {
        //   this.$message.error('上传失败');
        //   console.log(err);
        // });
      },
      handleSuccess(resp, file) {
        console.log("handleSuccess")
        if (!this.multiple) {
          this.dialogImageUrl = file.response;
          this.$emit("input", file.response)
        } else {
          this.fileList.push(file)
          this.$emit("input", this.fileList.map(f => f.response))
        }
      },
      handleRemove(file, fileList) {
        console.log("handleRemove")
        this.fileList = fileList;
        this.$emit("input", fileList.map(f => f.response))
      },
      handlePictureCardPreview(file) {
        console.log("handlePictureCardPreview")
        this.dialogImageUrl = file.response;
        this.show = true;
      },
      removeSingle() {
        console.log("removeSingle")
        this.dialogImageUrl = "";
        this.$refs.singleUpload.clearFiles();
      }
    },
    watch: {
      value:{
        deep:true,
        handler(val,oldVal){

          if (this.multiple) {
            this.fileList = val.map(f => {
              return {response: f,url:f}
            });
          } else {
            this.dialogImageUrl = val;
          }
        }
      }
    }
  }
</script>

<style scoped>
  .logo-uploader {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    float: left;
  }

  .logo-uploader:hover {
    border-color: #409EFF;
  }

  .logo-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    text-align: center;
  }

  .remove-btn {
    position: absolute;
    right: 0;
    font-size: 16px;
  }

  .remove-btn:hover {
    color: #c22;
  }
</style>
