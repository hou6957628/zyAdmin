<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/afterLoanManagement' }">逾期订单管理</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-col :span="6" style="height: 55px;">
        产品名称：
        <el-select v-model="electValue" placeholder="请选择" @change="selectChange">
          <el-option
            v-for="item in electData"
            :key="item.id"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        状态：
        <el-select v-model="electValue1" placeholder="请选择" @change="selectChange1">
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
      <el-input @click="searchProduct" class="searchContent"
        placeholder="用户手机号"
        v-model="finProduct"
        clearable>
      </el-input>
      </template>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        <template>逾期天数：
          <el-input @click="searchProduct" class="searchContent"
                    placeholder="逾期天数"
                    v-model="overdueNum"
                    clearable>
          </el-input>
        </template>
      </el-col>
      <el-col :span="12" style="height: 55px;">
        <template>
          应还款时间：
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
      <el-col :span="12" style="height: 55px;">
        <template>
          申请时间：
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
                          @change="logTimeChange1()">
          </el-date-picker>
        </template>
      </el-col>
      <el-col :span="12" style="height: 55px;">
        <template>
          放款时间：
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
      <el-col :span="6" style="height: 55px;">
        用户状态：
        <el-select v-model="electValue2" placeholder="请选择" @change="selectChange2">
          <el-option
            v-for="item in electData2"
            :key="item.id"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-button id="searchBtn" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="productCode"
          label="订单ID"
          width="120">
        </el-table-column>
        <el-table-column
          fixed
          prop="realName"
          label="姓名"
          min-width="80">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="productId"
          label="借款期限"
          width="120">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="借款金额"
          width="120">
        </el-table-column>
        <el-table-column
          prop="channelName"
          label="订单状态"
          width="120">
        </el-table-column>
        <el-table-column
          prop="subChannelName"
          label="综合费用"
          width="120">
        </el-table-column>
        <el-table-column
          prop="interestMethods"
          label="到账金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="逾期天数"
          width="150">
        </el-table-column>
        <el-table-column
          prop="isActiveApp"
          label="罚息"
          width="80">
        </el-table-column>
        <el-table-column
          prop="applyNumber"
          label="逾期费用"
          width="110">
        </el-table-column>
        <el-table-column
          prop="loanNumber"
          label="应还金额"
          width="80">
        </el-table-column>
        <el-table-column
          prop="isBlackList"
          label="实际还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="overdueNumber"
          label="已减免金额"
          width="80">
        </el-table-column>
        <el-table-column
          prop="reBorrow"
          label="是否展期"
          width="80">
        </el-table-column>
        <el-table-column
          prop="hasIncreaseAmount"
          label="展期实际还款金额"
          width="80">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="是否是部分还款"
          width="80">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="部分还款金额"
          width="80">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="剩余还款金额"
          width="80">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="应用"
          width="80">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="申请时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="放款时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="应还款日期"
          width="200">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="主渠道"
          width="200">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="子渠道"
          width="200">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="是否是黑名单"
          width="200">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="逾期次数"
          width="200">
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="催收员"
          width="200">
        </el-table-column>
        <el-table-column
          label="操作"
          width="60">
          <template slot-scope="scope">
            <el-button @click="detailProduct(scope.row)" type="text" size="small">详情</el-button>
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
      searchContent(data){
        if(data==""){
          this.getProductList(1,20,null,null);
          // this.$message.error('搜索内容不可以为空');
        }else {
          this.getProductList(1,20,data,this.finProduct);
          console.log(data);
        }
      },
      //每页显示多少条
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.getProductList(this.pageNum,val,this.finProduct,this.finProduct);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        console.log(this.nowPageSizes);
        this.getProductList(val,this.nowPageSizes,this.finProduct,this.finProduct);
      },
      //创建金融产品
      detailProduct(){
        this.$router.push({
          path: `/editFinanceProduct`,
        });
      },
      /**
       * 订单管理查询
       * @param data1 订单状态(
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
       * @param data2 用户手机号
       * @param data3 群组Id
       * @param data4 下单开始时间
       * @param data5 下单结束时间
       * @param data6 放款开始时间
       * @param data7 放款结束时间
       * @param data8 逾期天数
       * @param data9 预计还款时间开始时间
       * @param data10 预计还款时间结束时间
       * @param data11 产品id
       * @param data12 新老户 0-false-新户 1-true-老户
       * @param data13 委外状态 0：未委外 1：委外
       * @param data14 分页数量
       * @param data15 分页页数
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9,data10,data11,data12,data13,data14,data15){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getOrderList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            status: 10,
            mobile: data2,
            groupId: data3,
            orderStartDate: data4,
            orderEndDate: data5,
            repaymentPaymentStartDate: data6,
            repaymentPaymentEndDate: data7,
            repaymentOverdueDay: data8,
            orderRepaymentStartDate: data9,
            orderRepaymentEndDate: data10,
            productId: data11,
            reBorrow: data12,
            outSourceStatus: data13,
            pageSize: data14,
            pageNum: data15,
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
      //查询产品接口
      searchProduct(){
        this.getProductList(1,20,this.finProduct,this.finProduct);
      },
      //过滤类型字段
      typeFormatter(row){
        let status = row.type;
        if(status === 0){
          return '信贷产品'
        } else {
          return '分期产品'
        }
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
      //下拉选择
      selectChange(row){
        console.log(this.electValue);
      },
      selectChange1(row){
        console.log(this.electValue1);
      },
      selectChange2(row){
        console.log(this.electValue2);
      },
    },
    mounted:function () {
      // this.finProduct=this.$route.params.name;
       this.getProductList();
    },
    data() {
      return {
        electData: [
          {classifyId:0,classifyName:"全部产品"},
          {classifyId:1,classifyName:"借点儿"},
          {classifyId:2,classifyName:"夏花花"},
          {classifyId:3,classifyName:"取消救济"},
        ],
        electData1: [
          {classifyId:0,classifyName:"全部状态"},
          {classifyId:1,classifyName:"逾期"},
          {classifyId:2,classifyName:"坏账"},
          {classifyId:3,classifyName:"逾期已还"},
          {classifyId:4,classifyName:"坏账已还"},
        ],
        electData2: [
          {classifyId:0,classifyName:"全部状态"},
          {classifyId:1,classifyName:"新户"},
          {classifyId:2,classifyName:"老户"},
        ],
        tableData:[],
        electValue:0,
        electValue1:0,
        electValue2:0,
        finProduct: '',
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
        value5:'',
        value6:'',
        value7:'',
        overdueNum:'',
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
