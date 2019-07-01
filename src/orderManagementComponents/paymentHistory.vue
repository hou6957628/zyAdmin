<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>还款记录列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-col :span="6" style="height: 55px;">
        产品：
        <el-select v-model="productId" placeholder="请选择">
          <el-option
            v-for="item in productList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
      手机号：<el-input class="searchContent" placeholder="用户手机号" v-model="mobile" clearable></el-input>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        用户姓名：<el-input class="searchContent" placeholder="用户姓名" v-model="cusName" clearable></el-input>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        还款方式：
        <el-select v-model="method" placeholder="请选择">
          <el-option
            v-for="item in methodList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="10" style="height: 55px;">
        <template>
          还款时间：
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
                          @change="logTimeChange()">
          </el-date-picker>
        </template>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        还款状态：
        <el-select v-model="status" placeholder="请选择">
          <el-option
            v-for="item in statusList"
            :key="item.classifyId"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-button style="margin-right: 250px" type="primary" id="searchBtn" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
    </div>
    <template>
      <el-table
        ref="multipleTable"
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="orderId"
          label="订单ID"
          width="260">
        </el-table-column>
        <el-table-column
          fixed
          prop="memo"
          label="还款方式"
          :formatter="methodFormatter"
          min-width="120">
        </el-table-column>
        <el-table-column
          prop="gender"
          label="还款通道"
          :formatter="genderFormatter"
          min-width="120">
        </el-table-column>
        <el-table-column
          prop="status"
          label="状态"
          :formatter="statusFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          prop="name"
          label="姓名"
          width="100">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="tranFlowId"
          label="交易流水号"
          width="255">
        </el-table-column>
        <el-table-column
          prop="borrowingCapital"
          label="合同金额"
          width="90">
        </el-table-column>
        <el-table-column
          prop="lagInterest"
          label="逾期费用"
          width="90">
        </el-table-column>
        <el-table-column
          prop="amount"
          label="应还金额"
          width="90">
        </el-table-column>
        <el-table-column
          prop="discountAmount"
          label="已减免金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="paymentAmount"
          label="实际还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="partial"
          label="是否部分还款"
          width="100">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.method == 4 || scope.row.method == 7 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.method == 4 || scope.row.method == 7 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="partialRepayment"
          label="部分还款应还金额"
          :formatter="amountFormatter"
          width="100">
        </el-table-column>
        <el-table-column
          prop="defer"
          label="是否展期"
          width="100">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.method == 2 || scope.row.method == 3 || scope.row.method == 6 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.method == 2 || scope.row.method == 3 || scope.row.method == 6 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="repaymentDefer"
          label="展期还款金额"
          :formatter="amountFormatter2"
          width="110">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="收款账户"
          width="110">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="收款平台"
          width="100">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="创建时间"
          width="170">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="更新时间"
          width="170">
        </el-table-column>
        <el-table-column
          prop="collectorName"
          label="催收员"
          width="120">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="100">
          <template slot-scope="scope">
            <el-button v-if="hasPermissionCustom('order:repayment:detail')" @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
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
      //条件查询列表
      searchContent(data){
        this.getProductList(this.pageNum,this.pageSize,this.productId,this.mobile,this.startDate,this.endDate,this.cusName,this.method,this.status);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.productId,this.mobile,this.startDate,this.endDate,this.cusName,this.method,this.status);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.productId,this.mobile,this.startDate,this.endDate,this.cusName,this.method,this.status);
      },
      /**
       * 获取还款记录列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 产品id
       * @param data4 手机号
       * @param data5 开始时间
       * @param data6 结束时间
       * @param data7 用户姓名
       * @param data8 还款方式
       * @param data9 还款状态
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/repayment/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            productId: data3,
            mobile: data4,
            startDate: data5,
            endDate: data6,
            name: data7,
            method: data8,
            status: data9,
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
      //详情
      detailProduct(row){
        let id=row.customerId;
        let orderId=row.orderId;
        this.$router.push({
          path: `/orderDetailPaymentHistory/${id}/${orderId}`,
        });
      },
      //过滤还款方式字段
      methodFormatter(row){
        let method = row.method;
        if(method == 0){
          return '线下平账'
        } else if (method == 1){
          return '单独扣款'
        } else if (method == 2){
          return '线下展期平账'
        } else if (method == 3){
          return '线上展期扣款'
        } else if (method == 4){
          return '线下部分还款'
        } else if (method == 5){
          return '主动还款'
        } else if (method == 6){
          return 'APP线上展期'
        } else if (method == 7){
          return 'APP线上部分还款'
        } else if (method == 8){
          return '批扣'
        }
      },
      //过滤性别字段
      genderFormatter(row){
        return '合利宝'
      },
      //过滤状态字段
      statusFormatter(row){
        let status = row.status;
        if(status === 0){
          return '订单生成 '
        } else if (status === 1){
          return '付款交易进行中'
        } else if (status === 2){
          return '已完成'
        } else if (status === 3){
          return '已取消'
        } else if (status === 4){
          return '失败'
        }
      },
      //过滤部分还款应还金额
      amountFormatter(row){
        let method = row.method;
        if(method == 4 || method == 7) {
          return row.amount;
        }
      },
      //过滤展期应还金额
      amountFormatter2(row){
        let method = row.method;
        if (method == 2 || method == 3|| method == 6) {
          return row.amount;
        }
      },
      //还款时间筛选
      logTimeChange(){
        if(this.value5==''||this.value5==null){
          this.startDate=null;
          this.endDate=null;
        }else {
          var startTime=this.value5[0];
          var endTime=this.value5[1];
          this.startDate=startTime;
          this.endDate=endTime;
        }
      },
    },
    mounted:function () {
      this.startDate=this.dateFormatCustom(new Date(new Date().getFullYear(), new Date().getMonth()-1, new Date().getDate(), 0, 0, 0));
      this.endDate=this.dateFormatCustom(new Date());
      this.value5=[this.startDate,this.endDate];
      this.getProduct();
      this.getProductList(1,30,null,null,this.startDate,this.endDate,null,null,null);
    },
    data() {
      return {
        productList:[],
        methodList:[
          {id:null,name:"全部"},
          {id:0,name:"线下平账"},
          {id:1,name:"单独扣款"},
          {id:2,name:"线下展期平账"},
          {id:3,name:"线上展期扣款"},
          {id:4,name:"线下部分还款"},
          {id:5,name:"主动还款"},
          {id:6,name:"APP线上展期"},
          {id:7,name:"APP线上部分还款"},
          {id:8,name:"批扣"},
        ],
        tableData:[],
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
        productId:null,
        mobile:null,
        cusName:'',
        value5:'',
        startDate:null,
        endDate:null,
        method:null,
        status:null,
        statusList: [
          {classifyId:null,classifyName:"全部状态"},
          {classifyId:0,classifyName:"订单生成"},
          {classifyId:1,classifyName:"付款交易进行中"},
          {classifyId:2,classifyName:"已完成"},
          {classifyId:3,classifyName:"已取消"},
          {classifyId:4,classifyName:"失败"},
        ],
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
