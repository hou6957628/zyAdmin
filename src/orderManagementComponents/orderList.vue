<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>待放款列表</el-breadcrumb-item>
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
        新老户：
        <el-select v-model="reBorrow" placeholder="请选择">
          <el-option
            v-for="item in reBorrowList"
            :key="item.id"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        主渠道：<el-input class="searchContent" placeholder="主渠道" v-model="parentChannelName" clearable></el-input>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        子渠道：<el-input class="searchContent" placeholder="子渠道" v-model="childrenChannelName" clearable></el-input>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        性别：<el-select v-model="sex" placeholder="请选择">
          <el-option
            v-for="item in sexList"
            :key="item.id"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
      手机号：<el-input class="searchContent" placeholder="用户手机号" v-model="mobile" clearable></el-input>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        用户姓名：<el-input class="searchContent" placeholder="用户姓名" v-model="cusName" clearable></el-input>
      </el-col>
      <el-col :span="10" style="height: 55px;">
        <template>
          申请时间：
          <el-date-picker style="margin-left: 0px"
                          v-model="value4"
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
      <el-col :span="10" style="height: 55px;">
        <template>
          放款时间：
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
                          @change="logTimeChange2()">
          </el-date-picker>
        </template>
      </el-col>
      <el-col :span="10" style="height: 55px;">
        <template>
          到期时间：
          <el-date-picker style="margin-left: 0px"
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
                          @change="logTimeChange3()">
          </el-date-picker>
        </template>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        展期：
        <el-select v-model="defer" placeholder="请选择">
          <el-option
            v-for="item in deferList"
            :key="item.id"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-button type="primary" style="margin-right: 130px" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
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
          prop="name"
          label="姓名"
          min-width="80">
        </el-table-column>
        <el-table-column
          prop="gender"
          label="性别"
          :formatter="genderFormatter"
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
          width="150">
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
          prop="repaymentOverdueFee"
          label="逾期费用"
          width="100">
        </el-table-column>
        <el-table-column
          prop="repaymentPenaltyInterest"
          label="罚息"
          width="100">
        </el-table-column>
        <el-table-column
          prop="repaymentCapital"
          label="应还金额"
          :formatter="shouldFormatter"
          width="100">
        </el-table-column>
        <el-table-column
          prop="repaymentPaymentAmount"
          label="实际还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="repaymentDiscountAmount"
          label="已减免金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="defer"
          label="是否展期"
          width="100">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.defer == 1 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.defer == 1 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="repaymentDefer"
          label="展期应还金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="repaymentDeferPayment"
          label="实际展期还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="partial"
          label="是否部分还款"
          width="100">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.partial == 1 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.partial == 1 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="partialRepayment"
          label="部分还款应还金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="partialDiscountAmount"
          label="部分还款累计减免金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="partialRepaymentPayment"
          label="部分还款累计已还金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="partialUnpaidAmount"
          label="部分还款剩余还款金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="应用"
          width="100">
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
          prop="repaymentPaymentDate"
          label="实际还款时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="parentChannelName"
          label="主渠道"
          width="120">
        </el-table-column>
        <el-table-column
          prop="childrenChannelName"
          label="子渠道"
          width="120">
        </el-table-column>
        <el-table-column
          prop="reBorrow"
          label="用户标识"
          :formatter="reBorrowFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          prop="collectorName"
          label="催收员"
          width="120">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="450">
          <template slot-scope="scope">
            <div v-if="scope.row.status==8 || scope.row.status==9 || scope.row.status==11 || scope.row.status==12">
              <div v-if="scope.row.partial">
                <el-button @click="partialTip(scope.row)" type="text" size="medium">部分还款</el-button>
                <el-button @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
              </div>
              <div v-if="!scope.row.partial">
                <el-button v-if="hasPermissionCustom('order:orderAll:offlineRepayment')" @click="offlineRepaymentTip(scope.row)" type="text" size="medium">线下还款</el-button>
                <el-button v-if="hasPermissionCustom('order:orderAll:onlineRelief')" @click="onlineReliefTip(scope.row)" type="text" size="medium">线上减免</el-button>
                <el-button v-if="hasPermissionCustom('order:orderAll:separateDeduction')" @click="separateDeductionTip(scope.row)" type="text" size="medium">单独扣款</el-button>
                <el-button v-if="scope.row.enableDefer && hasPermissionCustom('order:orderAll:deffer')" @click="lineDefferTip(scope.row)" type="text" size="medium">展期还款</el-button>
                <el-button v-if="!scope.row.enableDefer && hasPermissionCustom('order:orderAll:deffer')" @click="lineDefferTip(scope.row)" type="text" size="medium">特例展期</el-button>
                <el-button v-if="hasPermissionCustom('order:orderAll:partial')" @click="partialTip(scope.row)" type="text" size="medium">部分还款</el-button>
                <el-button v-if="hasPermissionCustom('order:orderAll:detail')" @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
              </div>
            </div>
            <div v-else>
              <el-button v-if="hasPermissionCustom('order:orderAll:detail')" @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
            </div>
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
    <!--线下还款结构-->
    <el-dialog
      title="线下还款"
      :visible.sync="centerDialogVisible1"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="130px" class="demo-ruleForm">
        <el-form-item label="当前应还总金额:">
          <el-input v-model="ruleForm.shouldRepayment" disabled></el-input>
        </el-form-item>
        <el-form-item label="滞纳金+罚息:">
          <el-input v-model="ruleForm.interest" disabled></el-input>
        </el-form-item>
        <el-form-item label="减免金额:" prop="repaymentDiscountAmount">
          <el-input v-model="ruleForm.repaymentDiscountAmount"></el-input>
        </el-form-item>
        <el-form-item label="实际还款金额:">
          <el-input v-model="ruleForm.repaymentPaymentAmount" disabled></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="offlineRepayment('ruleForm')">确认平账<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible1 = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!--线上减免结构-->
    <el-dialog
      title="线上减免"
      :visible.sync="centerDialogVisible2"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="130px" class="demo-ruleForm">
        <el-form-item label="当前应还总金额:" prop="shouldRepayment">
          <el-input v-model="ruleForm.shouldRepayment" disabled></el-input>
        </el-form-item>
        <el-form-item label="滞纳金+罚息:">
          <el-input v-model="ruleForm.interest" disabled></el-input>
        </el-form-item>
        <el-form-item label="减免金额:" prop="repaymentDiscountAmount">
          <el-input v-model="ruleForm.repaymentDiscountAmount"></el-input>
        </el-form-item>
        <el-form-item label="实际还款金额:">
          <el-input v-model="ruleForm.repaymentPaymentAmount" disabled></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onlineRelief('ruleForm')">确认减免<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible2 = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!--单独扣款结构-->
    <el-dialog
      title="单独扣款"
      :visible.sync="centerDialogVisible3"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="130px" class="demo-ruleForm">
        <el-form-item label="当前应还总金额:" prop="shouldRepayment">
          <el-input v-model="ruleForm.shouldRepayment" disabled></el-input>
        </el-form-item>
        <el-form-item label="滞纳金+罚息:">
          <el-input v-model="ruleForm.interest" disabled></el-input>
        </el-form-item>
        <el-form-item label="减免金额:" prop="repaymentDiscountAmount">
          <el-input v-model="ruleForm.repaymentDiscountAmount"></el-input>
        </el-form-item>
        <el-form-item label="实际还款金额:">
          <el-input v-model="ruleForm.repaymentPaymentAmount" disabled></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="separateDeduction('ruleForm')">扣款<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible3 = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!--展期还款结构-->
    <el-dialog
      title="展期还款"
      :visible.sync="centerDialogVisible4"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="140px" class="demo-ruleForm">
        <el-form-item label="当前应还总金额:" prop="shouldRepayment">
          <el-input v-model="ruleForm.shouldRepayment" disabled></el-input>
        </el-form-item>
        <el-form-item label="展期应还金额:">
          <el-input v-model="ruleForm.repaymentDefer" disabled></el-input>
        </el-form-item>
        <el-form-item label="滞纳金+罚息:">
          <el-input v-model="ruleForm.interest" disabled></el-input>
        </el-form-item>
        <el-form-item label="展期减免金额:" prop="repaymentDeferDiscountAmount">
          <el-input v-model="ruleForm.repaymentDeferDiscountAmount"></el-input>
        </el-form-item>
        <el-form-item label="展期实际还款金额:" prop="repaymentDeferPayment">
          <el-input v-model="ruleForm.repaymentDeferPayment" disabled></el-input>
        </el-form-item>
        <el-button style="margin-left: 80px" type="primary" @click="lineDefferDisAmount('ruleForm')">线上展期减免</el-button>
        <el-button type="primary" @click="offLineDEFER('ruleForm')">线下展期</el-button>
        <el-button type="primary" @click="lineDeffer('ruleForm')">展期单独扣款</el-button>
        <el-button type="info"  @click="centerDialogVisible4 = false">取消</el-button>
      </el-form>
    </el-dialog>
    <!--部分还款结构-->
    <el-dialog
      title="部分还款"
      :visible.sync="centerDialogVisible5"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="140px" class="demo-ruleForm">
        <el-form-item label="当前应还总金额:">
          <el-input v-model="ruleForm.shouldRepayment" disabled></el-input>
        </el-form-item>
        <el-form-item label="逾期+罚息:">
          <el-input v-model="ruleForm.interest" disabled></el-input>
        </el-form-item>
        <el-form-item label="剩余应还金额:">
          <el-input v-model="ruleForm.partialUnpaidAmount" disabled></el-input>
        </el-form-item>
        <el-form-item label="本次部分还款金额:">
          <el-input v-model="ruleForm.partialRepayment" disabled></el-input>
        </el-form-item>
        <el-form-item label="本次部分实还金额:" prop="partialRepaymentPayment">
          <el-input v-model="ruleForm.partialRepaymentPayment"></el-input>
        </el-form-item>
        <el-form-item label="本次部分减免金额:" prop="partialDiscountAmount">
          <el-input v-model="ruleForm.partialDiscountAmount"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="partial('ruleForm')">线下部分还款<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible5 = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
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
      searchContent(){
        this.getProductList(this.pageNum,this.pageSize,this.productId,this.reBorrow,this.parentChannelName,this.childrenChannelName,this.sex,this.mobile,
          this.startDate,this.endDate,this.startDateLoan,this.endDateLoan,this.cusName,this.status,this.rePaymentStartDate,this.rePaymentEndDate,this.defer);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.productId,this.reBorrow,this.parentChannelName,this.childrenChannelName,this.sex,this.mobile,
          this.startDate,this.endDate,this.startDateLoan,this.endDateLoan,this.cusName,this.status,this.rePaymentStartDate,this.rePaymentEndDate,this.defer);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.productId,this.reBorrow,this.parentChannelName,this.childrenChannelName,this.sex,this.mobile,
          this.startDate,this.endDate,this.startDateLoan,this.endDateLoan,this.cusName,this.status,this.rePaymentStartDate,this.rePaymentEndDate,this.defer);
      },
      /**
       * 获取待放款订单列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 产品id
       * @param data4 新老户
       * @param data5 主渠道名称
       * @param data6 子渠道名称
       * @param data7 性别
       * @param data8 手机号
       * @param data9 申请开始时间
       * @param data10 申请结束时间
       * @param data11 放款开始时间
       * @param data12 放款结束时间
       * @param data13 用户姓名
       * @param data14 状态
       * @param data15 到期开始时间
       * @param data16 到期结束时间
       * @param data17 是否展期
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9,data10,data11,data12,data13,data14,data15,data16,data17){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            productId: data3,
            reBorrow: data4,
            parentChannelName: data5,
            childrenChannelName: data6,
            gender: data7,
            mobile: data8,
            name: data13,
            startDate: data9,
            endDate: data10,
            startDateLoan: data11,
            endDateLoan: data12,
            status: data14,
            rePaymentStartDate:data15,
            rePaymentEndDate:data16,
            defer:data17,
            sortColumn: 'create_date',
            direction: 'desc',
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
      //线下还款弹窗
      offlineRepaymentTip(row){
        this.centerDialogVisible1=true;
        this.orderId=row.orderId;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=row.repaymentCapital + row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //获取减免金额
        this.ruleForm.repaymentDiscountAmount=row.repaymentDiscountAmount;
        //计算实际还款金额
        this.ruleForm.repaymentPaymentAmount=this.ruleForm.shouldRepayment - this.ruleForm.repaymentDiscountAmount;
      },
      //线下还款
      offlineRepayment(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm('是否确定平账?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/order/admin/borrowing/offlineRepayment",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                params:{
                  orderId:this.orderId,
                  discountAmount:this.ruleForm.repaymentDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentPaymentAmount,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible1=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.getProductList(1,30,null,null,null,null,null,null,null,null,this.startDateLoan,this.endDateLoan,null);
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //线上减免弹窗
      onlineReliefTip(row){
        this.centerDialogVisible2=true;
        this.orderId=row.orderId;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=row.repaymentCapital + row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //获取减免金额
        this.ruleForm.repaymentDiscountAmount=row.repaymentDiscountAmount;
        //计算实际还款金额
        this.ruleForm.repaymentPaymentAmount=this.ruleForm.shouldRepayment - this.ruleForm.repaymentDiscountAmount;
      },
      //线上减免
      onlineRelief(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm('是否确定减免?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/order/admin/borrowing/onlineRelief",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                params:{
                  orderId:this.orderId,
                  discountAmount:this.ruleForm.repaymentDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentPaymentAmount,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible2=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.getProductList(1,30,null,null,null,null,null,null,null,null,this.startDateLoan,this.endDateLoan,null);
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //单独扣款弹窗
      separateDeductionTip(row){
        this.centerDialogVisible3=true;
        this.orderId=row.orderId;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=row.repaymentCapital + row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //获取减免金额
        this.ruleForm.repaymentDiscountAmount=row.repaymentDiscountAmount;
        //计算实际还款金额
        this.ruleForm.repaymentPaymentAmount=this.ruleForm.shouldRepayment - this.ruleForm.repaymentDiscountAmount;
      },
      //单独扣款
      separateDeduction(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm('是否确定单独扣款?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/order/admin/borrowing/separateDeduction",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                params:{
                  orderId:this.orderId,
                  discountAmount:this.ruleForm.repaymentDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentPaymentAmount,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible3=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.getProductList(1,30,null,null,null,null,null,null,null,null,this.startDateLoan,this.endDateLoan,null);
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //展期还款弹窗
      lineDefferTip(row){
        this.centerDialogVisible4=true;
        this.orderId=row.orderId;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //计算展期应还金额
        this.ruleForm.repaymentDefer=row.borrowingInterest + row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=row.repaymentCapital + row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //获取减免金额
        this.ruleForm.repaymentDeferDiscountAmount=row.repaymentDeferDiscountAmount;
        //计算实际还款金额
        this.ruleForm.repaymentDeferPayment=this.ruleForm.repaymentDefer - this.ruleForm.repaymentDeferDiscountAmount;
      },
      //线上展期减免
      lineDefferDisAmount(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm('是否确定线上展期减免?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/order/admin/borrowing/lineDefferDisAmount",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                params:{
                  orderId:this.orderId,
                  discountAmount:this.ruleForm.repaymentDeferDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentDeferPayment,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible4=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.getProductList(1,30,null,null,null,null,null,null,null,null,this.startDateLoan,this.endDateLoan,null);
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //线下展期
      offLineDEFER(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm('是否确定线下展期?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/order/admin/borrowing/offLineDEFER",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                params:{
                  orderId:this.orderId,
                  discountAmount:this.ruleForm.repaymentDeferDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentDeferPayment,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible4=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.getProductList(1,30,null,null,null,null,null,null,null,null,this.startDateLoan,this.endDateLoan,null);
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //展期单独扣款
      lineDeffer(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm('是否确定展期单独扣款?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/order/admin/borrowing/lineDeffer",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                params:{
                  orderId:this.orderId,
                  discountAmount:this.ruleForm.repaymentDeferDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentDeferPayment,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible4=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.getProductList(1,30,null,null,null,null,null,null,null,null,this.startDateLoan,this.endDateLoan,null);
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //部分还款弹窗
      partialTip(row){
        this.centerDialogVisible5=true;
        this.orderId=row.orderId;
        this.partialFlag=row.partial;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=row.repaymentCapital + row.repaymentOverdueFee + row.repaymentPenaltyInterest;
        //剩余应还金额
        if (row.partial) {
          this.ruleForm.partialUnpaidAmount=row.partialUnpaidAmount;
        } else {
          this.ruleForm.partialUnpaidAmount=0;
        }
        //本次部分还款金额
        this.ruleForm.partialRepayment=0;
        //本次部分实还金额
        this.ruleForm.partialRepaymentPayment=0;
        //本次部分减免金额
        this.ruleForm.partialDiscountAmount=0;
      },
      //部分还款
      partial(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm('是否确定线下部分还款?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/order/admin/borrowing/partial",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                params:{
                  orderId:this.orderId,
                  discountAmount:this.ruleForm.partialDiscountAmount,
                  paymentAmount: this.ruleForm.partialRepaymentPayment,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible5=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.getProductList(1,30,null,null,null,null,null,null,null,null,this.startDateLoan,this.endDateLoan,null);
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //详情
      detailProduct(row){
        let status=row.status;
        let id=row.customerId;
        let orderId=row.orderId;
        if (status == 0 || status == 1 || status == 3) {
          this.$router.push({
            path: `/orderDetail/${id}/${orderId}`,
          });
        } else if (status == 4 || status == 5) {
          this.$router.push({
            path: `/orderDetailPendingLoan/${id}/${orderId}`,
          });
        } else if (status == 8 || status == 9 || status == 11 || status == 12) {
          this.$router.push({
            path: `/orderDetailLoanMade/${id}/${orderId}`,
          });
        }else {
          this.$router.push({
            path: `/orderDetailPaymentHistory/${id}/${orderId}`,
          });
        }
      },
      //过滤性别字段
      genderFormatter(row){
        let gender = row.gender;
        if(gender == false){
          return '男'
        } else if (gender == true){
          return '女'
        } else {
          return '未知'
        }
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
      //过滤用户标识字段
      reBorrowFormatter(row){
        let reBorrow = row.reBorrow;
        if(reBorrow == false){
          return '新户'
        } else if (reBorrow == true){
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
      //申请时间筛选
      logTimeChange(){
        if(this.value4==''||this.value4==null){
          this.startDate=null;
          this.endDate=null;
        }else {
          var startTime=this.value4[0];
          var endTime=this.value4[1];
          this.startDate=startTime;
          this.endDate=endTime;
        }
      },
      //放款时间筛选
      logTimeChange2(){
        if(this.value5==''||this.value5==null){
          this.startDateLoan=null;
          this.endDateLoan=null;
        }else {
          var startTime=this.value5[0];
          var endTime=this.value5[1];
          this.startDateLoan=startTime;
          this.endDateLoan=endTime;
        }
      },
      //到期时间筛选
      logTimeChange3(){
        if(this.value6==''||this.value6==null){
          this.rePaymentStartDate=null;
          this.rePaymentEndDate=null;
        }else {
          var startTime=this.value6[0];
          var endTime=this.value6[1];
          this.rePaymentStartDate=startTime;
          this.rePaymentEndDate=endTime;
        }
      },
    },
    mounted:function () {
      this.getProduct();
      this.getProductList(1,30,null,null,null,null,null,null,null,null,null,null,null,null,null,null);
    },
    data() {
      //正常减免金额
      var validatorNumber = (rule, value, callback) => {
        if (value==null) {
          callback(new Error('输入不能为空'));
        } else {
          if((/^[+]{0,1}(\d+)$|^[+]{0,1}(\d+\.\d+)$/).test(value) == false){
            callback(new Error("请填写正整数"));
          }else{
            this.ruleForm.repaymentPaymentAmount=this.ruleForm.shouldRepayment - value;
            callback();
          }
        }
      };
      //本次部分实还金额
      var validatorNumber2 = (rule, value, callback) => {
        if (value==null) {
          callback(new Error('输入不能为空'));
        } else {
          if((/^[+]{0,1}(\d+)$|^[+]{0,1}(\d+\.\d+)$/).test(value) == false){
            callback(new Error("请填写正整数"));
          }else{
            this.ruleForm.partialRepayment=Number(this.ruleForm.partialDiscountAmount) + Number(value);
            callback();
          }
        }
      };
      //部分减免金额
      var validatorNumber3 = (rule, value, callback) => {
        if (value==null) {
          callback(new Error('输入不能为空'));
        } else {
          if((/^[+]{0,1}(\d+)$|^[+]{0,1}(\d+\.\d+)$/).test(value) == false){
            callback(new Error("请填写正整数"));
          }else{
            this.ruleForm.partialRepayment=Number(this.ruleForm.partialRepaymentPayment) + Number(value);
            callback();
          }
        }
      };
      //展期减免金额
      var validatorNumber4 = (rule, value, callback) => {
        if (value==null) {
          callback(new Error('输入不能为空'));
        } else {
          if((/^[+]{0,1}(\d+)$|^[+]{0,1}(\d+\.\d+)$/).test(value) == false){
            callback(new Error("请填写正整数"));
          }else{
            this.ruleForm.repaymentDeferPayment=this.ruleForm.repaymentDefer - value;
            callback();
          }
        }
      };
      return {
        partialFlag:null,//这笔订单是否已经是部分还款
        productList:[],
        electData1: [
          {classifyId:null,classifyName:"全部状态"},
          {classifyId:0,classifyName:"待机审"},
          {classifyId:1,classifyName:"机器审核中"},
          {classifyId:2,classifyName:"审核拒绝"},
          {classifyId:3,classifyName:"人工审核"},
          {classifyId:4,classifyName:"待放款"},
          {classifyId:5,classifyName:"放款中"},
          {classifyId:6,classifyName:"放款失败"},
          {classifyId:7,classifyName:"取消"},
          {classifyId:8,classifyName:"放款成功，待还款"},
          {classifyId:9,classifyName:"还款确认中"},
          {classifyId:10,classifyName:"正常还款 "},
          {classifyId:11,classifyName:"逾期未还"},
          {classifyId:12,classifyName:"坏账"},
          {classifyId:13,classifyName:"逾期已还"},
        ],
        reBorrowList: [
          {classifyId:null,classifyName:"全部状态"},
          {classifyId:0,classifyName:"新户"},
          {classifyId:1,classifyName:"老户"},
        ],
        deferList: [
          {classifyId:null,classifyName:"全部状态"},
          {classifyId:1,classifyName:"已展期"},
        ],
        sexList: [
          {classifyId:null,classifyName:"全部"},
          {classifyId:0,classifyName:"男"},
          {classifyId:1,classifyName:"女"},
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
        status:null,
        reBorrow:null,
        parentChannelName:null,
        childrenChannelName:null,
        sex:null,
        mobile:null,
        cusName:null,
        value4:'',
        value5:'',
        value6:'',
        defer:null,
        startDate:null,
        endDate:null,
        startDateLoan:null,
        endDateLoan:null,
        rePaymentStartDate:null,
        rePaymentEndDate:null,
        centerDialogVisible1:false,
        centerDialogVisible2:false,
        centerDialogVisible3:false,
        centerDialogVisible4:false,
        centerDialogVisible5:false,
        orderId:null,
        ruleForm: {
          shouldRepayment:null,//当前应还总金额
          interest:null,//逾期+罚息
          repaymentDiscountAmount:null,//正常减免金额
          repaymentPaymentAmount:null,//正常实际还款金额
          partialRepayment:null,//部分已还金额
          partialUnpaidAmount:null,//部分还款剩余应还金额
          partialDiscountAmount:null,//部分还款减免金额
          partialRepaymentPayment:null,//部分还款实际还款金额
          repaymentDefer:null,//展期应还金额
          repaymentDeferDiscountAmount:null,//展期减免金额
          repaymentDeferPayment:null,//展期实际还款金额
        },
        rules: {
          repaymentDiscountAmount: [
            { required: true, validator: validatorNumber, trigger: 'blur' }
          ],
          partialRepaymentPayment: [
            { required: true, validator: validatorNumber2, trigger: 'blur' }
          ],
          partialDiscountAmount: [
            { required: true, validator: validatorNumber3, trigger: 'blur' }
          ],
          repaymentDeferDiscountAmount: [
            { required: true, validator: validatorNumber4, trigger: 'blur' }
          ],
        }
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
