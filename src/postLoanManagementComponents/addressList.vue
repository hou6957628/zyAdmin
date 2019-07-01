<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>通讯录</el-breadcrumb-item>
      <el-button type="info"  @click="cancel">返回<i class="el-icon-close el-icon--right"></i></el-button>
    </el-breadcrumb>
    <template>
      <el-table
        ref="multipleTable"
        :data="tableData"
        border
        style="width: 50%">
        <el-table-column
          prop="name"
          label="姓名"
          width="250">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          min-width="80">
        </el-table-column>
        <el-table-column
          label="操作"
          min-width="80">
          <template slot-scope="scope">
            <el-button type="primary" size="mini" icon="el-icon-phone-outline" plain @click="toPhone(scope.row)"></el-button>
            <el-button type="primary" size="mini" icon="el-icon-edit" plain @click="addRecord()"></el-button>
            <el-button type="primary" size="mini" icon="el-icon-document" plain @click="recordList()"></el-button>
          </template>
        </el-table-column>
      </el-table>
    </template>
  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
      //打电话
      toPhone(row){
        let phone = row.mobile;
        let cc = this.EncryptCustom(phone);
        var a = document.createElement('a');
        a.setAttribute('href', 'https://xmdd.qxykjz.com/dist1/demo.html?phone=' + cc);
        a.setAttribute('target', '_blank');
        a.setAttribute('id', 'startTelMedicine');
        // 防止反复添加
        if(document.getElementById('startTelMedicine')) {
          document.body.removeChild(document.getElementById('startTelMedicine'));
        }
        document.body.appendChild(a);
        a.click();
      },
      //添加催收记录
      addRecord(){
        let orderId=this.orderId2;
        this.$router.push({
          path: `/addCollectRecord2/${orderId}`,
        });
      },
      //催收记录列表
      recordList(){
        let id=this.id;
        this.$router.push({
          path: `/collectRecordList/${id}`,
        });
      },
      /**
       * 通讯录列表
       */
      getProductList(data){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/credit/admin/getAddressListByCustomerId",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            customerId: data,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      cancel(){
        this.$router.go(-1);
      }
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.orderId2=this.$route.params.orderId;
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
