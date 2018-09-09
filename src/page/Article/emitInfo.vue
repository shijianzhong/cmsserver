<template>
    <div class="emitinfoCon">
        <div class="emitinfoitem" v-for="(item,index) in infos" :key="index">
            <span class="no">{{index+1}}:</span>
            <el-input class="info" v-model="item.info"></el-input>
            <el-button @click="plusClick(index)">+</el-button>
            <el-button @click="minusClick(index)">-</el-button>
        </div>
        <div class="submit">
            <el-button class="submitbtn" type="primary" @click="submitClick">提交</el-button>
        </div>
    </div>
</template>
<script>
import { ajax, storage } from "utils";
export default {
  data() {
    return {
      infos: [
        {
          info: "史建忠被清华大学录取"
        }
      ]
    };
  },
  created(){
      this.getEmitInfo();
  },
  methods: {
    plusClick(index) {
      this.infos.splice(index + 1, 0, {
        info: ""
      });
    },
    minusClick(index) {
      this.infos.splice(index, 1);
    },
    submitClick(){
        this.submitEmitInfo();
    },
    getEmitInfo() {
      ajax.call(this, "/getEmitInfoList", {}, (data, err) => {
        if (!err) {
          if(data.length>0){
              this.infos=[];
          }
          data.forEach(item => {
            this.infos.push({
              id:item.id,
              info: item.emiteinfo
            });
          });
        }
      });
    },
    deleteEmitInfo(item, index) {
      if (item.id == "") {
        this.infos.splice(index, 1);
      } else {
        let id = item.id;
        ajax.call(this, "/deleteEmitInfo", { id }, (data, err) => {
          if (data.affectedRows == 1) {
            this.infos.splice(index, 1);
            this.$message({
              message: "删除成功",
              type: "success"
            });
          }
        });
      }
    },
    submitEmitInfo() {
      let d = [];
      this.infos.forEach(item => {
        d.push({
          info: item.info
        });
      });
      let da = { a: JSON.stringify(d) };
      ajax.call(this, "/InsertEmitInfo", da, (data, err) => {
        if (!err) {
          this.$message({
            message: "保存成功",
            type: "success"
          });
        }
      });
    }
  }
};
</script>

<style lang="less">
.emitinfoCon {
  width: 500px;
  .emitinfoitem {
    margin-bottom: 20px;
    .info {
      width: 300px;
      margin-right: 20px;
    }
    .no {
      margin-right: 20px;
      display: inline-block;
      width: 35px;
    }
  }
  .submit {
    text-align: center;
    .submitbtn {
      width: 200px;
    }
  }
}
</style>

