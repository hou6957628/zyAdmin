<template>
  <div>
    <div style="width: 1000px;margin: 0 auto;">
    <el-row style="text-align: center;margin: 0 auto">
      <el-col style="width: 200px;margin-left: 5px" :span="4">
        <img class="ID1"  style="height: auto;width: 200px" :src="this.idCard.front"/>
      </el-col>
      <el-col style="width: 200px;margin-left: 50px" :span="4">
        <img class="ID2" style="height: auto;width: 200px" :src="this.idCard.back"/>
      </el-col>
      <el-col style="width: 200px;margin-left: 50px" :span="4">
        <img class="Face" style="height: auto;width: 200px" :src="this.idFace.image"/>
      </el-col>
    </el-row>
    <el-row style="text-align: center;margin: 0 auto">
    <el-col :span="4" style="width: 200px;text-align: center;margin-left: 5px;z-index: 1">身份证正面</el-col>
    <el-col :span="4" style="width: 200px;text-align: center;margin-left: 50px;z-index: 1">身份证背面</el-col>
    <el-col :span="4" style="width: 200px;text-align: center;margin-left: 50px;z-index: 1">人脸识别照片</el-col>
    </el-row>
    </div>
    <div class="jiben">
      <h3>基本信息</h3>
      <table >
        <tr>
          <td>用户ID：{{this.cusCustomer.id}}</td><td> 手机号：{{this.cusCustomer.mobile}}</td><td>  主渠道：{{this.cusCustomer.channelName==null?'无':this.cusCustomer.channelName}}</td>
          <td>新户老户：{{this.cusCustomer.reBorrow==1?'老户':'新户'}}</td>
          <td>注册时间：{{this.cusCustomer.createDate}}</td><td>所属平台：{{this.cusCustomer.productName}}</td>
        </tr>
        <tr>
          <td>姓名：{{this.cusCustomer.realName==null?'--':this.cusCustomer.realName}}</td><td>身份证号：{{this.cusCustomer.cardNumber==null?'--':this.cusCustomer.cardNumber}}</td>
          <td>是否是黑名单：{{this.cusCustomer.isBlackList==true?'是':'否'}}</td>
          <td>年龄：{{this.idCard.age==null?'--':this.idCard.age}}</td><td>婚姻状况：{{this.basicInfo.marital==null?'--':this.basicInfo.marital}}</td>
          <td>芝麻分：{{this.zhimaFen==null?'--':this.zhimaFen.zmScore}}</td>
          <td>西瓜分：{{this.tianjiReport==null?'--':this.tianjiReport.xgScore}}</td>
        </tr>
        <tr>
          <td>性别：{{this.idCard.gender | genderFalse}}</td><td>身份证有效期：{{this.idCard.validDate==null?'--':this.idCard.validDate}}</td>
          <td>身份证住址：{{this.idCard.address==null?'--':this.idCard.address}}</td><td>民族：{{this.idCard.race==null?'--':this.idCard.race}}</td>
        </tr>
      </table>
      <h3>设备信息</h3>
      <table >
        <tr>
          <td>手机型号：{{this.cusCustomer.device==null?'--':this.cusCustomer.device}}</td>
          <td>手机系统版本号：{{this.cusCustomer.systemVersion==null?'--':this.cusCustomer.systemVersion}}</td>
          <td>APP名称：{{this.cusCustomer.productName}}</td>
        </tr>
      </table>
      <h3>个人信息</h3>
      <table >
        <tr>
          <td>学历：{{this.basicInfo.education}}</td><td>婚姻情况：{{this.basicInfo.marital}}</td><td>居住地址：{{this.basicInfo.addressDetail==null?'--':this.basicInfo.addressDetail}}</td>
        </tr>
      </table>
      <h3>联系人</h3>
      <table >
        <template v-for="(item,index) in this.linkMan">
          <tr>
            <td>联系人{{index}}：{{item.name}}</td><td>联系人借款关系：{{item.relation}}</td><td>手机号：{{item.phoneNum}}</td>
            <td rowspan="2" v-if="index==0"><el-button @click="addressList">手机通讯录</el-button></td>
          </tr>
        </template>
      </table>
      <h3>绑卡信息</h3>
      <table >
        <tr v-for="(domain, index) in bankCard" :key="index">
          <td>银行名称：{{domain.bankName}}</td><td>卡号：{{domain.cardNumber}}</td><td>预留电话：{{domain.mobile}}</td><td>类型：放款卡</td><td>绑卡时间：{{domain.createDate}}</td>
        </tr>
      </table>
      <h3>认证信息</h3>
      <table >
        <template v-for="(item,index) in this.authorizationStatus">
          <!--<tr v-if="index==0"><td>身份证：{{item.authorizationStatus}}</td><td>身份证认证时间：{{item.authorizationStatus=='已认证'?item.updateDate:''}}</td></tr>-->
          <!--<tr v-if="index==1"><td>人脸识别认证：{{item.authorizationStatus}}</td><td>人脸识别认证时间：{{item.authorizationStatus=='已认证'?item.updateDate:''}}</td></tr>-->
          <!--<tr v-if="index==2"><td>运营商手机认证：{{item.authorizationStatus}}</td><td>运营商手机认证时间：{{item.authorizationStatus=='已认证'?item.updateDate:''}}</td></tr>-->
          <!--<tr v-if="index==3"><td>芝麻分认证：{{item.authorizationStatus}}</td><td>芝麻分认证时间：{{item.authorizationStatus=='已认证'?item.updateDate:''}}</td></tr>-->
          <!--<tr v-if="index==4"><td>绑卡：{{item.authorizationStatus}}</td><td>绑卡时间：{{item.authorizationStatus=='已认证'?item.updateDate:''}}</td></tr>-->
          <tr><td>{{item.templateName}}：{{item.authorizationStatus}}</td><td>认证时间：{{item.authorizationStatus=='已认证'?item.updateDate:''}}</td></tr>
        </template>
      </table>
      <el-button-group style="margin: 0 auto;width: 500px;display: block;margin-top: 40px;">
        <el-button v-if="!this.cusCustomer.isBlackList && (this.hasPermissionCustom('user:customer:setBlack') || this.hasPermissionCustom('user:black:add'))" class="la"
                   @click="addBlack()" type="danger">拉黑</el-button>
        <el-button v-if="this.cusCustomer.isBlackList && (this.hasPermissionCustom('user:customer:setBlack') || this.hasPermissionCustom('user:black:delete'))" class="la"
                   @click="removeBlack()" type="danger">移除黑名单</el-button>
        <el-button class="guan" @click="resetForm()" style="margin-left: 40px">关闭</el-button>
      </el-button-group>
    </div>
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
        tianjiReport:{},
        idCard:{
          "front":"http://39.96.195.239/images/zhan.jpg",
          "back":"http://39.96.195.239/images/zhan.jpg",
        },
        idFace:{
          "image":"http://39.96.195.239/images/zhan.jpg",
        },
        id:null,
        linkMan:[
          {
            name:'',
            relation:"",
            phoneNum:""
          }
        ],
        authorizationStatus:[],
        basicInfo:{
          marital:'',
          education:'',
        },
      };
    },
    methods: {
      //通讯录
      addressList(){
        let id=this.id;
        this.$router.push({
          path: `/addressList1/${id}`,
        });
      },
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
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
            if (res.data.body.idCard != undefined) {
              this.idCard = res.data.body.idCard;
            }
            if (res.data.body.idFace != null) {
              this.idFace = res.data.body.idFace;
            }
            this.linkMan = res.data.body.linkMan;
            this.authorizationStatus = res.data.body.authorizationStatus;
            if (res.data.body.basicInfo != null) {
              this.basicInfo = res.data.body.basicInfo;
              if (res.data.body.basicInfo.marital==1) {
                this.basicInfo.marital='已婚';
              } else if (res.data.body.basicInfo.marital==2) {
                this.basicInfo.marital='未婚';
              } else if (res.data.body.basicInfo.marital==3) {
                this.basicInfo.marital='离异';
              } else {
                this.basicInfo.marital='--';
              }
              if (res.data.body.basicInfo.education==0) {
                this.basicInfo.education='初中及以下';
              } else if (res.data.body.basicInfo.education==1) {
                this.basicInfo.education='高中';
              } else if (res.data.body.basicInfo.education==2) {
                this.basicInfo.education='专科';
              } else if (res.data.body.basicInfo.education==3) {
                this.basicInfo.education='本科';
              } else if (res.data.body.basicInfo.education==4) {
                this.basicInfo.education='研究生';
              } else if (res.data.body.basicInfo.education==5) {
                this.basicInfo.education='博士生';
              } else if (res.data.body.basicInfo.education==6) {
                this.basicInfo.education='留学生';
              } else {
                this.basicInfo.education='--';
              }
            }
            var _this=this;
            this.linkMan.forEach(function (item,index) {
              if (item.relation==0) {
                _this.linkMan[index].relation='父母';
              } else if (item.relation==1) {
                _this.linkMan[index].relation='配偶';
              } else if (item.relation==2) {
                _this.linkMan[index].relation='兄弟姐妹';
              } else if (item.relation==3) {
                _this.linkMan[index].relation='同事';
              } else if (item.relation==4) {
                _this.linkMan[index].relation='同学';
              } else if (item.relation==5) {
                _this.linkMan[index].relation='朋友';
              }
            });
            _this.authorizationStatus.forEach(function (item,index) {
              if (item.authorizationStatus==0) {
                _this.authorizationStatus[index].authorizationStatus='未认证';
              } else if (item.authorizationStatus==1) {
                _this.authorizationStatus[index].authorizationStatus='认证中';
              } else if (item.authorizationStatus==2) {
                _this.authorizationStatus[index].authorizationStatus='已认证';
              } else if (item.authorizationStatus==3) {
                _this.authorizationStatus[index].authorizationStatus='已失效';
              } else if (item.authorizationStatus==4) {
                _this.authorizationStatus[index].authorizationStatus='认证失败';
              }
            });
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
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
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getUserDetail(this.id);
    },
    filters:{
      genderFalse:function(arg1){
        if(arg1==true){
          var result = "女";
          return result;
        }else if(arg1==false){
          var result = "男";
          return result;
        } else {
          return '--';
        }
      }
    }
  }
