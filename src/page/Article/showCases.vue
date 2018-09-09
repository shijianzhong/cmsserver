<template>
    <div class="show-cases">
        <el-form ref="form" :model="data" :rules="rules" label-width="80px">
            <el-form-item label="标题" prop="title">
                <el-input v-model="data.title"></el-input>
            </el-form-item>

            <el-form-item label="头像" prop="pic">
                <el-input v-model="data.pic"></el-input>
                <up-file ref="upload" :upload="{}" @successUpload="successUpload"></up-file>
                <el-button @click="upImg" :disabled="grade.upFile">上传图片</el-button>
            </el-form-item>
            <el-form-item label="内容" prop="content">
                <VueEditor :content="data.content" :height="250" :auto-height="false" @change="changeContent"></VueEditor>
            </el-form-item>
            <el-form-item style="text-align: right">
                <el-button @click="addArticle">新增</el-button>
                <el-button type="primary" :disabled="grade.updateArticle||loading" @click="saveArticle">保存文章</el-button>
            </el-form-item>
        </el-form>
        <div class="caselist">
            <el-table :data="tableData" border style="width: 100%">
                <el-table-column fixed prop="title" label="标题" width="320">
                </el-table-column>
                <el-table-column prop="imgsrc" label="头像" width="120">
                    <template slot-scope="scope">
                        <img :src="scope.row.imgsrc" alt="" style="width: 50px;height: 50px">
                    </template>
                </el-table-column>
                <el-table-column prop="content" label="内容">
                </el-table-column>
                <el-table-column label="操作" width="100">
                    <template slot-scope="scope">
                        <el-button @click="deletes(scope.row)" type="text" size="small">删除</el-button>
                        <el-button @click="edite(scope.row)" type="text" size="small">编辑</el-button>
                    </template>
                </el-table-column>
            </el-table>
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
      tableData: [
        {
          title: "王小虎",
          imgsrc: "http://localhost:3001/upFile/1536482373259.png",
          content: "普陀区"
        }
      ],
      page_grade: common.page_grade,
      grade: {
        updateArticle: !0,
        upFile: !0
      },
      userInfo: {},
      read_type: common.user_type,
      loading: false,
      data: {
        title: "",
        pic: "",
        content: "",
        id: ""
      },
      rules: {
        title: [
          {
            required: true,
            message: "标题不能为空",
            trigger: "change"
          },
          {
            pattern: /^.{1,100}$/,
            message: "请输入1-100个字符的文章标题",
            trigger: "blur"
          }
        ],
        pic: {
          required: true,
          message: "头像比传"
        },
        content: {
          required: true,
          message: "文章内容不能为空"
        }
      }
    };
  },
  //         getCasesList,
  // deleteShowCases,
  // InsertShowCases
  //UpdateCases
  created() {},
  methods: {
    addArticle() {
      this.data = {
        title: "",
        pic: "",
        content: "",
        id: ""
      };
    },
    deletes(item) {
      ajax.call(this, "/deleteShowCases", item, (data, err) => {
        this.loading = false;
        if (!err) {
          this.getCasesList();
          this.$message({
            message: "删除",
            type: "success"
          });
        }
      });
    },
    edite(item) {
      this.data.title = item.title;
      this.data.pic = item.imgsrc;
      this.data.content = item.content;
      this.data.id = item.id;
    },
    saveArticle() {
      if (this.data.id != "") {
        ajax.call(this, "/UpdateCases", this.data, (data, err) => {
          this.loading = false;
          if (!err) {
            this.getCasesList();
            this.$message({
              message: "更新成功",
              type: "success"
            });
          }
        });
      } else {
        this.$refs.form.validate(v => {
          if (v) {
            this.visible = true;
            ajax.call(this, "/InsertShowCases", this.data, (data, err) => {
              this.loading = false;
              if (!err) {
                this.getCasesList();
                this.$message({
                  message: "保存成功",
                  type: "success"
                });
              }
            });
          }
        });
      }
    },
    getCasesList() {
      ajax.call(this, "/getCasesList", {}, (data, err) => {
        if (data.length > 0) {
          this.tableData = data.map(item => ({
            id: item.id,
            title: item.title,
            imgsrc: item.headimg,
            content: item.content
          }));
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
      // this.data.content += '<img src="'+data.filename+'" />';
    }
  },
  mounted() {
    storage.get("userInfo", obj => {
      this.data.user_name = obj.userInfo.user_name;
    });
    this.getCasesList();
  },
  watch: {},
  mixins: [common.mixin],
  components
};
</script>
<style lang="less">
.show-cases {
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
}
</style>
