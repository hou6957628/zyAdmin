<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/ruleClassification' }">规则分类列表</el-breadcrumb-item>
      <el-breadcrumb-item>添加规则分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="分类名称:" prop="classifyName" :rules="[ruleForm.classifyName,{validator:checkName, required: true, trigger:'blur'}]">
        <el-input v-model="ruleForm.classifyName"></el-input>
      </el-form-item>
      <el-form-item label="分类说明:" prop="classifyDescription">
        <el-input v-model="ruleForm.classifyDescription"></el-input>
      </el-form-item>
      <el-form-item label="是否启用:" prop="enabled">
        <el-select v-model="ruleForm.enabled" placeholder="请选择" @change="selectChange">
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
      return {
        electData: [
          {key:0,Id:"不启用"},
          {key:1,Id:"启用"},
        ],
        electValue:'',
        ruleForm: {
          classifyName: '',
          classifyDescription: '',
          enabled: 1,
        },
        rules: {
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ]
        }
      };
    },
    methods: {
      //验证名称是否重名
      checkName(rule, value, callback) {
        if (!value) {
          callback(new Error('请输入名称'));
        } else if (value.length < 3 | value.length > 10) {
          callback(new Error('长度在 3 到 10 个字符'));
        } else {
          axios({
            method:"GET",
            url:"http://"+this.baseUrl+"/risk/admin/classification/getByName",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              name: value,
              id:1
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body != 0) {
                callback(new Error('名称重复'));
              } else {
                callback();
              }
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }
      },
      //提交按钮
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/risk/admin/classification/add",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params:{
                classifyName: this.ruleForm.classifyName,
                description: this.ruleForm.classifyDescription,
                enabled: this.ruleForm.enabled,
                classifyType: 1,
              }
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.$router.push('/ruleClassifyList');
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
      selectChange(){
        console.log(this.ruleForm.enabled);
      },
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      }
    },
    mounted:function () {
      // this.getProductList();
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
