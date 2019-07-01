<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/channelconnectionManagement' }">渠道连接管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加渠道</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :key="1" :rules="rules" ref="ruleForm" label-width="120px" class="demo-ruleForm">
      <el-form-item label="主渠道名称:" prop="parentChannelCode">
        <el-select v-model="ruleForm.parentChannelCode" filterable placeholder="请选择" @change="selectChange1($event,channelList)">
          <el-option
            v-for="item in channelList"
            :key="item.id"
            :label="item.parentChannelName"
            :value="item.parentChannelCode">
          </el-option>
        </el-select>
        <el-button @click="addChannel()" type="primary" style="position: absolute;right:0;top: 0;">创建主渠道</el-button>
      </el-form-item>
      <el-form-item label="渠道账户:" prop="parentChannelCode">
        <el-input v-model="ruleForm.parentChannelCode" disabled="disabled"></el-input>
      </el-form-item>
      <el-form-item label="所属应用:" prop="productId">
        <el-select v-model="ruleForm.productId" disabled="disabled" placeholder="请选择">
          <el-option
            v-for="item in productList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="创建子渠道数:" prop="number">
        <el-tooltip class="item" effect="dark" content="一次性创建请小于10个" placement="right-start">
          <el-input v-model="ruleForm.number" maxlength="10" placeholder="请输入子渠道数"></el-input>
        </el-tooltip>
      </el-form-item>
      <el-form-item label="计价方式:" prop="countType">
        <el-select v-model="ruleForm.countType" placeholder="请选择" @change="selectChange3()">
          <el-option
            v-for="item in countTypeList"
            :key="item.key"
            :label="item.name"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="CPA:" prop="cpaPrice" v-if="ruleForm.countType==1">
        <el-input v-model="ruleForm.cpaPrice" placeholder="请输入CPA单价"></el-input>
      </el-form-item>
      <el-form-item label="CPS:" prop="cpsPrice" v-if="ruleForm.countType==2">
        <el-input v-model="ruleForm.cpsPrice" placeholder="请输入CPS单价"></el-input>
      </el-form-item>
      <el-form-item label="UV:" prop="uvPrice" v-if="ruleForm.countType==3">
        <el-input v-model="ruleForm.uvPrice" placeholder="请输入UV单价"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">保存</el-button>
        <el-button type="" @click="resetForm('ruleForm')">取消</el-button>
      </el-form-item>
    </el-form>
    <!--创建主渠道结构-->
    <el-form :model="ruleForm1" :key="2" :rules="rulesList" ref="ruleForm1" label-width="100px" class="demo-ruleForm">
    <el-dialog
      title="创建主渠道"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-form-item label="主渠道名称:" prop="parentChannelName1">
        <el-input v-model="ruleForm1.parentChannelName1" placeholder="主渠道名称"></el-input>
      </el-form-item>
      <el-form-item label="账号:" prop="account">
        <el-input v-model="ruleForm1.account" placeholder="主渠道账号"></el-input>
      </el-form-item>
      <el-form-item label="密码:" prop="passwd">
        <el-input v-model="ruleForm1.passwd" placeholder="主渠道密码"></el-input>
      </el-form-item>
      <el-form-item label="所属应用:" prop="productId1">
        <el-select v-model="ruleForm1.productId1" placeholder="请选择" @change="selectChange4($event,productList)">
          <el-option
            v-for="item in productList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="备注:" prop="description">
        <el-input v-model="ruleForm1.description" placeholder="备注"></el-input>
      </el-form-item>
      <span slot="footer" class="dialog-footer">
    <el-button @click="centerDialogVisible=false">取 消</el-button>
    <el-button type="primary" @click="submitForm1('ruleForm1')">确 定</el-button>
  </span>
    </el-dialog>
    </el-form>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      //账户名称不能重复
      var validateName1 = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入主渠道名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/channel/admin/account/checkRepetition",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              parentChannelName: value
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body.parentChannelName == 'false') {
                callback(new Error('名称重复'));
              } else {
                callback();
              }
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }
      };
      //账号名称不能重复
      var validateName2 = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入账号名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/channel/admin/account/checkRepetition",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              accout: value
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body.accout == 'false') {
                callback(new Error('名称重复'));
              } else {
                callback();
              }
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }
      };
      //创建子渠道数小于10个
      var validateName = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入创建子渠道数'));
        } else if (value > 10) {
          callback(new Error('一次性创建子渠道数要小于10个'));
        } else {
          callback();
        }
      };
      return {
        ruleForm: {
          parentChannelName: '',
          parentChannelCode: '',
          productId: '',
          productName: '',
          productCode: '',
          number: '',
          countType: '',
          uvPrice: '',
          cpaPrice: '',
          cpsPrice: '',
        },
        rules: {
          parentChannelCode: [
            {required: true, message: '请选择主渠道', trigger: 'change'}
          ],
          productId: [
            {required: true, message: '请选择产品', trigger: 'change'}
          ],
          number: [
            {required: true, validator: validateName, trigger: 'blur'}
          ],
          countType: [
            {required: true, message: '请选择计价方式', trigger: 'change'}
          ],
          cpaPrice: [
            {required: true, message: '请输入CPA单价', trigger: 'blur'}
          ],
          cpsPrice: [
            {required: true, message: '请输入CPS单价', trigger: 'blur'}
          ],
          uvPrice: [
            {required: true, message: '请输入UV单价', trigger: 'blur'}
          ],
        },
        ruleForm1: {
          parentChannelName1: '',
          account: '',
          passwd: '',
          productId1: '',
          productName1: '',
          productCode1: '',
          description: '',
        },
        rulesList: {
          parentChannelName1: [
            {required: true, validator: validateName1, trigger: 'blur'},
          ],
          account: [
            {required: true, validator: validateName2, trigger: 'blur'}
          ],
          passwd: [
            {required: true, message: '请输入密码', trigger: 'blur'},
          ],
          productId1: [
            {required: true, message: '请选择所属应用', trigger: 'change'},
          ],
        },
        channelList:[],
        productList:[],
        tableData:[],
        centerDialogVisible:false,
        centerDialogVisible1:false,
        countTypeList:[
          {key:1,name:'CPA'},
          {key:2,name:'CPS'},
          {key:3,name:'UV'},
        ]
      };
    },
    methods: {
      //获取所有产品
      getFenList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/productMerchantInfo",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productList=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //保存渠道
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method: "POST",
              url: "http://"+this.baseUrl+"/channel/admin/channel/add",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params:{
                parentChannelName: this.ruleForm.parentChannelName,
                parentChannelCode: this.ruleForm.parentChannelCode,
                productName: this.ruleForm.productName,
                productCode: this.ruleForm.productCode,
                productId: this.ruleForm.productId,
                merchantId: this.ruleForm.merchantId,
                merchantName: this.ruleForm.merchantName,
                merchantCode: this.ruleForm.merchantCode,
                number: this.ruleForm.number,
                countType: this.ruleForm.countType,
                cpaPrice: this.ruleForm.cpaPrice,
                cpsPrice: this.ruleForm.cpsPrice,
                uvPrice: this.ruleForm.uvPrice,
              },
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.$router.push('/channelconnectionManagement');
              } else if (res.data.msgCd == 'ZYCASH-1009') {
                this.$message.error(res.data.msgInfo);
              }
              else {
                this.$message.error(res.data.msgInfo);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //添加主渠道
      submitForm1(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method: "POST",
              url: "http://"+this.baseUrl+"/channel/admin/account/add",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params:{
                parentChannelName: this.ruleForm1.parentChannelName1,
                accountName: this.ruleForm1.parentChannelName1,
                account: this.ruleForm1.account,
                passwd: this.ruleForm1.passwd,
                productId: this.ruleForm1.productId1,
                productName: this.ruleForm1.productName1,
                productCode: this.ruleForm1.productCode1,
                description: this.ruleForm1.description,
              },
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.getAccountList();
                this.centerDialogVisible=false;
              } else if (res.data.msgCd == 'ZYCASH-1009') {
                this.$message.error(res.data.msgInfo);
              }
              else {
                this.$message.error(res.data.msgInfo);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //主渠道名称下拉选择
      selectChange1(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.parentChannelCode ===vId;
        });
        this.ruleForm.parentChannelName=obj.parentChannelName;
        this.ruleForm.productId=obj.productId;
        this.ruleForm.productName=obj.productName;
        this.ruleForm.productCode=obj.productCode;
      },
      //所属应用下拉选择
      selectChange2(vId,list){
        let obj1 = {};
        obj1 = list.find((item)=>{
          return item.productId ===vId;
        });
        this.ruleForm.productName=obj1.productName;
        this.ruleForm.productCode=obj1.productCode;
        this.ruleForm.merchantId=obj1.merchantId;
        this.ruleForm.merchantName=obj1.merchantName;
        this.ruleForm.merchantCode=obj1.merchantCode;
      },
      //所属应用下拉选择
      selectChange4(vId,list){
        let obj1 = {};
        obj1 = list.find((item)=>{
          return item.productId ===vId;
        });
        this.ruleForm1.productName1=obj1.productName;
        this.ruleForm1.productCode1=obj1.productCode;
      },
      //选择计价方式
      selectChange3(){
        if (this.ruleForm.countType == 1) {
          this.ruleForm.cpsPrice='';
          this.ruleForm.uvPrice='';
        } else if (this.ruleForm.countType == 2) {
          this.ruleForm.cpaPrice='';
          this.ruleForm.uvPrice='';
        } else if (this.ruleForm.countType == 3) {
          this.ruleForm.cpsPrice='';
          this.ruleForm.cpaPrice='';
        }
      },
      //获取所有账户
      getAccountList() {
        axios({
          method: "GET",
          url: "http://"+this.baseUrl+"/channel/admin/account/list",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: 1,
            pageSize: 600,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.channelList = res.data.body.list;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //取消按钮
      resetForm() {
        this.$router.go(-1);
      },
      //跳转到主渠道
      addChannel(){
        this.centerDialogVisible=true;
      },
    },
    mounted: function () {
      this.getAccountList();
      this.getFenList();
    }
  }
</script>

<style scoped>
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }

  .demo-ruleForm {
    width: 500px;
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
</style>
