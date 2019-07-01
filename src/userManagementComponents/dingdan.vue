<template>
  <div>
    <div class="jiben">
      <h3>总平台信息</h3>
      <table >
        <tr><td>注册平台数：{{this.count.registerNum}}</td><td>所有平台提交申请数：{{this.count.allBorrowingNum}}</td><td>所有平台总放款数：{{this.count.allCPSNum}}</td>
          <td>所有平台拒绝次数：{{this.count.allRefuseNum}}</td><td>当前平台申请次数：{{this.count.BorrowingNum}}</td><td>当前平台放款次数：{{this.count.CPSNum}}</td></tr>
        <tr><td>所有平台逾期次数：{{this.count.allOverdueNum}}</td><td>所有平台催收次数：{{this.count.allCollectionNum}}</td><td>当前平台逾期次数：{{this.count.overdueNum}}</td>
          <td>当前平台催收次数：{{this.count.collectionNum}}</td><td>是否是黑名单：{{this.count.isBlackList | typeFalse}}</td></tr>
      </table>
      <div v-for="(keys,val) in this.orders">
        <h3>{{val}}</h3>
        <div v-for="(key,val) in keys">
          <table>
            <tr><td>订单号：{{key.orderId}}</td><td>手机号：{{key.mobile}}</td><td>渠道：{{key.parentChannelName}}</td><td>订单状态：{{key.status | statusFalse}}</td>
              <td>新户老户：{{key.reBorrow | reborrowFalse}}</td><td>所属平台：{{key.productName}}</td><td>申请时间：{{key.createDate}}</td></tr>
            <tr><td>放款时间：{{key.borrowingPaymentDate==null?'无':key.borrowingPaymentDate}}</td>
              <td>预计还款时间：{{key.repaymentEndDate==null?'无':key.repaymentEndDate}}</td>
              <td>实际还款时间：{{key.repaymentPaymentDate==null?'无':key.repaymentPaymentDate}}</td>
              <td>借款金额(元)：{{key.borrowingCapital}}</td><td>期限：{{key.borrowingPeriod}}</td><td>是否逾期：{{key.repaymentOverdueDay | yuqi}}</td>
              <td>逾期天数：{{key.repaymentOverdueDay==null?'0':key.repaymentOverdueDay}}</td></tr>
            <tr><td>应还利息（元）：{{key.borrowingInterest==null?'0':key.borrowingInterest}}</td>
              <td> 罚息（元）：{{key.repaymentPenaltyInterest==null?'0':key.repaymentPenaltyInterest}}</td>
              <td>滞纳金（元）：{{key.repaymentOverdueFee==null?'0':key.repaymentOverdueFee}}</td>
              <td>应还总还金额（元）：{{key.repaymentCapital + key.repaymentOverdueFee + key.repaymentPenaltyInterest}}</td>
              <td>是否可展期：{{key.enableDefer | enableDeferFalse}}</td><td>展期应还金额：{{key.repaymentDefer==null?'0':key.repaymentDefer}}</td>
              </tr>
            <tr><td>实际还款金额（元）：{{key.repaymentPaymentAmount==null?'0':key.repaymentPaymentAmount}}</td>
              <td>减免金额：{{key.repaymentDiscountAmount==null?'0':key.repaymentDiscountAmount}}</td><td>是否展期：{{key.defer==1?'是':'否'}}</td>
              <td>展期实际还款金额：{{key.repaymentDeferPayment==null?'0':key.repaymentDeferPayment}}</td><td>催收员：{{key.collectorName==null?'无':key.collectorName}}</td></tr>
          </table>
        </div>
      </div>
      <el-button-group style="margin: 0 auto;width: 500px;display: block;margin-top: 40px;">
        <el-button class="la" type="danger">拉黑</el-button>
        <el-button class="guan" style="margin-left: 40px">关闭</el-button>
      </el-button-group>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        count:[],
        orders:[],
      };
    },
    methods: {
      //用户基本信息
      getCollection(id) {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/user_center/admin/borrowing_form/get",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.count = res.data.body.count;
            this.orders = res.data.body.orders;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getCollection(this.id);
    },
    filters:{
      enableDeferFalse:function(arg1){
        if(arg1==null){
          return "未跑展期流程";
        }else if(arg1==false){
          return "否";
        }else if(arg1==true){
          return "是";
        }
      },
      reborrowFalse:function(arg1){
        if(arg1==true){
          return "老户";
        }else {
          return "新户";
        }
      },
      typeFalse:function(arg1){
        console.log(arg1);
        if(arg1==true){
          var result = "是";
          return result;
        }else if(arg1==false){
          var result = "否";
          return result;
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
      yuqi:function(arg1){
        if(arg1 > 0){
          var result = "是";
          return result;
        }else{
          var result = "否";
          return result;
        }

      }
    }
  }
</script>

<style scoped>
  .jiben{
    margin-top: 20px;
    font-size: 14px;
    margin-bottom: 40px;
  }
  h3{
    margin-top: 15px;
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
