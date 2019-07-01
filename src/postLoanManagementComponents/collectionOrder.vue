<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>催收分单</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="listContent">
      <div class="listBox" v-for="(item,index) in productList" :class="isactive == index ? 'addclass' : ''" @click="xuan(item,index)" :key="index">{{item.productName}}</div>
    </div>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-col :span="12" style="height: 55px;">
        <template>
          到期时间：
          <el-date-picker style="margin-left: 0"
                          v-model="value7"
                          type="datetimerange"
                          align="right"
                          unlink-panels
                          range-separator="至"
                          start-placeholder="开始日期"
                          end-placeholder="结束日期"
                          :picker-options="pickerOptions2"
                          format="yyyy-MM-dd HH:mm:ss"
                          value-format="yyyy-MM-dd HH:mm:ss"
                          @change="logTimeChange2()">
          </el-date-picker>
        </template>
      </el-col>
      <el-col :span="12" style="height: 55px;">
        渠道：
        <el-select v-model="channelNames" placeholder="请选择" multiple style="width: 400px">
          <el-option
            v-for="(item ,index) in channelList"
            :key="index"
            :label="item.subChannelName"
            :value="item.subChannelName">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="7" style="height: 55px;">
        期数：
        <el-select v-model="period" placeholder="请选择">
          <el-option
            v-for="(item ,index) in periodList"
            :key="item.id"
            :label="item.period"
            :value="item.period">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="8" style="height: 55px;">
        新老户：
        <el-select v-model="reBorrow" placeholder="请选择">
          <el-option
            v-for="(item ,index) in reBorrowList"
            :key="index"
            :label="item.value"
            :value="item.id">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;line-height: 40px;text-indent: 15px">
        案件数量:<span style="margin-right: 15px">{{casesNumber}}</span>
        <el-button type="primary"@click="searchContent()" >查询<i class="el-icon-search el-icon--right"></i></el-button>
      </el-col>
      <el-col :span="24">
        <h3 style="color: #606266;font-weight: 400;height: 40px">导入催收员</h3>
        <template>
          <el-alert style="padding-left: 0"
            title="不可重复添加催收员"
            type="warning">
          </el-alert>
        </template>
        <el-form-item style="margin-left: -20px;margin-top: 20px" class="labelList" v-for="(domain, index) in adminIds" :key="index" prop="" :label="'催收员' + index + '：'">
            <div style="margin-left: -100px">
             姓名：
              <el-select v-model="domain.adminId" @change="changeSelect($event,index)">
                <el-option
                  v-for="(item,index2) in collectionList"
                  :key="index2"
                  :label="item.userName"
                  :value="item.id"
                  :disabled="item.disabled">
                </el-option>
              </el-select>&nbsp;&nbsp;&nbsp;
              <el-button type="info"  @click="removeDomain(index,adminIds)" size="medium">删除</el-button>
            </div>
        </el-form-item>
          <el-button v-if="hasPermissionCustom('order:distribution:distribution')" type="primary"  @click="addDomain" size="medium">添加催收员</el-button>
      </el-col>
      <el-col :span="24" style="margin-top: 30px;text-align: center">
        <el-button v-if="hasPermissionCustom('order:distribution:distribution')" type="primary" class="btntn" @click="fendan('ruleForm')">分单</el-button>
        <!--<el-button class="btntn" @click="resetForm('ruleForm')">取消</el-button>-->
      </el-col>
    </el-form>
    <el-dialog
      title="分单提醒"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-col :span="6" v-for="(item,index) in reminderOrderList" :key="index" style="margin-bottom: 15px">{{item.name}}&nbsp;:&nbsp;{{item.adminSize}}单</el-col>
      <el-col :span="24" style="height: 40px;margin-bottom: 20px">
        <el-col :span="6" style="font-size: 16px">共计：{{totalFormCount}}单</el-col>
        <el-col :span="6" style="font-size: 16px">催收员：{{totalReminderCount}}人</el-col>
      </el-col>
      <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="realAssignForm()">确 定</el-button>
      <el-button @click="centerDialogVisible = false">取 消</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      return {
        dis:false,
        productId:null,
        productList:[],
        adminIdsList:[],
        adminNamesList:[],
        channelList:[],
        collectionList:[],
        value7:'',
        startDate:'',
        endDate:'',
        channelNames:[],
        period:'全部期数',
        reBorrow:null,
        casesNumber:'',
        centerDialogVisible:false,
        adminIds:[
          {adminId: null}
        ],
        adminNames:[],
        reminderOrderList:[],
        totalFormCount:'',
        totalReminderCount:'',
        ids_reminder:[],
        ids_form:'',
        ids_reminder_name:[],
        periodList:[],
        reBorrowList:[
          {id:null,value:'全部用户'},
          {id:0,value:'新户'},
          {id:1,value:'老户'},
        ],
        ruleForm: {},
        rules: {},
        pickerOptions2: {
          shortcuts: [{
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          }]
        },
        isactive:0,
        isCollec:false,//是否有没选择的催收员框
      };
    },
    methods: {
      //选择不同产品
      xuan(item,index){
        this.isactive = index;
        this.productId = item.productId;
        this.periodList=[];
        this.adminIds=[{adminId: null}];
        this.adminIdsList=[];
        this.adminNamesList=[];
        this.getCaseNumber(null,null,null,this.startDate,this.startDate,this.productId);
        this.getCollectionList(this.productId);
        this.getBorrowingProductByProductCode(item.productCode);
      },
      //查询所有产品
      getProductList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getProductList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productList=res.data.body;
            this.productId=this.productList[0].productId;
            this.getCaseNumber(null,null,null,this.startDate,this.startDate,this.productId);
            this.getCollectionList(this.productId);
            this.getBorrowingProductByProductCode(this.productList[0].productCode);
          } else if (res.data.msgCd=='ZYCASH-1005') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else if (res.data.msgCd=='SYS00001') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else {
            this.$message.error(res);
          }
        })
      },
      //查询所有渠道
      getChannelList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/channel/admin/channel/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:1,
            pageSize:100
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.channelList=res.data.body.list;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //查询所有催收员
      getCollectionList(id) {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getCollection",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            productId: id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$forceUpdate();
            this.collectionList=res.data.body;
            this.collectionList.forEach(function (item,index) {
              item.disabled=false;
            });
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //查询金融产品下的期数
      getBorrowingProductByProductCode(code) {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/product/getBorrowingProductBymMerchantProductCode",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            productCode: code,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200') {
            this.$forceUpdate();
            if (res.data.body != null) {
              this.periodList.unshift({id: null, period: '全部期数'});
              var temp = res.data.body.list;
              let _this = this;
              for (var i = 0; i < temp.length; i++) {
                var flag = true;
                for (var j = 0; j < _this.periodList.length; j++) {
                  if (temp[i].period == _this.periodList[j].period) {
                    flag = false;
                  }
                }
                if (flag) {
                  _this.periodList.push(temp[i]);
                }
              }
            } else if (res.data.msgCd == 'ZYCASH-1009') {
              this.$message.error(res.data.msgInfo);
            } else {
              this.$message.error(res);
            }
          }
        })
      },
      //条件查询规则集列表
      searchContent(){
        if (this.period == '全部期数') {
          this.getCaseNumber(this.channelNames,null,this.reBorrow,this.startDate,this.endDate,this.productId);
        } else {
          this.getCaseNumber(this.channelNames,this.period,this.reBorrow,this.startDate,this.endDate,this.productId);
        }
      },
      /**
       * 查询案件数量
       * @param data1 渠道
       * @param data2 期数
       * @param data3 新老户
       * @param data4 开始时间
       * @param data5 结束时间
       * @param data6 产品id
       */
      getCaseNumber(data1,data2,data3,data4,data5,data6) {
        var data = {
          'channelNames':data1,
          'period':data2,
          'reBorrow':data3,
          'startDate':data4,
          'endDate':data5,
          'productId':data6,
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/find",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(data),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.casesNumber=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //封装姓名
      changeSelect(vId,index){
        let index2 = '';
        this.collectionList.forEach(function (item,index3) {
          if (item.id==vId) {
            index2=index3;
          }
        });
        this.$forceUpdate();
        let obj = {};
        obj = this.collectionList.find((item)=>{
          return item.id === vId;
        });
        this.adminIdsList[index]=obj.id;
        this.adminNamesList[index]=obj.userName;
        this.collectionList[index2].disabled=!this.collectionList[index2].disabled;//最新选择的催收员状态取反
      },
      //添加催收员
      addDomain() {
        this.adminIds.push({
          adminId: null,
        });
      },
      //删除催收员
      removeDomain(index,list) {
        let adminId = this.adminIds[index].adminId;
        let index2 = '';
        this.collectionList.forEach(function (item,index3) {
          if (item.id==adminId) {
            index2=index3;
          }
        });
        list.splice(index, 1);
        this.isCollec = false;
        this.collectionList[index2].disabled=false;
      },
      //分单提醒
      fendan(){
        let _this = this;
        this.adminIds.forEach(function (item,index) {
          if (item.adminId == null) {
            _this.isCollec = true;
          }
        });
        if (this.isCollec == true) {
          this.$message({
            showClose: true,
            message: '请选择催收员',
            type: 'warning'
          });
        } else if (this.casesNumber==0) {
          this.$message({
            showClose: true,
            message: '没有可分订单',
            type: 'warning'
          });
        } else {
          this.centerDialogVisible=true;
          if (this.period == '全部期数') {
            _this.period = null;
          }
          var data = {
            'channelNames':this.channelNames,
            'period':this.period,
            'reBorrow':this.reBorrow,
            'startDate':this.startDate,
            'endDate':this.endDate,
            'productId':this.productId,
            'adminIds':this.adminIdsList,
            'adminNames':this.adminNamesList,
          };
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/order/admin/borrowing/assignForm",
            headers:{
              'Content-Type':'application/json',
              'Authorization': localStorage.token
            },
            data:JSON.stringify(data),
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              this.reminderOrderList=res.data.body.dataMap.reminders;
              this.totalFormCount=res.data.body.dataMap.totalFormCount;
              this.totalReminderCount=res.data.body.dataMap.totalReminderCount;
              this.ids_form=res.data.body.ids_form;
              this.ids_reminder_name=res.data.body.ids_reminder_name;
              this.ids_reminder=res.data.body.ids_reminder;
            }else if(res.data.msgCd=='ZYCASH-1009'){
              this.$message.error(res.data.msgInfo);
            }
            else {
              this.$message.error(res);
            }
          })
        }
      },
      //真正分单
      realAssignForm() {
        var data = {
          'ids_form':this.ids_form,
          'ids_reminder':this.ids_reminder,
          'ids_reminder_name':this.ids_reminder_name,
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/realAssignForm",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(data),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.centerDialogVisible=false;
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            //将案件数量归零，催收员归零
            this.getCaseNumber(this.channelNames,this.period,this.reBorrow,this.startDate,this.endDate,this.productId);
            this.adminIds = [{adminId: null}];
            //所有催收员可选
            this.collectionList.forEach(function (item,index) {
              item.disabled=false;
            });
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //时间筛选2
      logTimeChange2(){
        if(this.value7==''||this.value7==null){
          this.startDate=this.startDate;
          this.endDate=this.startDate;
        }else {
          var startTime=this.value7[0];
          var endTime=this.value7[1];
          this.startDate=startTime;
          this.endDate=endTime;
        }
      },
    },
    mounted:function () {
      this.startDate=this.dateFormatCustom(new Date(new Date().getFullYear(), new Date().getMonth(), new Date().getDate(), 0, 0, 0));
      this.endDate=this.startDate;
      this.value7=[this.startDate,this.endDate];
      this.getProductList();
      this.getChannelList();
    }
  }
</script>

<style scoped>
  .btntn {
    width: 120px;
    height: 50px;
    font-size: 18px;
    letter-spacing: 2px;
    margin: 0 20px;
  }
  .content{
    padding-left: 200px;
    padding-top: 80px;
  }
  .demo-ruleForm{
    width: 80%;
    text-align: left;
  }
  .fs-16{
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .listContent{
    width: 100%;
    height: 80px;
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
    margin-bottom: 10px;
  }
  .addclass{
    background-color: #118efe;
  }
</style>
