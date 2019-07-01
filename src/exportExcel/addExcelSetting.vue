<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/exportExcel' }">导出设置</el-breadcrumb-item>
      <el-breadcrumb-item>创建导出设置</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="excel名称:" prop="excelName" :rules="[ruleForm.excelName,{validator:checkName, required: true, trigger:'blur'}]">
        <el-input v-model="ruleForm.excelName"></el-input>
      </el-form-item>
      <el-form-item label="对象列名:" prop="colomnNames">
        <el-input v-model="ruleForm.colomnNames"></el-input>
      </el-form-item>
      <el-form-item label="excel列名:" prop="excelColomnNames">
        <el-input  type="textarea" :autosize="{ minRows: 6, maxRows: 24}"
                  placeholder="请输入内容"  v-model="ruleForm.excelColomnNames"></el-input>
      </el-form-item>
      <el-form-item label="sql语句:" prop="sqlStr">
        <el-input  type="textarea" :autosize="{ minRows: 6, maxRows: 24}"
                   placeholder="请输入内容"  v-model="ruleForm.sqlStr"></el-input>
      </el-form-item>

      <el-form-item label="是否启用:" prop="enable">
        <el-select v-model="ruleForm.enable" placeholder="请选择">
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
          excelName: '',
          colomnNames: '',
          excelColomnNames: '',
          sqlStr: '',
          enable: 1,
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
        } else {
          axios({
            method:"GET",
            url:"http://"+this.baseUrl+"/export/admin/setting/checkName",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              name: value
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
            var param = new FormData();  // 创建form对象
            param.append('excelName', this.ruleForm.excelName);
            param.append('colomnNames', this.ruleForm.colomnNames); // 通过append向form对象添加数据
            param.append('excelColomnNames', this.ruleForm.excelColomnNames); // 通过append向form对象添加数据
            param.append('sqlStr', this.ruleForm.sqlStr); // 通过append向form对象添加数据
            param.append('enable', this.ruleForm.enable);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/export/admin/setting/add",
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
                this.$router.push('/exportExcel');
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
