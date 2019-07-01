<template>
  <div class="container" id="app">
    <!--<p class="topClass"><img src="http://www.zytech360.com/images/daichaoLogo.png"/></p>-->
    <div class="contentBox">
      <div class="content">
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="60px" class="demo-ruleForm">
          <el-form-item label="账号" prop="username" style="margin-bottom: 30px;font-size: 16px;">
            <el-input v-model="ruleForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password" style="margin-bottom: 10px;font-size: 16px;">
            <el-input type="password" v-model="ruleForm.password" @keyup.enter.native="submitForm('ruleForm')"></el-input>
          </el-form-item>
          <el-form-item style="text-align: left">
            <el-checkbox-group v-model="ruleForm.type">
              <el-checkbox label="记住我的账号密码" name="type"></el-checkbox>
            </el-checkbox-group>
          </el-form-item>
          <el-form-item>
            <el-button class="btn" type="primary" @click="submitForm('ruleForm')">登录</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios';
  import qs from 'qs';
  export default {
    data() {
      return {
        topBar:false,
        leftSilder:false,
        ruleForm: {
          username: 'admin',
          password: 'admin123',
          type: true,
        },
        rules: {
          username: [
            {required: true, message: '请输入账户', trigger: 'blur'},
          ],
          password: [
            {required: true, message: '请输入密码', trigger: 'blur'},
          ]
        },
      };
    },
    methods: {
      submitForm(formName) {
        var data =  qs.stringify({
          userName: this.ruleForm.username,
          password: this.ruleForm.password
        });
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              url:"http://"+this.baseUrl+"/operate/admin/user/login",
              method:"post",
              data:data,
              headers:{
                'Content-Type':'application/x-www-form-urlencoded'
              }
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                localStorage.token=res.data.body.token;
                this.$message({
                  message: '登录成功',
                  type: 'success'
                });
                this.getPermissionByUserId();
                this.$forceUpdate();
              }else {
                this.$message.error(res.data.msgInfo);
              }

            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
      },
      goHome(url){
        this.$router.push('/'+url);
      },
      //获取登录用户的权限
      getPermissionByUserId(){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/getAuthoryByUser",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            //左侧列表权限
            let arrV1 = ["operate:productlist","operate:merchantlist","operate:productManage:list","user:customer:list","user:black:list","credit:bank:getBankCardInfoByParams",
              "risk:field:list","risk:tag:list","risk:rule:list","risk:rule_collection:list","risk:collection_flow:list","risk:parent_flow:list","risk:flow_log:list",
              "order:overdue:orderList","order:overdue:noRepayOrderList","order:overdue:repayedOrderList","order:outsourcing:orderList","order:outsourcing:repayedOrderList",
              "order:label:list","order:callType:list","operate:collection:list","operate:group:list","order:distribution:find","order:borrowing:findAssignedOrder",
              "order:borrowing:findCollectionOrder","order:borrowing:findCollectionOrder","operate:role:list","operate:account:list","channel:channel:list","channel:channel_statistics:list",
              "channel:account:list","order:audit:list","order:pending:list","order:borrowing:list","order:borrowing:list","order:repayment:list","order:payment:list",
              "quota:amountField:list","quota:amountTag:list","quota:amountFlow:list","quota:amountFlow:list","order:flowField:getFieldList","order:flowField:list",
              "order:ordertTag:hitList","message:message_classify:find","message:message:smsMessage","message:message:reminderMessage","message:message:popUpMessage","message:message:pushMessage",
              "message:task:list","message:messagelog:list"];
            let arrV2 = ["financeProductList","merchantProductList","productProductList","userProductList","blackList","bankCardManagement","fieldList","tagList","ruleList",
              "collectionList","flowList","flowHeadList","flowLogList","afterLoanManagement","afterLoanNoRepay","afterLoanRepayed","outsourcingOrderList","outsourcingOrderRepayedList",
              "collectionTag","callTypeTag","collectionPersonManagement","collectionGroupManagement","collectionOrder","collectionOrderList","collectionTaskList","collectionTaskFinishList",
              "roleList","accountManagement","channelconnectionManagement","channelCount","accountCenter","pendingApproval","pendingLoan","loanMade","orderList",
              "paymentHistory","loanRecord","amountFieldList","amountTagList","amountFlowList","amountFlowLogList","orderFieldList","orderFlowList","orderFlowLogList",
              "messageClassify","smsMessage","reminderMessage","popUpMessage","pushMessage","messageConfigurationList","messageRecord"];
            let arr = res.data.body;
            let flag = '';
            for(var i=0;i<arr.length;i++) {
              flag = arrV1.indexOf(arr[i]);
              if (flag != -1) {
                console.log(flag);
                break;
              }
            }
            localStorage.authorities=res.data.body;
            if(localStorage.authorities.length){
              this.goHome(arrV2[flag]);
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
  }
</script>

<style scoped>
  * {
    margin: 0;
    padding: 0;
  }

  .container {
    height: 100%;
    width: 100%;
    max-height: 100%;
    overflow: hidden;
    background-color: #0a93f7;
    /*background: url("http://www.zytech360.com/images/banner2.jpg");*/
    background-size: cover;
  }

  .container .topClass {
    width: 600px;
    margin: 0 auto;
    margin-top: 100px;
    margin-bottom: 20px;
    font-size: 20px;
  }

  .container .topClass img {
    display: inline-block;
    width: 800px;
    height: auto;
    margin: 0 auto;
  }

  .contentBox {
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -250px;
    margin-top: -200px;
    border-radius: 4px;
    -moz-border-radius: 4px;
    -webkit-border-radius: 4px;
    background-color: #fff;
    width: 500px;
    height: 400px;
    box-shadow: 0 2px 10px #999;
    -moz-box-shadow: #999 0 2px 10px;
    -webkit-box-shadow: #999 0 2px 10px;
  }

  .demo-ruleForm {
    width: 400px;
    margin: 0 auto;
    text-align: center;
    margin-top: 100px;
  }

  .contentBox .el-form-item__content {
    height: 60px;
    line-height: 60px;
  }

  .contentBox .btn {
    width: 120px;
    height: 40px;
    font-size: 16px;
    letter-spacing: 2px;
    color: #fff;
    background-color: #409eff;
    border-color: #409eff;
    margin-top: 40px;
  }
  .contentBox .btn:hover {
    background: #66b1ff;
    border-color: #66b1ff;
    color: #fff;
  }
</style>
