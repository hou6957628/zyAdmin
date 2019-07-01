<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>紧急联系人</el-breadcrumb-item>
      <el-button type="info"  @click="cancel">返回<i class="el-icon-close el-icon--right"></i></el-button>
    </el-breadcrumb>
    <template>
      <el-table
        ref="multipleTable"
        :data="tableData"
        border
        style="width: 60%">
        <el-table-column
          fixed
          prop="name"
          label="姓名"
          width="250">
        </el-table-column>
        <el-table-column
          fixed
          prop="phoneNum"
          label="手机号"
          min-width="80">
        </el-table-column>
        <el-table-column
          fixed
          prop="relation"
          label="关系"
          :formatter="relationFormatter"
          min-width="80">
        </el-table-column>
      </el-table>
    </template>
  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
      /**
       * 紧急联系人
       */
      getProductList(id) {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/user_center/admin/customer/find",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.tableData = res.data.body.linkMan;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //返回
      cancel(){
        this.$router.go(-1);
      },
      //关系字段
      relationFormatter(row){
        let relation = row.relation;
        if(relation==0){
          var result = "父母";
          return result;
        }else if(relation==1){
          var result = "配偶";
          return result;
        }
        else if(relation==2){
          var result = "兄弟姐妹";
          return result;
        }
        else if(relation==3){
          var result = "同事";
          return result;
        }
        else if(relation==4){
          var result = "同学";
          return result;
        }
        else if(relation==5){
          var result = "朋友";
          return result;
        }
      },
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getProductList(this.id);
    },
    data() {
      return {
        tableData:[],
      }
    }
  }
</script>

<style scoped>
  .el-col-8{
    height: 55px;
  }
  .select{
    margin-left: 20px;
  }
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }
  .fs-16{
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .operationContent{
    text-align: left;
    margin: 25px 30px 15px 0;
  }
  .operationContent .upLoadBtn{

  }
  .operationContent .searchContent{
    margin-left:0px;
    width: 200px;
    margin-right: 30px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
