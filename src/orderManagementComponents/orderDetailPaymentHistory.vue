<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/pendingLoan' }">还款记录列表</el-breadcrumb-item>
      <el-breadcrumb-item>详情</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="jiben">
      <h3>订单信息</h3>
      <table>
        <tr>
          <td>订单号：{{this.borrowingForm.orderId}}</td>
          <td>手机号：{{this.borrowingForm.mobile}}</td>
          <td>渠道：{{this.borrowingForm.parentChannelName==null?'--':this.borrowingForm.parentChannelName}}</td>
          <td>订单状态：{{this.borrowingForm.status | statusFalse}}</td>
          <td>新户老户：{{this.borrowingForm.reBorrow==true?'老户':'新户'}}</td>
          <td>所属平台：{{this.borrowingForm.productName}}</td>
        </tr>
        <tr>
          <td>申请时间：{{this.borrowingForm.createDate}}</td>
          <td>放款时间：{{this.borrowingForm.borrowingPaymentDate==null?'--':this.borrowingForm.borrowingPaymentDate}}</td>
          <td>预计还款时间：{{this.borrowingForm.repaymentEndDate}}</td>
          <td>实际还款时间：{{this.borrowingForm.repaymentPaymentDate==null?'--':this.borrowingForm.repaymentPaymentDate}}</td>
          <td>借款金额：{{this.borrowingForm.borrowingCapital}}</td>
          <td>期限：{{this.borrowingForm.borrowingPeriod}}</td>
        </tr>
        <tr>
          <td>是否逾期：{{this.borrowingForm.repaymentOverdueDay==null?'否':'是'}}</td>
          <td>逾期天数：{{this.borrowingForm.repaymentOverdueDay}}</td>
          <td>应还利息（元）：{{this.borrowingForm.borrowingInterest}}</td>
          <td>罚息（元）：{{this.borrowingForm.repaymentPenaltyInterest}}</td>
          <td>滞纳金（元）：{{this.borrowingForm.repaymentOverdueFee}}</td>
          <td>应还总金额（元）：{{this.borrowingForm.repaymentCapital + this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest}}</td>
        </tr>
        <tr>
          <td>是否可展期：{{this.borrowingForm.enableDefer | enableDeferFalse}}</td>
          <td>展期应还金额：{{this.borrowingForm.repaymentDefer}}</td>
          <td>展期实际还款金额（元）：{{this.borrowingForm.repaymentDeferPayment}}</td>
          <td>减免金额：{{this.borrowingForm.repaymentDiscountAmount}}</td>
        </tr>
        <tr>
          <td>催收员：{{this.borrowingForm.collectorName}}</td>
        </tr>
      </table>
    </div>
    <el-button-group style="margin: 0 auto;width: auto;display: block;margin-top: 40px;margin-bottom: 40px">
      <el-button v-if="!this.cusCustomer.isBlackList && (hasPermissionCustom('order:repayment:black') || hasPermissionCustom('order:orderAll:black'))" class="la" type="danger" @click="addBlack()">拉黑</el-button>
      <el-button style="margin-left: 10px" v-if="this.cusCustomer.isBlackList  && (hasPermissionCustom('order:repayment:black') || hasPermissionCustom('order:orderAll:black'))" class="la" type="danger" @click="removeBlack()">移除黑名单</el-button>
      <el-button style="margin-left: 10px" class="la" type="danger" @click="resetForm()">关闭</el-button>
    </el-button-group>
    <div class="listContent">
      <router-link :to="'/jibenOrder4/'+this.id+'/'+this.orderId2" tag="li">基本信息</router-link>
      <router-link v-if="hasPermissionCustom('order:orderAll:hitList')" :to="'/fenxianOrder4/' + this.id" tag="li">风险命中列表</router-link>
      <router-link :to="'/yunyingOrder4/' + this.id" tag="li">运营商通讯录比对</router-link>
      <template v-if="hasPermissionCustom('order:orderAll:tianji')">
        <a :href="this.tianjiReport.tianjiUrl | htmlFalse" target="_blank" class="ddd">天机报告</a>
      </template>
      <a href="http://www.baidu.com" target="_blank" class="ddd">支付宝报告</a>
      <router-link :to="'/yonghuOrder4/' + this.id" tag="li">用户催收记录</router-link>
      <router-link :to="'/dingdanOrder4/' + this.id" tag="li">订单记录</router-link>
      <router-link :to="'/fangkuanOrder4/' + this.id" tag="li">放款记录</router-link>
      <router-link :to="'/huankuanOrder4/' + this.id" tag="li">还款记录</router-link>
    </div>
    <router-view/>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        borrowingForm:{},
        productList:[],
        zhimaFen:{},
        bankCard:[],
        cusCustomer:{},
        tianjiReport:{
          tianjiUrl:''
        },
        idCard:{},
        idFace:{},
        linkMan:[],
        id:null,
        orderId2:'',
        authorizationStatus:[],
        basicInfo:null,
        tableData:[],
        electValue:0,
        isactive:0,
      };
    },
    methods: {
      //拉黑
      addBlack() {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/user_center/admin/black/setBlack",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            customerId: this.id,
            description: '后台系统手动拉黑',
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.$router.go(-1);
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //移除黑名单
      removeBlack() {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/user_center/admin/customer/deleteBlack",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: this.id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.$router.go(-1);
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //取消按钮
      resetForm() {
        this.$router.go(-1);
      },
      //用户基本信息
      getUserDetail1(id) {
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
            this.productList = res.data.body.productList;
            this.zhimaFen = res.data.body.zhimaFen;
            this.bankCard = res.data.body.bankCard;
            this.cusCustomer = res.data.body.cusCustomer;
            this.tianjiReport = res.data.body.tianjiReport;
            this.tianjiReport.tianjiUrl = res.data.body.tianjiReport.html;
            this.idCard = res.data.body.idCard;
            this.idFace = res.data.body.idFace;
            this.linkMan = res.data.body.linkMan;
            this.authorizationStatus = res.data.body.authorizationStatus;
            this.basicInfo = res.data.body.basicInfo;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //用户订单信息
      getOrderInfo(id) {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/info",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            orderId: id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.borrowingForm = res.data.body;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.orderId2=this.$route.params.orderId;
      this.getUserDetail1(this.id);
      this.getOrderInfo(this.orderId2);
    },
    filters:{
      typeFalse:function(arg1){
        if(arg1==true){
          var result = "是";
          return result;
        }else if(arg1==false){
          var result = "否";
          return result;
        }
      },
      reborrowFalse:function(arg1){
        if(arg1==true){
          return "老户";
        }else if(arg1==false){
          return "新户";
        }
      },
      statusFalse:function(arg1){
        if(arg1==0){
          return '待机器审核 ';
        } else if (arg1 === 1){
          return '机器审核中';
        } else if (arg1 === 2){
          return '审核拒绝';
        } else if (arg1 === 3){
          return '人工审核';
        } else if (arg1 === 4){
          return '待放款';
        } else if (arg1 === 5){
          return '放款中';
        } else if (arg1 === 6){
          return '放款失败';
        } else if (arg1 === 7){
          return '取消';
        } else if (arg1 === 8){
          return '放款成功';
        } else if (arg1 === 9){
          return '还款确认中';
        } else if (arg1 === 10){
          return '正常还款 ';
        } else if (arg1 === 11){
          return '逾期未还';
        } else if (arg1 === 12){
          return '坏账';
        } else if (arg1 === 13){
          return '逾期还款';
        }
      },
      htmlFalse:function(arg1){
        var result = arg1.substring(13);
        return 'http://39.96.195.239' + result;
      },
      enableDeferFalse:function(arg1){
        if(arg1==null){
          return "未跑展期流程";
        }else if(arg1==false){
          return "否";
        }else if(arg1==true){
          return "是";
        }
      },
    }
  }
</script>

<style scoped>
  .jiben{
    margin-top: 20px;
    font-size: 14px;
    margin-bottom: 40px;
  }
  .ddd{
    display: inline-block;
    padding: 10px 15px;
    margin-right: 10px;
    margin-left: 5px;
    font-size: 16px;
    color: #ffffff;
    background-color: #118efe;
    border: 1px solid #0f91ff;
    border-radius: 5px;
    cursor: pointer;
    text-decoration: none;
  }
  .ddd:hover{
    background-color: #0abcfe;
    border: 1px solid #0fbeff;
  }
  .listBox{
    display: inline-block;
    float: left;
    padding: 10px 20px;
    margin: 0 5px;
    font-size: 17px;
    color: #000;
    background-color: #dedede;
    border: 1px solid #b0b0b0;
    border-radius: 10px;
    cursor: pointer;
  }
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }
  .listContent{
    clear: both;
    height: 80px;
  }
  .listContent li{
    display: inline-block;
    padding: 10px 15px;
    margin-right: 10px;
    margin-left: 5px;
    font-size: 16px;
    color: #ffffff;
    background-color: #118efe;
    border: 1px solid #0f91ff;
    border-radius: 5px;
    cursor: pointer;
  }
  .listContent li:hover{
    color: #ffffff;
    background-color: #0abcfe;
    border: 1px solid #0fbeff;
  }
  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .addclass{
    background-color: #118efe;
  }
  .la{
    padding: 15px 40px;
    font-size: 20px;
    text-align: center;
    margin: 0 auto;
  }
  .guan{
    padding: 15px 40px;
    font-size: 20px;
    text-align: center;
    margin-left: 40px;
  }
  table,table tr th, table tr td { border:1px solid #838383; }
  table { width: 95%; min-height: 40px; line-height: 40px; text-align: center; border-collapse: collapse; padding:10px 5px;margin-top: 20px}
</style>
