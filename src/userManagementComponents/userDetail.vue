<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/userProductList' }">用户列表</el-breadcrumb-item>
      <el-breadcrumb-item>用户详情</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="listContent">
      <div class="listBox" v-for="(item,index) in productList" :class="isactive == index ? 'addclass' : ''" @click="fen(item,index)">{{item.productName}}</div>
    </div>
    <div class="line" ></div>
    <div class="listContent">
      <router-link :to="'/jiben/'+this.id" tag="li">基本信息</router-link>
      <router-link v-if="hasPermissionCustom('user:black:hitList')" :to="'/fenxian/' + this.id" tag="li">风险命中列表</router-link>
      <router-link :to="'/yunying/' + this.id" tag="li">运营商通讯录比对</router-link>
      <template v-if="this.tianjiReport && hasPermissionCustom('user:black:tiji')">
      <a :href="this.tianjiReport.tianjiUrl | htmlFalse" target="_blank" class="ddd">天机报告</a>
      </template>
      <a href="http://www.baidu.com" target="_blank" class="ddd">支付宝报告</a>
      <router-link :to="'/yonghu/' + this.id" tag="li">用户催收记录</router-link>
      <router-link :to="'/dingdan/' + this.id" tag="li">订单记录</router-link>
      <router-link :to="'/fangkuan/' + this.id" tag="li">放款记录</router-link>
      <router-link :to="'/huankuan/' + this.id" tag="li">还款记录</router-link>
    </div>
    <router-view/>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
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
        authorizationStatus:[],
        basicInfo:{},
        tableData:[],
        electValue:0,
        isactive:0,
      };
    },
    methods: {
      //用户基本信息
      getUserDetail(id) {
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
            if (this.tianjiReport != null) {
              this.tianjiReport.tianjiUrl = res.data.body.tianjiReport.html;
            }
            this.idCard = res.data.body.idCard;
            this.idFace = res.data.body.idFace;
            this.linkMan = res.data.body.linkMan;
            this.authorizationStatus = res.data.body.authorizationStatus;
            this.basicInfo = res.data.body.basicInfo;
            this.selectProduct(this.id,this.productList);
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //切换产品
      fen(item,index){
        this.isactive = index;
        this.id=item.id;
        let id=this.id;
        this.getUserDetail(this.id);
        this.$router.push({
          name: 'jiben', params: { id }
        });
      },
      //默认选择产品
      selectProduct(customerId,productList){
        if (productList != null && productList.size > 0) {
          for(var i=0;i<productList.length;i++){
            var pro = productList[i];
            if (pro.id == customerId) {
              this.isactive = i;
            }
          }
        }
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getUserDetail(this.id);
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
      }
    }
  }
</script>

<style scoped>
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
    padding: 15px 30px;
    margin: 0 5px;
    font-size: 17px;
    color: #000;
    background-color: #ffffff;
    border: 1px solid #dcdfe6;
    cursor: pointer;
  }
  .listBox:hover{
    border: 1px solid #b5b5b5;
    -moz-box-shadow:2px 6px 15px #b5b5b5;
    -webkit-box-shadow:2px 6px 15px #b5b5b5;
    box-shadow:2px 6px 15px #b5b5b5;
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
    color: #67c23a;
    background: #f0f9eb;
    border-color: #c2e7b0;
  }
  .line{
    width: 95%;
    height: 1px;
    background-color: #b0b0b0;
    margin-bottom: 30px;
    margin-left: 5px;
  }
</style>
