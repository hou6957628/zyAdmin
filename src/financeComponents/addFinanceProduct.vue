<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/productList' }">金融产品列表</el-breadcrumb-item>
      <el-breadcrumb-item>产品配置</el-breadcrumb-item>
    </el-breadcrumb>
    <hr>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="150px" class="demo-ruleForm">
      <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>基本信息</el-breadcrumb-item>
      </el-breadcrumb>
      <el-form-item label="产品名称" prop="name">
        <el-input v-model="ruleForm.name" placeholder="请填写产品名称"></el-input>
      </el-form-item>
      <el-form-item label="产品说明" prop="description">
        <el-input type="textarea" v-model="ruleForm.description" placeholder="请填写对此产品的备注信息"></el-input>
      </el-form-item>
      <el-form-item label="产品类型" prop="electValueType">
        <el-select v-model="ruleForm.electValueType">
          <el-option
            v-for="item in electDataType"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="是否启用" prop="electValueEnabled">
        <el-select v-model="ruleForm.electValueEnabled">
          <el-option
            v-for="item in electDataEnabled"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>金融产品信息</el-breadcrumb-item>
      </el-breadcrumb>
      <el-form-item label="最小金额" prop="minCapital">
        <el-input v-model="ruleForm.minCapital" placeholder="请输入产品最小金额"></el-input>
      </el-form-item>
      <el-form-item label="最大金额" prop="maxCapital">
        <el-input v-model="ruleForm.maxCapital" placeholder="请输入产品最大金额"></el-input>
      </el-form-item>
      <el-form-item label="金额递增倍数" prop="capitalMultiple">
        <el-input v-model="ruleForm.capitalMultiple" placeholder="请输入产品金额递增倍数"></el-input>
      </el-form-item>
      <el-form-item label="计息方式" prop="electValueMethod">
        <el-select v-model="ruleForm.electValueMethod">
          <el-option
            v-for="item in electDataMethod"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="周期类型" prop="electValuePeriodType">
        <el-select v-model="ruleForm.electValuePeriodType">
          <el-option
            v-for="item in electDataPeriodType"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="计息周期" prop="electValueInterestPeriod">
        <el-select v-model="ruleForm.electValueInterestPeriod">
          <el-option
            v-for="item in electDataInterestPeriod"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="利率周期" prop="electValueInterestRatePeriod">
        <el-select v-model="ruleForm.electValueInterestRatePeriod">
          <el-option
            v-for="item in electDataInterestRatePeriod"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="周期及利率" prop="electDataList">
          <!--周期：<el-input style="display: inline-block;width: 200px;margin-right: 5px;margin-bottom: 10px;" v-model="electDataList.domains.period"></el-input>-->
          <!--利率：<el-input style="display: inline-block;width: 200px;margin-right: 5px;margin-bottom: 10px;" v-model="electDataList.domains.rate"></el-input>-->
        <el-form-item v-for="(domain, index) in electDataList.domains" :key="index">
          周期：<el-input style="display: inline-block;width: 130px;margin-right: 5px;margin-bottom: 10px;" v-model.number="domain.period"></el-input>
          金额：<el-input style="display: inline-block;width: 130px;margin-right: 5px;margin-bottom: 10px;" v-model.number="domain.amount"></el-input>
          利率：<el-input style="display: inline-block;width: 130px;margin-right: 5px;margin-bottom: 10px;" v-model.number="domain.rate"></el-input>
          <el-button style="display: inline-block;width: 100px" @click.prevent="removeDomain(domain)">删除</el-button>
        </el-form-item>
        <el-form-item>
          <el-button @click="addDomain">添加</el-button>
        </el-form-item>
      </el-form-item>
      <el-form-item label="是否支持循环贷" prop="electValueRevolvingLoan">
        <el-select v-model="ruleForm.electValueRevolvingLoan">
          <el-option
            v-for="item in electDataRevolvingLoan"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="起息时间" prop="interestTime">
        <el-input v-model="ruleForm.interestTime" placeholder="例: “0”表示放款当天计息"></el-input>
      </el-form-item>
      <el-form-item label="借款服务费率" prop="borrowingFeeRateValue">
        <el-input v-model="ruleForm.borrowingFeeRateValue" placeholder="请输入借款服务费率"></el-input>
      </el-form-item>
      <el-form-item label="借款服务费收取方式" prop="electValueBorrowingFeePayRule">
        <el-select v-model="ruleForm.electValueBorrowingFeePayRule">
          <el-option
            v-for="item in electDataBorrowingFeePayRule"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="还款服务费率" prop="repaymentFeeRateValue">
        <el-input v-model="ruleForm.repaymentFeeRateValue" placeholder="请输入还款服务费率"></el-input>
      </el-form-item>
      <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>提前还款配置</el-breadcrumb-item>
      </el-breadcrumb>
      <el-form-item label="是否允许提前还款" prop="electValueEnabledPrepayment">
        <el-select v-model="ruleForm.electValueEnabledPrepayment">
          <el-option
            v-for="item in electDataEnabledPrepayment"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="提前还款规则" prop="electValuePrepaymentRule">
        <el-select v-model="ruleForm.electValuePrepaymentRule">
          <el-option
            v-for="item in electDataPrepaymentRule"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="提前还款锁定天数" prop="prepaymentLockedTime">
        <el-input v-model="ruleForm.prepaymentLockedTime" placeholder="例: “1”表示放款后，1天内不能操作提前还款"></el-input>
      </el-form-item>
      <el-form-item label="提前还款罚率" prop="repaymentLeadPenaltyRateValue">
        <el-input v-model="ruleForm.repaymentLeadPenaltyRateValue" placeholder="请输入提前还款罚率"></el-input>
      </el-form-item>
      <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>逾期及罚息</el-breadcrumb-item>
      </el-breadcrumb>
      <el-form-item label="逾期还款时间" prop="electValueRepaymentLagStartTime">
        <el-select v-model="ruleForm.electValueRepaymentLagStartTime">
          <el-option
            v-for="item in electDataRepaymentLagStartTime"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="逾期罚率" prop="repaymentLagPenaltyRateValue">
        <el-input v-model="ruleForm.repaymentLagPenaltyRateValue" placeholder="请输入逾期罚率"></el-input>
      </el-form-item>
      <el-form-item label="容时期时间" prop="repaymentLagTime">
        <el-input v-model="ruleForm.repaymentLagTime" placeholder="请输入容时期时间"></el-input>
      </el-form-item>
      <el-form-item label="容时期罚率" prop="repaymentLagRateValue">
        <el-input v-model="ruleForm.repaymentLagRateValue" placeholder="请输入容时期罚率"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
        <el-button type="info" @click="resetForm('ruleForm')">取消<i class="el-icon-close el-icon--right"></i></el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import axios from 'axios'
  import qs from 'qs'
  export default {
    data() {
      return {
        electDataType:[
          {key:0,Id:'现金贷产品'},
          {key:1,Id:'现金分期产品'},
        ],
        electDataEnabled:[
          {key:0,Id:'否'},
          {key:1,Id:'是'},
        ],
        electDataMethod:[
          {key:0,Id:'每期等额本息'},
          {key:1,Id:'每期付息到期还本'},
          {key:2,Id:'到期还本付息'},
          {key:3,Id:'等本等息'},
          {key:4,Id:'等额本金'},
          {key:5,Id:'放款时一次性收取'},
        ],
        electDataPeriodType:[
          {key:0,Id:'年'},
          {key:1,Id:'月'},
          {key:2,Id:'天'},
        ],
        electDataInterestPeriod:[
          {key:0,Id:'年'},
          {key:1,Id:'月'},
          {key:2,Id:'天'},
        ],
        electDataInterestRatePeriod:[
          {key:0,Id:'年'},
          {key:1,Id:'月'},
          {key:2,Id:'天'},
        ],
        electDataRevolvingLoan:[
          {key:0,Id:'否'},
          {key:1,Id:'是'},
        ],
        electDataBorrowingFeePayRule:[
          {key:0,Id:'放款金额中扣除'},
          {key:1,Id:'额外先期支付'},
        ],
        electDataEnabledPrepayment:[
          {key:0,Id:'否'},
          {key:1,Id:'是'},
        ],
        electDataPrepaymentRule:[
          {key:0,Id:'收取剩余期限罚息'},
          {key:1,Id:'提前还款金额'},
          {key:2,Id:'按还款计划'},
        ],
        electDataRepaymentLagStartTime:[
          {key:0,Id:'还款到期隔天开始计算逾期费用'},
        ],
        electDataList: {
          domains: [
            {period: null,rate: null,amount: null},
            ]
        }
        ,
        ruleForm: {
          name: '',
          description: '',
          electValueType:'',
          electValueEnabled:1,
          minCapital: '',
          maxCapital: '',
          capitalMultiple: '',
          electValueInterestPeriod: '',
          electValueMethod: '',
          electValuePeriodType: '',
          electValueInterestRatePeriod: '',
          electValueRevolvingLoan: '',
          interestTime: '',
          borrowingFeeRateValue:'',
          electValueBorrowingFeePayRule:'',
          repaymentFeeRateValue:'',
          electValueEnabledPrepayment:'',
          electValuePrepaymentRule:'',
          prepaymentLockedTime:'',
          repaymentLeadPenaltyRateValue:'',
          electValueRepaymentLagStartTime:'',
          repaymentLagPenaltyRateValue:'',
          repaymentLagTime:'',
          repaymentLagRateValue:''
        },
        rules: {
          name: [
            {required: true, message: '请填写产品名称', trigger: 'blur'}
          ],
          description: [
            {required: true, message: '请填写产品说明', trigger: 'blur'}
          ],
          electValueType: [
            {required: true, message: '请选择产品类型', trigger: 'blur'}
          ],
          electValueEnabled: [
            {required: true, message: '请选择是否启用', trigger: 'blur'}
          ],
          minCapital: [
            {required: true, message: '请填写最小金额', trigger: 'blur'}
          ],
          maxCapital: [
            {required: true, message: '请填写最大金额', trigger: 'blur'}
          ],
          capitalMultiple: [
            {required: true, message: '请填写金额递增倍数', trigger: 'blur'},
            {min: 3, max: 3, message: '金额递增倍数', trigger: 'blur'}
          ],
          electValueInterestPeriod: [
            {required: true, message: '请选择计息周期', trigger: 'blur'}
          ],
          electValueMethod: [
            {required: true, message: '请选择计息方式', trigger: 'blur'}
          ],
          electValuePeriodType: [
            {required: true, message: '请选择周期类型', trigger: 'blur'}
          ],
          electValueInterestRatePeriod: [
            {required: true, message: '请选择利率周期', trigger: 'blur'}
          ],
          domains: [
            {required: true, message: '请填写周期及利率', trigger: 'blur'}
          ],
          electValueRevolvingLoan: [
            {required: true, message: '请选择是否支持循环贷', trigger: 'blur'}
          ],
          interestTime:[
            {required: true, message: '请选择起息时间', trigger: 'blur'}
          ],
          borrowingFeeRateValue: [
            {required: true, message: '请填写借款服务费率', trigger: 'blur'}
          ],
          electValueBorrowingFeePayRule: [
            {required: true, message: '请选择借款服务费收取方式', trigger: 'blur'}
          ],
          repaymentFeeRateValue: [
            {required: true, message: '请填写还款服务费率', trigger: 'blur'}
          ],
          electValueEnabledPrepayment: [
            {required: true, message: '请选择是否允许提前还款', trigger: 'blur'}
          ],
          electValuePrepaymentRule: [
            {required: true, message: '请选择提前还款规则', trigger: 'blur'}
          ],
          prepaymentLockedTime: [
            {required: true, message: '请填写提前还款锁定天数', trigger: 'blur'}
          ],
          repaymentLeadPenaltyRateValue: [
            {required: true, message: '请填写提前还款罚率', trigger: 'blur'}
          ],
          electValueRepaymentLagStartTime: [
            {required: true, message: '请选择逾期还款时间', trigger: 'blur'}
          ],
          repaymentLagPenaltyRateValue: [
            {required: true, message: '请填写逾期罚率', trigger: 'blur'}
          ],
          repaymentLagTime: [
            {required: true, message: '请填写容时期时间', trigger: 'blur'}
          ],
          repaymentLagRateValue: [
            {required: true, message: '请填写容时期罚率', trigger: 'blur'}
          ]
        }
      };
    },
    methods: {
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var data  = {
              'name': this.ruleForm.name,
              'description': this.ruleForm.description,
              'type': this.ruleForm.electValueType,
              'enabled': this.ruleForm.electValueEnabled,
              'minCapital': this.ruleForm.minCapital,
              'maxCapital': this.ruleForm.maxCapital,
              'capitalMultiple': this.ruleForm.capitalMultiple,
              'interestPeriod': this.ruleForm.electValueInterestPeriod,
              'interestMethods': this.ruleForm.electValueMethod,
              'periodType': this.ruleForm.electValuePeriodType,
              'interestRatePeriod': this.ruleForm.electValueInterestRatePeriod,
              'revolvingLoan': this.ruleForm.electValueRevolvingLoan,
              'interestTime': this.ruleForm.interestTime,
              'borrowingFeeRateValue': this.ruleForm.borrowingFeeRateValue,
              'borrowingFeePayRule': this.ruleForm.electValueBorrowingFeePayRule,
              'repaymentFeeRateValue': this.ruleForm.repaymentFeeRateValue,
              'enabledPrepayment': this.ruleForm.electValueEnabledPrepayment,
              'prepaymentRule': this.ruleForm.electValuePrepaymentRule,
              'prepaymentLockedTime': this.ruleForm.prepaymentLockedTime,
              'repaymentLeadPenaltyRateValue': this.ruleForm.repaymentLeadPenaltyRateValue,
              'repaymentLagStartTime': this.ruleForm.electValueRepaymentLagStartTime,
              'repaymentLagPenaltyRateValue': this.ruleForm.repaymentLagPenaltyRateValue,
              'repaymentLagTime': this.ruleForm.repaymentLagTime,
              'repaymentLagRateValue': this.ruleForm.repaymentLagRateValue,
              'list': this.electDataList.domains
            };
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/operate/admin/product/addProduct",
              headers:{
                'Content-Type':'application/json',
                'Authorization': localStorage.token
              },
              data: JSON.stringify(data),
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.$router.push('/financeProductList');
              }else if(res.data.msgCd=='ZYCASH-SUPERMARKET-1009'){
                this.$message.error(res.data.msgInfo);
              } else {
                this.$message.error(res);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      removeDomain(item) {
        var index = this.electDataList.domains.indexOf(item)
        if (index !== -1) {
          this.electDataList.domains.splice(index, 1)
        }
      },
      addDomain() {
        this.electDataList.domains.push({
          period: null,
          amount: null,
          rate: null,
        });
      }
    }
  }
</script>

<style scoped>
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }

  .demo-ruleForm {
    width: 800px;
    text-align: left;
  }

  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }

  .upload-file {
    position: relative;
    display: inline-block;
    background: #D0EEFF;
    border: 1px solid #99D3F5;
    border-radius: 4px;
    padding: 4px 12px;
    overflow: hidden;
    color: #1E88C7;
    text-decoration: none;
    text-indent: 0;
    line-height: 30px;
    height: 30px;
  }

  .upload-file input {
    position: absolute;
    font-size: 100px;
    right: 0;
    top: 0;
    opacity: 0;
  }

  .upload-file:hover {
    background: #AADFFD;
    border-color: #78C3F3;
    color: #004974;
    text-decoration: none;
  }
  .content .el-form-item__label{
    width: 150px;
  }
</style>