</script>

<style scoped >
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
  .ID1{
    transition: All 0.4s ease-in-out;
    -webkit-transition: All 0.4s ease-in-out;
    -moz-transition: All 0.4s ease-in-out;
    -o-transition: All 0.4s ease-in-out;
    z-index: 1000;
    cursor: pointer;
  }
  .ID1:hover{
    transform: scale(3);
    -webkit-transform: scale(3);
    -moz-transform: scale(3);
    -o-transform: scale(3);
    -ms-transform: scale(3);
    z-index: 1000;
  }
  .ID2{
    transition: All 0.4s ease-in-out;
    -webkit-transition: All 0.4s ease-in-out;
    -moz-transition: All 0.4s ease-in-out;
    -o-transition: All 0.4s ease-in-out;
    z-index: 1000;
    cursor: pointer;
  }
  .ID2:hover{
    transform: scale(3);
    -webkit-transform: scale(3);
    -moz-transform: scale(3);
    -o-transform: scale(3);
    -ms-transform: scale(3);
    z-index: 1000;
  }
  .Face{
    transition: All 0.4s ease-in-out;
    -webkit-transition: All 0.4s ease-in-out;
    -moz-transition: All 0.4s ease-in-out;
    -o-transition: All 0.4s ease-in-out;
    z-index: 1000;
    cursor: pointer;
  }
  .Face:hover{
    transform: scale(3);
    -webkit-transform: scale(3);
    -moz-transform: scale(3);
    -o-transform: scale(3);
    -ms-transform: scale(3);
    z-index: 1000;
  }
</style>
