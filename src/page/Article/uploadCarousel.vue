<template>
    <div class="upload-carousel">
        
        
        <el-input v-model="url" style="margin-top:10px" placeholder="点击上传按钮之前：输入该图跳转链接"></el-input>
        <el-input v-model="type" style="margin-top:10px" placeholder="点击上传按钮之前：如果是首页的报名图请填数字1，轮播图不用填"></el-input>
        <el-input style="margin-top:10px" v-model="data.pic"></el-input>
        <up-file ref="upload" :upload="{}" @successUpload="successUpload"></up-file>
        <el-button @click="upImg">上传图片</el-button>

        <div class="upload-view">
            <div class="upload-view-contain">
                <div class="upload-view-item" v-for="(item,index) in  data.content" :key="index">
                    <img :src="item.src">
                    <el-button type="text" @click="deleteImg(item,index)">删除</el-button>
                </div>
                <div class="upload-view-item" v-if="data.content.length>0">
                    <el-button @click="submitImg">统一提交</el-button>
                </div>
            </div>
        </div>

    </div>
</template>
<script type="text/javascript">
import { ajax, storage } from "utils";
import common from "common";
import components from "components";
module.exports = {
  name: "list",
  data() {
    return {
      loading: false,
      url:"https://www.baidu.com",
      type:'',
      data: {
        id: 0,
        title: "",
        sort_id: "",
        description: "",
        user_name: "",
        pic: "",
        read_type: 4,
        content: []
      }
    };
  },
  created() {
    this.getCarouselList();
  },
  methods: {
    getCarouselList() {
      ajax.call(this, "/getCarouselList", {}, (data, err) => {
        if (!err) {
          data.forEach(item => {
            this.data.content.push({
              id: item.id,
              src: item.imgsrc
            });
          });
        }
      });
    },
    deleteImg(item, index) {
      if (item.id == "") {
        this.data.content.splice(index, 1);
      } else {
        let id = item.id;
        ajax.call(this, "/deleteCarouselImg", { id }, (data, err) => {
          if (data.affectedRows == 1) {
            this.data.content.splice(index, 1);
            this.$message({
              message: "删除成功",
              type: "success"
            });
          }
        });
      }
    },
    submitImg() {
      let d = [];
      this.data.content.forEach(item => {
        d.push({
          imgsrc: item.src,
          url:item.url,
          type:item.type
        });
      });
      let da = { a: JSON.stringify(d) };
      ajax.call(this, "/uploadCarouselImg", da, (data, err) => {
        if (!err) {
          this.$message({
            message: "保存成功",
            type: "success"
          });
        }
      });
    },

    saveArticle() {
      this.$refs.form.validate(v => {
        if (v) {
          this.visible = true;
          this.data.article_extend = JSON.stringify({ pic: this.data.pic });
          ajax.call(this, "/updateArticle", this.data, (data, err) => {
            this.loading = false;
            if (!err) {
              this.$message({
                message: "保存成功",
                type: "success"
              });
              this.$router.push("/article/list");
            }
          });
        }
      });
    },
    formatTooltip(v) {
      return common.user_type[v];
    },
    changeContent(v) {
      this.data.content = v;
      this.$refs.form.validateField("content");
    },
    upImg() {
      this.$refs.upload.SelectFile();
    },
    backList() {
      this.$router.push("/article/list");
    },
    successUpload(data) {
      this.data.pic = data.filename;
      this.data.content.push({
        id: "",
        src: data.filename,
        url:this.url,
        type:this.type
      });
    }
  },
  mounted() {
    ajax.call(this, "/listSort", {}, (data, err) => {
      if (!err) {
        let arr = data.data;
        arr.sort((a, b) => (a.parent_id > b.parent_id ? 1 : -1));
        for (let i = arr.length; i--; ) {
          if (arr[i].parent_id) {
            let obj = arr.pop();
            arr.forEach(item => {
              if (item.id === obj.parent_id) {
                item.children = item.children || [];
                item.children.push(obj);
              }
            });
          }
        }
        this.sort_data = arr;
        let id = this.$route.params.id;
        if (id) {
          ajax.call(this, "/getArticleById", { id }, (obj, err) => {
            if (!err) {
              if (
                this.userInfo.user_type > 2 &&
                obj.user_id !== this.userInfo.id
              ) {
                return this.$router.back();
              }
              Object.getOwnPropertyNames(this.data).forEach(key => {
                this.data[key] = obj[key];
              });
              try {
                this.data.pic = JSON.parse(obj.article_extend).pic;
              } catch (e) {
                this.data.pic = "";
              }
              //显示分类
              const cid = obj.sort_id;
              let hasFind = false;
              let cb = (array, a) => {
                !hasFind &&
                  array &&
                  array.forEach(item => {
                    a = a || [];
                    let _a = [].concat(a);
                    _a.push(item.id);
                    if (item.id === cid) {
                      hasFind = true;
                      this.sort_id = _a;
                    } else {
                      cb(item.children, _a);
                    }
                  });
              };
              cb(arr);
            }
          });
        }
      }
    });
  },
  watch: {
    sort_id(val) {
      this.data.sort_id = val.length ? val.slice(-1)[0] : "";
    }
  },
  mixins: [common.mixin],
  components
};
</script>
<style lang="less">
.upload-carousel {
  .el-input,
  .el-textarea__inner,
  .el-slider {
    max-width: 60%;
  }
  .el-cascader {
    width: 100%;
    max-width: 60%;
    .el-input {
      width: 100%;
    }
  }
  .upload-view {
    .upload-view-contain {
      display: flex;
      display: -webkit-flex;
      padding-top: 20px;
      flex-wrap: wrap;
      .upload-view-item {
        width: 100px;
        height: 100px;
        margin: 20px 30px;
        display: flex;
        display: -webkit-flex;
        align-items: center;

        img {
          max-width: 100px;
          max-height: 100px;
        }
      }
    }
  }
}
</style>
