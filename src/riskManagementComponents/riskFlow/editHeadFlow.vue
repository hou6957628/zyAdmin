<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/flowList' }">风控总流程管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑风控总流程</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="总流程名称:" prop="flowName">
        <el-input v-model="ruleForm.flowName"></el-input>
      </el-form-item>
      <el-form-item label="总流程说明:">
        <el-input v-model="ruleForm.flowDetail"></el-input>
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
          {key:false,Id:"不启用"},
          {key:true,Id:"启用"},
        ],
        electValue:'',
        ruleForm: {
          id:'',
          flowName: '',
          flowDetail: '',
          enabled: '',
        },
        rules: {
          flowName: [
            { required: true, message: '请输入总流程名称', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
          ],
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ]
        }
      };
    },
    methods: {
      //根据id查询流程
      getFlowById(ruleId){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/risk/admin/parent_flow/id",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: ruleId,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm = res.data.body.flow;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //提交按钮
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('id', this.ruleForm.id);
            param.append('flowName', this.ruleForm.flowName);
            param.append('flowDetail', this.ruleForm.flowDetail);
            param.append('enabled', this.ruleForm.enabled);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/risk/admin/parent_flow/update",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '修改成功',
                  type: 'success'
                });
                this.$router.push('/flowHeadList');
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
        // console.log(this.ruleForm.enabled);
      },
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      }
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getFlowById(this.id);
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
