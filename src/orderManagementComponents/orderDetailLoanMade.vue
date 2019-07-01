<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/pendingApproval' }">已放款列表</el-breadcrumb-item>
      <el-breadcrumb-item>详情</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="jiben">
    <h3>订单信息</h3>
    <table>
      <tr>
        <td>订单号：{{this.borrowingForm.orderId}}</td>
        <td>手机号：{{this.borrowingForm.mobile}}</td>
        <td>渠道：{{this.borrowingForm.parentChannelName==null?'--':this.borrowingForm.parentChannelName}}</td>
        <td>订单状态：{{this.borrowingForm.status | statusFalse}}</td>
        <td>新户老户：{{this.borrowingForm.reBorrow==true?'老户':'新户'}}</td>
        <td>所属平台：{{this.borrowingForm.productName}}</td>
      </tr>
      <tr>
        <td>申请时间：{{this.borrowingForm.createDate}}</td>
        <td>放款时间：{{this.borrowingForm.borrowingPaymentDate==null?'--':this.borrowingForm.borrowingPaymentDate}}</td>
        <td>预计还款时间：{{this.borrowingForm.repaymentEndDate}}</td>
        <td>实际还款时间：{{this.borrowingForm.repaymentPaymentDate==null?'--':this.borrowingForm.repaymentPaymentDate}}</td>
        <td>借款金额：{{this.borrowingForm.borrowingCapital}}</td>
        <td>期限：{{this.borrowingForm.borrowingPeriod}}</td>
      </tr>
      <tr>
        <td>是否逾期：{{this.borrowingForm.repaymentOverdueDay==null?'否':'是'}}</td>
        <td>逾期天数：{{this.borrowingForm.repaymentOverdueDay}}</td>
        <td>应还利息（元）：{{this.borrowingForm.borrowingInterest}}</td>
        <td>罚息（元）：{{this.borrowingForm.repaymentPenaltyInterest}}</td>
        <td>滞纳金（元）：{{this.borrowingForm.repaymentOverdueFee}}</td>
        <td>应还总金额（元）：{{this.borrowingForm.repaymentCapital + this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest}}</td>
      </tr>
      <tr>
        <td>是否可展期：{{this.borrowingForm.enableDefer | enableDeferFalse}}</td>
        <td>展期应还金额：{{this.borrowingForm.repaymentDefer}}</td>
        <td>展期实际还款金额（元）：{{this.borrowingForm.repaymentDeferPayment}}</td>
        <td>减免金额：{{this.borrowingForm.repaymentDiscountAmount}}</td>
      </tr>
      <tr>
        <td>催收员：{{this.borrowingForm.collectorName}}</td>
      </tr>
    </table>
    </div>
    <el-button-group v-if="this.borrowingForm.partial" style="margin: 0 auto;width: auto;display: block;margin-top: 40px;margin-bottom: 40px">
      <el-button v-if="this.borrowingForm.partial && hasPermissionCustom('order:assigned:partial')" class="la" type="danger" @click="partialTip()">部分还款</el-button>
      <el-button class="la" type="danger" @click="resetForm()">关闭</el-button>
    </el-button-group>
    <div v-if="!this.borrowingForm.partial" style="margin: 0 auto;width: auto;display: block;margin-top: 40px;margin-bottom: 40px">
      <el-button-group v-if="this.borrowingForm.defer" style="margin: 0 auto;width: auto;display: block;margin-top: 40px;margin-bottom: 40px">
        <el-button v-if="this.borrowingForm.defer && hasPermissionCustom('order:assigned:deffer')" class="la" type="danger" @click="lineDefferTip()">展期</el-button>
        <el-button class="la" type="danger" @click="resetForm()">关闭</el-button>
      </el-button-group>
      <el-button-group v-if="!this.borrowingForm.defer" style="margin: 0 auto;width: auto;display: block;margin-top: 40px;margin-bottom: 40px">
        <el-button v-if="hasPermissionCustom('order:assigned:offlineRepayment') || hasPermissionCustom('order:collectionTask:offlineRepayment') || hasPermissionCustom('order:orderAll:offlineRepayment')"
                   class="la" type="danger" @click="offlineRepaymentTip()">线下还款</el-button>
        <el-button v-if="hasPermissionCustom('order:assigned:onlineRelief') || hasPermissionCustom('order:collectionTask:onlineRelief') || hasPermissionCustom('order:orderAll:onlineRelief')"
                   class="la" style="margin-left: 10px" type="danger" @click="onlineReliefTip()">线上减免</el-button>
        <el-button v-if="hasPermissionCustom('order:assigned:separateDeduction') || hasPermissionCustom('order:collectionTask:separateDeduction') || hasPermissionCustom('order:orderAll:separateDeduction')"
                   class="la" style="margin-left: 10px" type="danger" @click="separateDeductionTip()">单独扣款</el-button>
        <el-button v-if="this.borrowingForm.enableDefer && hasPermissionCustom('order:assigned:deffer') || hasPermissionCustom('order:collectionTask:deffer') || hasPermissionCustom('order:orderAll:deffer')"
                   class="la" style="margin-left: 10px" type="danger" @click="lineDefferTip()">展期还款</el-button>
        <el-button v-if="!this.borrowingForm.enableDefer && hasPermissionCustom('order:assigned:deffer') || hasPermissionCustom('order:collectionTask:deffer') || hasPermissionCustom('order:orderAll:deffer')"
                   class="la" style="margin-left: 10px" type="danger" @click="lineDefferTip()">特例展期</el-button>
        <el-button v-if="hasPermissionCustom('order:assigned:partial') || hasPermissionCustom('order:collectionTask:partial') || hasPermissionCustom('order:orderAll:partial')"
                   class="la" style="margin-left: 10px" type="danger" @click="partialTip()">部分还款</el-button>
        <el-button class="la" style="margin-left: 10px" type="danger" @click="resetForm()">关闭</el-button>
      </el-button-group>
    </div>
    <div class="listContent">
      <router-link :to="'/jibenOrder2/'+this.id+'/'+this.orderId2" tag="li">基本信息</router-link>
      <router-link v-if="hasPermissionCustom('order:assigned:hitList') || hasPermissionCustom('order:collectionTask:hitList') || hasPermissionCustom('order:orderAll:hitList')"
                   :to="'/fenxianOrder2/' + this.id" tag="li">风险命中列表</router-link>
      <router-link :to="'/yunyingOrder2/' + this.id" tag="li">运营商通讯录比对</router-link>
      <template v-if="hasPermissionCustom('order:assigned:tianji') || hasPermissionCustom('order:collectionTask:tianji') || hasPermissionCustom('order:orderAll:tianji')">
        <a :href="this.tianjiReport.tianjiUrl | htmlFalse" target="_blank" class="ddd">天机报告</a>
      </template>
      <a href="http://www.baidu.com" target="_blank" class="ddd">支付宝报告</a>
      <router-link :to="'/yonghuOrder2/' + this.id" tag="li">用户催收记录</router-link>
      <router-link :to="'/dingdanOrder2/' + this.id" tag="li">订单记录</router-link>
      <router-link :to="'/fangkuanOrder2/' + this.id" tag="li">放款记录</router-link>
      <router-link :to="'/huankuanOrder2/' + this.id" tag="li">还款记录</router-link>
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
    <router-view/>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
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
        borrowingForm:[],
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
        orderId2:'',
        authorizationStatus:[],
        basicInfo:{},
        tableData:[],
        electValue:0,
        isactive:0,
        centerDialogVisible1:false,
        centerDialogVisible2:false,
        centerDialogVisible3:false,
        centerDialogVisible4:false,
        centerDialogVisible5:false,
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
      };
    },
    methods: {
      //线下还款弹窗
      offlineRepaymentTip(){
        this.centerDialogVisible1=true;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=this.borrowingForm.repaymentCapital + this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //获取减免金额
        this.ruleForm.repaymentDiscountAmount=this.borrowingForm.repaymentDiscountAmount;
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
                  orderId:this.orderId2,
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
                  this.$router.go(-1);
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
      onlineReliefTip(){
        this.centerDialogVisible2=true;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=this.borrowingForm.repaymentCapital + this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //获取减免金额
        this.ruleForm.repaymentDiscountAmount=this.borrowingForm.repaymentDiscountAmount;
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
                  orderId:this.orderId2,
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
                  this.$router.go(-1);
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
        //计算逾期费用 + 罚息
        this.ruleForm.interest=this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=this.borrowingForm.repaymentCapital + this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //获取减免金额
        this.ruleForm.repaymentDiscountAmount=this.borrowingForm.repaymentDiscountAmount;
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
                  orderId:this.orderId2,
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
                  this.$router.go(-1);
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
      lineDefferTip(){
        this.centerDialogVisible4=true;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //计算展期应还金额
        this.ruleForm.repaymentDefer=this.borrowingForm.borrowingInterest + this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=this.borrowingForm.repaymentCapital + this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //获取减免金额
        this.ruleForm.repaymentDeferDiscountAmount=this.borrowingForm.repaymentDeferDiscountAmount;
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
                  orderId:this.orderId2,
                  discountAmount:this.ruleForm.repaymentDeferDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentDeferPayment,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible3=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.$router.go(-1);
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
                  orderId:this.orderId2,
                  discountAmount:this.ruleForm.repaymentDeferDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentDeferPayment,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible3=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.$router.go(-1);
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
                  orderId:this.orderId2,
                  discountAmount:this.ruleForm.repaymentDeferDiscountAmount,
                  paymentAmount: this.ruleForm.repaymentDeferPayment,
                }
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible3=false;
                  this.$message({
                    message: '操作成功',
                    type: 'success'
                  });
                  this.$router.go(-1);
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
      partialTip(){
        this.centerDialogVisible5=true;
        //计算逾期费用 + 罚息
        this.ruleForm.interest=this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //计算应还金额：还款本金 + 逾期费用 + 罚息
        this.ruleForm.shouldRepayment=this.borrowingForm.repaymentCapital + this.borrowingForm.repaymentOverdueFee + this.borrowingForm.repaymentPenaltyInterest;
        //剩余应还金额
        if (this.borrowingForm.partial) {
          this.ruleForm.partialUnpaidAmount=this.borrowingForm.partialUnpaidAmount;
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
                  orderId:this.orderId2,
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
                  this.$router.go(-1);
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
      //取消按钮
      resetForm() {
        this.$router.go(-1);
      },
      //用户基本信息
      getUserDetail1(id) {
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
            this.tianjiReport.tianjiUrl = res.data.body.tianjiReport.html;
            this.idCard = res.data.body.idCard;
            this.idFace = res.data.body.idFace;
            this.linkMan = res.data.body.linkMan;
            this.authorizationStatus = res.data.body.authorizationStatus;
            this.basicInfo = res.data.body.basicInfo;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //用户订单信息
      getOrderInfo(id) {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/info",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            orderId: id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.borrowingForm = res.data.body;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.orderId2=this.$route.params.orderId;
      this.getUserDetail1(this.id);
      this.getOrderInfo(this.orderId2);
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
      },
      enableDeferFalse:function(arg1){
        if(arg1==null){
          return "未跑展期流程";
        }else if(arg1==false){
          return "否";
        }else if(arg1==true){
          return "是";
        }
      },
    }
  }
</script>

<style scoped>
  .jiben{
    margin-top: 20px;
    font-size: 14px;
    margin-bottom: 40px;
  }
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
    padding: 10px 20px;
    margin: 0 5px;
    font-size: 17px;
    color: #000;
    background-color: #dedede;
    border: 1px solid #b0b0b0;
    border-radius: 10px;
    cursor: pointer;
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
    background-color: #118efe;
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
