<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/merchantProductList' }">商户列表</el-breadcrumb-item>
      <el-breadcrumb-item>添加商户</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="商户名称:" prop="merchantName">
        <el-input v-model="ruleForm.merchantName"></el-input>
      </el-form-item>
      <el-form-item label="公司地址:" prop="companyAddress">
        <el-input v-model="ruleForm.companyAddress"></el-input>
      </el-form-item>
      <el-form-item label="公司描述:">
        <el-input type="textarea" v-model="ruleForm.companyDetail"></el-input>
      </el-form-item>
      <el-form-item label="是否启用" prop="enabled">
        <el-select v-model="ruleForm.enabled" placeholder="请选择">
          <el-option
            v-for="item in electData"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
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
  export default {
    data() {
      var validateName = (rule, value, callback) => {
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
        electData:[
          {key:0,Id:'不启用'},
          {key:1,Id:'启用'},
        ],
        ruleForm: {
          merchantName: '',
          companyAddress: '',
          companyDetail: null,
          enabled: 1,
        },
        rules: {
          merchantName: [
            { required: true, validator: validateName, trigger: 'blur' }
          ],
          companyAddress: [
            { required: true, message: '请输入公司地址', trigger: 'blur' }
          ],
          enabled: [
            { required: true, message: '请选择商户名称', trigger: 'change' }
          ]
        }
      };
    },
    methods: {
      //添加商户
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('merchantName', this.ruleForm.merchantName)  // 通过append向form对象添加数据
            param.append('companyAddress', this.ruleForm.companyAddress) // 添加form表单中其他数据
            param.append('companyDetail', this.ruleForm.companyDetail) // 添加form表单中其他数据
            param.append('enabled', this.ruleForm.enabled) // 添加form表单中其他数据
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
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.$router.push('/merchantProductList');
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
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      //编辑产品接口
      editProduct(row){
        var param = new FormData();  // 创建form对象
        param.append('merchantName', this.ruleForm.merchantName);
        param.append('enabled', this.ruleForm.enabled);
        param.append('enableDelete', 0);
        param.append('company', null);
        param.append('companyAddress', this.ruleForm.companyAddress);
        param.append('companyDetail', this.ruleForm.companyDetail);
        param.append('email', null);
        param.append('mobile', null);
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/merchant/get",
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
            this.$router.push('/merchantProductList');
          }else if(res.data.msgCd=='ZYCASH--1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
    },
    mounted:function () {
      //this.getProductList();
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
