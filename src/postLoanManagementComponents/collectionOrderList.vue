<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/afterLoanManagement' }">催收已分订单管理</el-breadcrumb-item>
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
      <el-col :span="12" style="height: 55px;">
        <template>
          预计还款时间：
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
                          @change="logTimeChange3()">
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
      <el-button id="cancelBtn" @click="cancelContent()" slot="append" icon="el-icon-delete">取消分单</el-button>
    </div>
    <template>
      <el-table
        ref="multipleTable"
        :data="tableData"
        @selection-change="handleSelectionChange"
        highlight-current-row
        border
        style="width: 98%">
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
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
          width="100">
        </el-table-column>
        <el-table-column
          prop="borrowingCapital"
          label="借款金额"
          width="100">
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
          width="120">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="应用"
          width="100">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="下单时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="borrowingPaymentDate"
          label="放款时间"
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
          prop="distributDate"
          label="分配时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="childrenChannelName"
          label="分配状态"
          :formatter="distributFormatter"
          width="100">
        </el-table-column>
        <el-table-column
          prop="outSourceDate"
          label="转出时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="内催天数"
          width="100">
        </el-table-column>
        <el-table-column
          prop="outSourceDate"
          label="委外天数"
          width="100">
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
            <el-button @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
            <el-button @click="collectionLog(scope.row)" type="text" size="medium">催记</el-button>
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
      //二级联动：选择平台后加载对应催收员
      selectChange(){
        this.adminId=null;
        this.getCollectionList();
      },
      //查询所有催收员
      getCollectionList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getCollection",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            productId: this.productId,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.collectionList=res.data.body;
            this.collectionList.unshift({id:null,roleName:"全部"});
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
        if (this.status==null) {
          this.getProductList(this.pageNum,this.pageSize,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
            this.repaymentPaymentEndDate,this.orderRepaymentStartDate,this.orderRepaymentEndDate,this.repaymentOverdueDay,this.productId,this.adminId,1);
        } else {
          this.getProductList(this.pageNum,this.pageSize,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
            this.repaymentPaymentEndDate,this.orderRepaymentStartDate,this.orderRepaymentEndDate,this.repaymentOverdueDay,this.productId,this.adminId,null);
        }
      },
      //每页显示多少条
      handleSizeChange(val) {
        if (this.status==null) {
          this.getProductList(this.pageNum,val,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
            this.repaymentPaymentEndDate,this.orderRepaymentStartDate,this.orderRepaymentEndDate,this.repaymentOverdueDay,this.productId,this.adminId,1);
        } else {
          this.getProductList(this.pageNum,val,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
            this.repaymentPaymentEndDate,this.orderRepaymentStartDate,this.orderRepaymentEndDate,this.repaymentOverdueDay,this.productId,this.adminId,null);
        }
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        if (this.status==null) {
          this.getProductList(val,this.nowPageSizes,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
            this.repaymentPaymentEndDate,this.orderRepaymentStartDate,this.orderRepaymentEndDate,this.repaymentOverdueDay,this.productId,this.adminId,1);
        } else {
          this.getProductList(val,this.nowPageSizes,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
            this.repaymentPaymentEndDate,this.orderRepaymentStartDate,this.orderRepaymentEndDate,this.repaymentOverdueDay,this.productId,this.adminId,null);
        }
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
       * @param data9 到期开始时间
       * @param data10 到期结束时间
       * @param data11 逾期天数
       * @param data12 所属平台
       * @param data13 催收员
       * @param data14 1-逾期，坏账，逾期已还
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9,data10,data11,data12,data13,data14){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/findAssignedOrder",
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
            orderRepaymentStartDate: data9,
            orderRepaymentEndDate: data10,
            repaymentOverdueDay: data11,
            productId: data12,
            adminId: data13,
            orderType: data14,
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
          return '逾期还款'
        }
      },
      //应还金额字段
      shouldFormatter(row){
        let repaymentCapital = row.repaymentCapital;
        let repaymentOverdueFee = row.repaymentOverdueFee;
        let repaymentPenaltyInterest = row.repaymentPenaltyInterest;
        return repaymentCapital + repaymentOverdueFee + repaymentPenaltyInterest;
      },
      //分配状态字段
      distributFormatter(row){
        return '已分配';
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
      //预计还款时间筛选
      logTimeChange3(){
        if(this.value7!=''||this.value7!=null){
          var startTime=this.value7[0];
          var endTime=this.value7[1];
          this.orderRepaymentStartDate=startTime;
          this.orderRepaymentEndDate=endTime;
        }
      },
      //取消分单
      cancelContent(){
        if (this.orderIds.length==0) {
          this.$message({
            showClose: true,
            message: '请选择至少一个订单',
            type: 'warning'
          });
        } else {
          this.$confirm('是否确认取消分单?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            center: true
          }).then(() => {
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/order/admin/borrowing/cancelAssignedOrder",
              headers:{
                'Content-Type':'application/json',
                'Authorization': localStorage.token
              },
              data:JSON.stringify(this.orderIds),
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                if (this.status==null) {
                  this.getProductList(this.pageNum,this.pageSize,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
                    this.repaymentPaymentEndDate,this.orderRepaymentStartDate,this.orderRepaymentEndDate,this.repaymentOverdueDay,this.productId,this.adminId,1);
                } else {
                  this.getProductList(this.pageNum,this.pageSize,this.status,this.mobile,this.orderStartDate,this.orderEndDate,this.repaymentPaymentStartDate,
                    this.repaymentPaymentEndDate,this.orderRepaymentStartDate,this.orderRepaymentEndDate,this.repaymentOverdueDay,this.productId,this.adminId,null);
                }
              }else if(res.data.msgCd=='ZYCASH-1009'){
                this.$message.error(res.data.msgInfo);
              }
              else {
                this.$message.error(res);
              }
            })
          });
        }
      },
      //全选
      handleSelectionChange(val) {
        this.multipleSelection = val;
        let ids = []
        this.multipleSelection.map((item)=> {
          ids.push(item.id);
        })
        this.orderIds = ids;
      },
      //选中变色
      // handleSelectionChangeGlobal(val) {
      //   this.currentRow = val;
      // },
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
      //详情
      detailProduct(row){
        let status=row.status;
        let id=row.customerId;
        let orderId=row.orderId;
        if (status == 9 || status == 11) {
          let routeData = this.$router.resolve({
            path: `/orderDetailLoanMade/${id}/${orderId}`,
          });
          window.open(routeData.href, '_blank');
        } else {
          let routeData2 = this.$router.resolve({
            path: `/orderDetailPaymentHistory/${id}/${orderId}`,
          });
          window.open(routeData2.href, '_blank');
        }
      },
      //催记
      collectionLog(row){
        let id=row.customerId;
        let routeData = this.$router.resolve({
          path: `/collectionLog/${id}`,
        });
        window.open(routeData.href, '_blank');
      },
    },
    mounted:function () {
      this.getProduct();
      this.getProductList(1,30,null,null,null,null,null,null,null,null,null,null,null,1);
    },
    data() {
      return {
        productList:[],
        collectionList:[],
        statusList: [
          {classifyId:null,classifyName:"全部状态"},
          {classifyId:11,classifyName:"逾期未还"},
          {classifyId:12,classifyName:"坏账"},
          {classifyId:13,classifyName:"逾期已还"},
        ],
        tableData:[],
        status:null,
        mobile:null,
        orderStartDate:null,
        orderEndDate:null,
        repaymentPaymentStartDate:null,
        repaymentPaymentEndDate:null,
        orderRepaymentStartDate: null,
        orderRepaymentEndDate: null,
        repaymentOverdueDay: null,
        productId:null,
        adminId: null,
        value5:'',
        value6:'',
        value7:'',
        orderIds:[],
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
        currentRow: null
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
