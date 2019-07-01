<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>逾期已还订单</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-col :span="6" style="height: 55px;">
        产品名称：
        <el-select v-model="productId" placeholder="请选择">
          <el-option
            v-for="item in productListData"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        状态：
        <el-select v-model="status" placeholder="请选择">
          <el-option
            v-for="item in electData1"
            :key="item.id"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
      <template>手机号码：
      <el-input class="searchContent"
        placeholder="用户手机号"
        v-model="mobile"
        clearable>
      </el-input>
      </template>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        <template>逾期天数：
          <el-input class="searchContent"
                    placeholder="逾期天数"
                    v-model="repaymentOverdueDay"
                    clearable>
          </el-input>
        </template>
      </el-col>
      <el-col :span="12" style="height: 55px;">
        <template>
          应还款时间：
          <el-date-picker style="margin-left: 0px"
                          v-model="orderRepaymentDate"
                          type="datetimerange"
                          align="right"
                          unlink-panels
                          range-separator="至"
                          start-placeholder="开始日期"
                          end-placeholder="结束日期"
                          :picker-options="pickerOptions2"
                          format="yyyy-MM-dd HH:mm:ss"
                          value-format="yyyy-MM-dd HH:mm:ss"
                          @change="logTimeChange()">
          </el-date-picker>
        </template>
      </el-col>
      <el-col :span="12" style="height: 55px;">
        <template>
          申请时间：
          <el-date-picker style="margin-left: 0"
                          v-model="orderDate"
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
                          v-model="repaymentPaymentDate"
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
        用户状态：
        <el-select v-model="reBorrow" placeholder="请选择">
          <el-option
            v-for="item in electData2"
            :key="item.classifyId"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-button id="searchBtn" type="primary" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
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
          prop="reBorrow"
          label="用户标识"
          :formatter="reBorrowFormatter"
          width="100">
        </el-table-column>
        <el-table-column
          prop="borrowingPeriod"
          label="借款期限"
          width="110">
        </el-table-column>
        <el-table-column
          prop="borrowingCapital"
          label="借款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="status"
          label="订单状态"
          width="110"
          :formatter="statusFormatter">
        </el-table-column>
        <el-table-column
          prop="borrowingInterest"
          label="综合费用"
          width="110">
        </el-table-column>
        <el-table-column
          prop="borrowingPaymentAmount"
          label="到账金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="repaymentOverdueDay"
          label="逾期天数"
          width="100">
        </el-table-column>
        <el-table-column
          prop="repaymentOverdueFee"
          label="罚息"
          width="110">
        </el-table-column>
        <el-table-column
          prop="repaymentPenaltyInterest"
          label="逾期费用"
          width="110">
        </el-table-column>
        <el-table-column
          prop="loanNumber"
          label="应还金额"
          :formatter="shouldFormatter"
          width="110">
        </el-table-column>
        <el-table-column
          prop="repaymentPaymentAmount"
          label="实际还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="repaymentDiscountAmount"
          label="已减免金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="defer"
          label="是否展期"
          width="110">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.defer == 1 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.defer == 1 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="repaymentDefer"
          label="展期实际还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="partial"
          label="是否是部分还款"
          width="110">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.partial == 1 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.partial == 1 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="partialRepayment"
          label="部分还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="剩余还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="应用"
          width="110">
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
          label="应还款日期"
          width="200">
        </el-table-column>
        <el-table-column
          prop="parentChannelName"
          label="主渠道"
          width="150">
        </el-table-column>
        <el-table-column
          prop="childrenChannelName"
          label="子渠道"
          width="150">
        </el-table-column>
        <el-table-column
          prop="collectorName"
          label="催收员"
          width="150">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="60">
          <template slot-scope="scope">
            <el-button @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
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
      //查询金融产品
      searchContent(){
        this.getOrderList(this.productId,this.status,this.mobile,this.repaymentOverdueDay,this.orderRepaymentDate,
          this.orderDate,this.repaymentPaymentDate,this.reBorrow,this.pageNum,this.pageSize,null);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getOrderList(this.productId,this.status,this.mobile,this.repaymentOverdueDay,this.orderRepaymentDate,
          this.orderDate,this.repaymentPaymentDate,this.reBorrow,this.pageNum,val,null);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getOrderList(this.productId,this.status,this.mobile,this.repaymentOverdueDay,this.orderRepaymentDate,
          this.orderDate,this.repaymentPaymentDate,this.reBorrow,val,this.nowPageSizes,null);
      },
      //查询产品列表
      getProductList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getProductList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productListData=res.data.body;
            this.productListData.unshift({productId: null, productName: '全部产品'});
          } else if (res.data.msgCd=='ZYCASH-1005') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else if (res.data.msgCd=='SYS00001') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      /**
       * 订单管理查询
       * @param data1 产品id
       * @param data2 订单状态(
       * 0  待机器审核
       * 1  机器审核中
       * 2  审核拒绝
       * 3  人工审核
       * 4  待放款
       * 5  放款中
       * 6  放款失败
       * 7  取消
       * 8  放款成功，待还款
       * 9  还款确认中
       * 10 逾期
       * 11 坏账
       * 12 正常还款
       * 13 逾期还款)
       * @param data3 用户手机号
       * @param data4 逾期天数
       * @param data5 预计还款时间开始时间、结束时间
       * @param data6 申请时间开始时间、结束时间
       * @param data7 放款时间开始时间、结束时间
       * @param data8 新老户 0-false-新户 1-true-老户
       * @param data9 分页数量
       * @param data10 分页页数
       * @param data11 1-逾期，坏账，逾期已还
       */
      getOrderList(data1,data2,data3,data4,data5,data6,data7,data8,data9,data10,data11){
        let orderRepaymentStartDate = null;
        let orderRepaymentEndDate = null;
        let orderStartDate = null;
        let orderEndDate = null;
        let repaymentPaymentStartDate = null;
        let repaymentPaymentEndDate = null;
        if (data5!='' & data5!=null) {
          orderRepaymentStartDate=data5[0];
          orderRepaymentEndDate=data5[1];
        }
        if (data6!='' & data6!=null) {
          orderStartDate=data6[0];
          orderEndDate=data6[1];
        }
        if (data7!='' & data7!=null) {
          repaymentPaymentStartDate=data7[0];
          repaymentPaymentEndDate=data7[1];
        }
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getOrderList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            // this.getOrderList(this.productId,this.status,this.mobile,this.repaymentOverdueDay,this.orderRepaymentDate,
            // this.orderDate,this.repaymentPaymentDate,this.reBorrow,this.pageNum,this.pageSize
            productId: data1,
            status: data2,
            mobile: data3,
            repaymentOverdueDay: data4,
            orderRepaymentStartDate: orderRepaymentStartDate,
            orderRepaymentEndDate: orderRepaymentEndDate,
            orderStartDate: orderStartDate,
            orderEndDate: orderEndDate,
            repaymentPaymentStartDate: repaymentPaymentStartDate,
            repaymentPaymentEndDate: repaymentPaymentEndDate,
            reBorrow: data8,
            pageNum: data9,
            pageSize: data10,
            orderType: data11,
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
        if(status === 11){
          return '逾期未还 '
        } else if (status === 12){
          return '坏账'
        } else if (status === 13){
          return '逾期还款'
        }
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
      //应还金额字段
      shouldFormatter(row){
        let repaymentCapital = row.repaymentCapital;
        let repaymentOverdueFee = row.repaymentOverdueFee;
        let repaymentPenaltyInterest = row.repaymentPenaltyInterest;
        return repaymentCapital + repaymentOverdueFee + repaymentPenaltyInterest;
      },
      //时间筛选
      logTimeChange(){
        if(this.value7==''||this.value7==null){
          this.getProductList(this.pageNum,this.nowPageSizes,this.value8,null,null);
        }else {
          var startTime=this.value7[0];
          var endTime=this.value7[1];
          this.startTime=startTime;
          this.endTime=endTime;
          console.log("开始时间 : "+this.startTime+"结束时间 : "+this.endTime);
          // this.getProductList(this.pageNum,this.nowPageSizes,this.value8,this.startTime,this.endTime);
        }
      },
      //时间筛选1
      logTimeChange1(){
        if(this.value6==''||this.value6==null){
          this.getProductList(this.pageNum,this.nowPageSizes,this.value8,null,null);
        }else {
          var startTime=this.value6[0];
          var endTime=this.value6[1];
          this.startTime=startTime;
          this.endTime=endTime;
          console.log("开始时间 : "+this.startTime+"结束时间 : "+this.endTime);
          // this.getProductList(this.pageNum,this.nowPageSizes,this.value8,this.startTime,this.endTime);
        }
      },
      //时间筛选2
      logTimeChange2(){
        if(this.value5==''||this.value5==null){
          this.getProductList(this.pageNum,this.nowPageSizes,this.value8,null,null);
        }else {
          var startTime=this.value5[0];
          var endTime=this.value5[1];
          this.startTime=startTime;
          this.endTime=endTime;
          console.log("开始时间 : "+this.startTime+"结束时间 : "+this.endTime);
          // this.getProductList(this.pageNum,this.nowPageSizes,this.value8,this.startTime,this.endTime);
        }
      },
      //详情
      detailProduct(row){
        let id=row.customerId;
        let orderId=row.orderId;
        let routeData = this.$router.resolve({
          path: `/orderDetailAfterLoanRepayed/${id}/${orderId}`
        });
        window.open(routeData.href, '_blank');
        // let id=row.customerId;
        // let orderId=row.orderId;
        // this.$router.push({
        //   path: `/orderDetailAfterLoanRepayed/${id}/${orderId}`,
        // });
      }
    },
    mounted:function () {
      this.getOrderList(null,13,null,null,null,null,null,null,1,30,null);
      this.getProductList();
    },
    data() {
      return {
        electData1: [
          {classifyId:13,classifyName:"逾期还款"},
        ],
        electData2: [
          {classifyId:'',classifyName:"全部状态"},
          {classifyId:0,classifyName:"新户"},
          {classifyId:1,classifyName:"老户"},
        ],
        tableData:[],
        productListData:[],
        productId:null,
        status:13,
        mobile:'',
        repaymentOverdueDay:'',
        orderRepaymentDate:'',
        orderDate:'',
        repaymentPaymentDate:'',
        reBorrow:'',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
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
    margin-left:5px;
    width: 200px;
    margin-right: 30px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
