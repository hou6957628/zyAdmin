<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/bankCardManagement' }">银行卡管理列表</el-breadcrumb-item>
      <el-breadcrumb-item>银行卡详情</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="bankBox">
      <p><span>编号：</span>{{bankCard.id}}</p>
      <p><span>卡号：</span>{{bankCard.cardNumber}}</p>
      <p><span>类型：</span>{{bankCard.type==0?'储蓄卡':'信用卡'}}</p>
      <p><span>银行名称：</span>{{bankCard.bankName}}</p>
      <p><span>开户支行：</span>{{bankCard.subBranch}}</p>
      <p><span>预留手机号：</span>{{bankCard.mobile}}</p>
      <p><span>所属应用：</span>{{bankCard.productName}}</p>
      <p><span>是否是放款卡：</span>{{bankCard.id}}</p>
      <p><span>是否是快捷支付：</span>{{bankCard.id}}</p>
      <p><span>创建时间：</span>{{bankCard.createDate}}</p>
      <p><span>更新时间：</span>{{bankCard.updateDate}}</p>
      <p><span>用户姓名：</span>{{bankCard.name}}</p>
      <p><span>身份证号：</span>{{bankCard.number}}</p>
      <p><span>注册手机号：</span>{{bankCard.registerMobile}}</p>
    </div>
    <el-button-group class="btGrop">
      <el-button v-if="this.hasPermissionCustom('credit:bank:delete')" class="jie" type="primary" @click="editProduct()">解绑</el-button>
      <el-button @click="guan()" class="guan">关闭</el-button>
    </el-button-group>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    methods: {
      //根据id获取银行卡
      getBankCardById(id){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/credit/admin/bank/getBankCardInfoById",
          // url:"http://localhost:9996/credit/admin/bank/getBankCardInfoById",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id:id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.bankCard=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //提示删除银行卡
      editProduct(){
        this.$confirm('是否确认解绑此银行卡?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }).then(() => {
          this.deleteBankCard();
        });
      },
      //确认删除银行卡接口
      deleteBankCard(row){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/credit/admin/bank/deleteBankCardInfo",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id:this.bankCard.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '删除成功',
              type: 'success'
            });
            this.$route.go(-1);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      guan(){
        this.$router.push({
          path: `/bankCardManagement`,
        });
      },
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getBankCardById(this.id);
    },
    data() {
      return {
        bankCard: '',
      }
    }
  }
</script>

<style scoped>

  .content {
    padding-left: 200px;
    padding-top: 80px;
  }
  .fs-16{
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .operationContent .searchContent{
    margin-left:5px;
    width: 200px;
    margin-right: 30px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
  .bankBox p{
    height: 35px;
    line-height: 35px;
  }
  .bankBox p span{
    display: inline-block;
    width: 140px;
  }
  .btGrop{
    display: block;
    width: 500px;
    margin: 0 auto;
    margin-top: 50px;
  }
  .jie{
    display: inline-block;
    font-size: 20px;
    padding: 15px 40px;
  }
  .content .guan{
    display: inline-block;
    font-size: 20px;
    padding: 15px 40px;
    margin-left: 40px;
  }
</style>
