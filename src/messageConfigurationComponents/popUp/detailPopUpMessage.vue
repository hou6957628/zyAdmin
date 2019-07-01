<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>弹窗消息</el-breadcrumb-item>
      <el-breadcrumb-item>详情</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="bankBox">
      <p><span>产品：</span>{{ruleForm.productName}}</p>
      <p><span>名称：</span>{{ruleForm.name}}</p>
      <p><span>分类：</span>{{ruleForm.classifyName}}</p>
      <p><span>备注：</span>{{ruleForm.description}}</p>
      <p><span>内容：</span>{{ruleForm.content}}</p>
      <p><span>位置：</span>{{ruleForm.positionName}}</p>
      <p><span>次数：</span>{{ruleForm.popupCount}}</p>
      <p><span>弹窗图：</span><img style="height: 108px;width: 170px"
                               :src="'http://'+this.baseUrl + '/message/admin/message/down?file='+ruleForm.picture+'&productCode='+ruleForm.productCode"/></p>
    </div>
    <el-button-group class="btGrop">
      <el-button @click="resetForm()" class="guan">返回</el-button>
    </el-button-group>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        ruleForm: {},
      };
    },
    methods: {
      //查询消息详情
      getMessageById(id) {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //取消按钮
      resetForm() {
        this.$router.go(-1);
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getMessageById(this.id);
    },
  }
</script>

<style scoped>
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }
  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
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
  .content .guan{
    display: inline-block;
    font-size: 20px;
    padding: 15px 40px;
    margin-left: 40px;
  }
</style>
