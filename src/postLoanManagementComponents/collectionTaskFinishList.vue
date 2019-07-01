<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>催回列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-col :span="6" style="height: 55px;">
        订单状态：
        <el-select v-model="status" placeholder="请选择">
          <el-option
            v-for="item in statusList"
            :key="item.classifyId"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        <template>手机号码：
          <el-input class="searchContent" placeholder="用户手机号" v-model="mobile" clearable></el-input>
        </template>
      </el-col>
      <el-col :span="12" style="height: 55px;">
        <template>
          下单时间：
          <el-date-picker style="margin-left: 0px"
                          v-model="value5"
                          type="datetimerange"
                          align="right"
                          unlink-panels
                          range-separator="至"
                          start-placeholder="开始日期"
                          end-placeholder="结束日期"
                          :picker-options="pickerOptions2"
                          format="yyyy-MM-dd HH:mm:ss"
                          value-format="yyyy-MM-dd HH:mm:ss"
                          @change="logTimeChange1()">
          </el-date-picker>
        </template>
      </el-col>
      <el-col :span="12" style="height: 55px;">
        <template>
          放款时间：
          <el-date-picker style="margin-left: 0"
                          v-model="value6"
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
      <el-col :span="6" style="height: 55px;">
        逾期天数：
        <el-input class="searchContent" placeholder="逾期天数" v-model="repaymentOverdueDay" clearable></el-input>
      </el-col>
      <el-col :span="5.5" style="height: 55px;">
        所属平台：
        <el-select v-model="productId" placeholder="请选择" @change="selectChange">
          <el-option
            v-for="item in productList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        群组：
        <el-select v-model="groupId" placeholder="请选择" @change="selectChange2">
          <el-option
            v-for="item in groupList"
            :key="item.id"
            :label="item.groupName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        催收员：
        <el-select v-model="adminId" placeholder="请选择">
          <el-option
            v-for="item in collectionList"
            :key="item.id"
            :label="item.userName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-col>
      <el-button type="primary" id="searchBtn" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="orderId"
          label="订单ID"
          width="250">
        </el-table-column>
        <el-table-column
          fixed
          prop="name"
          label="姓名"
          min-width="80">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="borrowingPeriod"
          label="借款期限"
          width="120">
        </el-table-column>
        <el-table-column
          prop="borrowingCapital"
          label="合同金额"
          width="120">
        </el-table-column>
        <el-table-column
          prop="status"
          label="订单状态"
          :formatter="statusFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          prop="borrowingInterest"
          label="综合费用"
          width="100">
        </el-table-column>
        <el-table-column
          prop="borrowingPaymentAmount"
          label="到账金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="repaymentOverdueDay"
          label="逾期天数"
          width="100">
        </el-table-column>
        <el-table-column
          prop="repaymentPenaltyInterest"
          label="逾期费用"
          width="100">
        </el-table-column>
        <el-table-column
          prop="repaymentPenaltyInterest"
          label="应还金额"
          :formatter="shouldFormatter"
          width="100">
        </el-table-column>
        <el-table-column
          prop="defer"
          label="是否展期"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.defer == 1 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.defer == 1 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="repaymentDefer"
          label="展期还款金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="reBorrow"
          label="展期次数"
          width="80">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="应用"
          width="80">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="申请时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="borrowingPaymentDate"
          label="放款时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="repaymentEndDate"
          label="预计还款时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="parentChannelName"
          label="主渠道"
          width="200">
        </el-table-column>
        <el-table-column
          prop="childrenChannelName"
          label="子渠道"
          width="200">
        </el-table-column>
        <el-table-column
          prop="reBorrow"
          label="用户标识"
          :formatter="reBorrowFormatter"
          width="100">
        </el-table-column>
        <el-table-column
          prop="collectorName"
          label="催收员"
          width="150">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="100">
          <template slot-scope="scope">
            <el-button v-if="hasPermissionCustom('order:reclaim:detail')" @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
          </template>
        </el-table-column>
      </el-table>
    </template>
    <div class="block">
      <el-pagination class="paginationBox"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :unique-opened="true"
        :current-page="pageNum"
        :page-sizes="pageSizes"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="proTotal">
      </el-pagination>
    </div>
  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
      //查询所有产品
      getProduct() {
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
            this.productList.unshift({productId: null, productName: '全部产品'});
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
      //查询所有群组
      getGroupList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/collection/getGroup",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.groupList=res.data.body.operationGroupList;
            this.operationAdminGroup=res.data.body.operationAdminGroup;
            if (this.operationAdminGroup.groupRole == 1) {
              this.groupId = this.operationAdminGroup.groupId;
              this.getCollectionList(this.operationAdminGroup.groupRole,this.operationAdminGroup.adminId);
              this.getProductList(1,30,this.status,null,null,null,null,null,null,null,this.groupId,this.adminId);
            } else if (this.operationAdminGroup.groupRole == 2) {
              this.groupId = this.operationAdminGroup.groupId;
              this.adminId = this.operationAdminGroup.adminId;
              this.getCollectionList(this.operationAdminGroup.groupRole,this.operationAdminGroup.adminId);
              this.getProductList(1,30,this.status,null,null,null,null,null,null,null,this.groupId,this.adminId);
            } else {
              this.groupList.unshift({id: null, groupName: '全部群组'});
              this.getProductList(1,30,this.status,null,null,null,null,null,null,null,this.groupId,this.adminId);
            }
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //二级联动：选择平台后加载对应催收员
      selectChange(){
        if (this.operationAdminGroup.groupRole != 1 && this.operationAdminGroup.groupRole != 2) {
          this.groupId=null;
          this.getCollectionList();
        }
      },
      //二级联动：选择群组后加载对应催收员
      selectChange2(){
        this.productId=null;
        this.collectionList=[];
        this.getCollectionList();
      },
      //查询所有催收员
      getCollectionList(groupRole,adminId) {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/collection/getCollectionRole",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            productId: this.productId,
            groupId: this.groupId,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.collectionList=res.data.body;
            if (groupRole == 1) {
              this.collectionList.unshift({id:null,userName:"全部"});
            } else if (groupRole == 2) {
              //将整个组中的催收员与本账号匹配一下
              let obj = {};
              obj = this.collectionList.find((item)=>{
                return item.id === adminId;
              });
              this.collectionList = [];
              this.collectionList.push(obj);
              // this.adminId=adminId;
            } else {
              this.collectionList.unshift({id:null,userName:"全部"});
            }
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //条件查询
      searchContent(data){
        this.getProductList(this.pageNum,this.pageSize,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
          this.repaymentPaymentEndDate,this.repaymentOverdueDay,this.productId,this.groupId,this.adminId);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
          this.repaymentPaymentEndDate,this.repaymentOverdueDay,this.productId,this.groupId,this.adminId);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.pageSize,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
          this.repaymentPaymentEndDate,this.repaymentOverdueDay,this.productId,this.groupId,this.adminId);
      },
      /**
       * 催收已分订单列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 订单状态
       * @param data4 手机号码
       * @param data5 下单开始时间
       * @param data6 下单结束时间
       * @param data7 放款开始时间
       * @param data8 放款结束时间
       * @param data9 逾期天数
       * @param data10 所属平台
       * @param data11 群组id
       * @param data12 催收员
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9,data10,data11,data12){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/findCollectionOrder",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            status: data3,
            mobile: data4,
            orderStartDate: data5,
            orderEndDate: data6,
            repaymentPaymentStartDate: data7,
            repaymentPaymentEndDate: data8,
            repaymentOverdueDay: data9,
            productId: data10,
            groupId: data11,
            adminId: data12,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body.list;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //过滤状态字段
      statusFormatter(row){
        let status = row.status;
        if(status === 0){
          return '待机审 '
        } else if (status === 1){
          return '机器审核中'
        } else if (status === 2){
          return '审核拒绝'
        } else if (status === 3){
          return '人工审核'
        } else if (status === 4){
          return '待放款'
        } else if (status === 5){
          return '放款中'
        } else if (status === 6){
          return '放款失败'
        } else if (status === 7){
          return '取消'
        } else if (status === 8){
          return '放款成功，待还款'
        } else if (status === 9){
          return '还款确认中'
        } else if (status === 10){
          return '正常还款 '
        } else if (status === 11){
          return '逾期未还'
        } else if (status === 12){
          return '坏账'
        } else if (status === 13){
          return '逾期已还'
        }
      },
      //应还金额字段
      shouldFormatter(row){
        let repaymentCapital = row.repaymentCapital;
        let repaymentOverdueFee = row.repaymentOverdueFee;
        let repaymentPenaltyInterest = row.repaymentPenaltyInterest;
        return repaymentCapital + repaymentOverdueFee + repaymentPenaltyInterest;
      },
      //过滤用户标识字段
      reBorrowFormatter(row){
        let reBorrow = row.reBorrow;
        if(reBorrow === false){
          return '新户'
        } else if (reBorrow === true){
          return '老户'
        } else{
          return '---'
        }
      },
      //下单时间筛选
      logTimeChange1(){
        if(this.value5!=''||this.value5!=null){
          var startTime=this.value5[0];
          var endTime=this.value5[1];
          this.orderStartDate=startTime;
          this.orderEndDate=endTime;
        }
      },
      //放款时间筛选
      logTimeChange2(){
        if(this.value6!=''||this.value6!=null){
          var startTime=this.value6[0];
          var endTime=this.value6[1];
          this.repaymentPaymentStartDate=startTime;
          this.repaymentPaymentEndDate=endTime;
        }
      },
      //详情
      detailProduct(row){
        let id=row.customerId;
        let orderId=row.orderId;
        this.$router.push({
          path: `/orderDetailCollectionFinish/${id}/${orderId}`,
        });
      },
      //紧急联系人
      Contacts(row){
        let id=row.customerId;
        this.$router.push({
          path: `/addressListEmergency/${id}`,
        });
      },
      //通讯录
      mailList(row){
        let id=row.customerId;
        this.$router.push({
          path: `/addressList/${id}`,
        });
      },
    },
    mounted:function () {
      this.getProduct();
      this.getGroupList();
      // this.getProductList(1,30,this.status,null,null,null,null,null,null,null,null);
    },
    data() {
      return {
        productList:[],
        groupList:[],
        operationAdminGroup:'',
        collectionList:[],
        statusList: [
          {classifyId:13,classifyName:"逾期已还"},
        ],
        tableData:[],
        status:13,
        mobile:null,
        orderStartDate:null,
        orderEndDate:null,
        repaymentPaymentStartDate:null,
        repaymentPaymentEndDate:null,
        repaymentOverdueDay: null,
        productId:null,
        groupId:null,
        adminId: null,
        value5:'',
        value6:'',
        pageNum: null,
        proTotal:null,
        pageSize:30,
        pageSizes:[20,30,50],
        nowPageSizes:30,
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
