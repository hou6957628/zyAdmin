<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/productProductList' }">产品列表</el-breadcrumb-item>
      <el-breadcrumb-item>添加产品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="产品名称:" prop="productName">
        <el-input v-model="ruleForm.productName"></el-input>
      </el-form-item>
      <el-form-item label="产品说明:">
        <el-input v-model="ruleForm.description"></el-input>
      </el-form-item>
      <el-form-item label="商户名称:" prop="merchantId">
        <el-select v-model="ruleForm.merchantId" placeholder="请选择">
          <el-option
            v-for="item in electData"
            :key="item.id"
            :label="item.merchantName"
            :value="item.id">
          </el-option>
        </el-select>&nbsp;&nbsp;
        <el-button @click="addTag()">添加商户</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
        <el-button type="info" @click="resetForm('ruleForm')">取消<i class="el-icon-close el-icon--right"></i></el-button>
      </el-form-item>
    </el-form>
    <!--添加商户结构-->
    <el-dialog
      title="添加商户"
      :visible.sync="centerDialogVisible1"
      width="45%"
      center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="100px" class="demo-ruleForm">
        <el-form-item label="商户名称:" prop="merchantName">
          <el-input v-model="ruleForm2.merchantName"></el-input>
        </el-form-item>
        <el-form-item label="公司地址:" prop="companyAddress">
          <el-input v-model="ruleForm2.companyAddress"></el-input>
        </el-form-item>
        <el-form-item label="公司描述:">
          <el-input v-model="ruleForm2.companyDetail"></el-input>
        </el-form-item>
        <el-form-item label="是否启用:" prop="enabled">
          <el-select v-model="ruleForm2.enabled" placeholder="请选择" @change="selectChange">
            <el-option
              v-for="item in electDataEnabled"
              :key="item.key"
              :label="item.Id"
              :value="item.key">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="addMerchant('ruleForm2')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible1 = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      //验证产品名称是否重复
      var validateName = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入产品名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/operate/admin/productManage/checkRepeateName",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              productName: value
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body == false) {
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
      //验证商户名称是否重复
      var validateName2 = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入商户名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/operate/admin/merchant/checkRepeateName",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              merchantName: value
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body == false) {
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
      return {
        centerDialogVisible1:false,
        electData:[],
        ruleForm: {
          productName: '',
          description: '',
          merchantId: null,
        },
        rules: {
          productName: [
            { required: true, validator: validateName, trigger: 'blur' }
          ],
          merchantId: [
            { required: true, message: '请选择商户名称', trigger: 'change' }
          ]
        },
        electDataEnabled: [
          {key:1,Id:'启用'},
          {key:0,Id:'不启用'},
        ],
        ruleForm2:{
          merchantName:'',
          companyAddress:'',
          companyDetail:'',
          enabled:1,
        },
        rules2: {
          merchantName: [
            { required: true, validator: validateName2, trigger: 'blur' }
          ],
          companyAddress: [
            { required: true, message: '请输入公司地址', trigger: 'blur' }
          ],
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ],
        },
      };
    },
    methods: {
      //添加产品
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('productName', this.ruleForm.productName)  // 通过append向form对象添加数据
            param.append('description', this.ruleForm.description) // 添加form表单中其他数据
            param.append('merchantId', this.ruleForm.merchantId) // 添加form表单中其他数据
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/operate/admin/productManage/createProduct",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.$router.push('/productProductList');
              }else if(res.data.msgCd=='ZYCASH-1009'){
                this.$message.error(res.data.msgInfo);
              }
              else {
                this.$message.error(res);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //获取商户列表
      getProductList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/merchant/getNameList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.electData= res.data.body.list;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //添加商户弹窗
      addTag(){
        this.centerDialogVisible1=true;
      },
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      //添加商户
      addMerchant(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('merchantName', this.ruleForm2.merchantName)  // 通过append向form对象添加数据
            param.append('companyAddress', this.ruleForm2.companyAddress) // 添加form表单中其他数据
            param.append('companyDetail', this.ruleForm2.companyDetail) // 添加form表单中其他数据
            param.append('enabled', this.ruleForm2.enabled) // 添加form表单中其他数据
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/operate/admin/merchant/save",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.centerDialogVisible1 = false;
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.getProductList();
              }else if(res.data.msgCd=='ZYCASH-1009'){
                this.$message.error(res.data.msgInfo);
              }
              else {
                this.$message.error(res);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
    },
    mounted:function () {
      this.getProductList();
    }
  }
</script>

<style scoped>
  .content{
    padding-left: 200px;
    padding-top: 80px;
  }
  .demo-ruleForm{
    width: 600px;
    text-align: left;
  }
  .fs-16{
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .upload-file{
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
