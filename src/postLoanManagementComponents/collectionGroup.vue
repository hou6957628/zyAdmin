<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/collectionGroupManagement' }">催收群组管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加催收群组</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="群组名称:" prop="groupName">
        <el-input v-model="ruleForm.groupName"></el-input>
      </el-form-item>
      <el-form-item label="备注:" prop="remark">
        <el-input v-model="ruleForm.remark"></el-input>
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
      //群组名称不可重复
      var validateNameGroup = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入群组名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/operate/admin/group/checkName",
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
      };
      return {
        electData:[
          {key:0,Id:'不启用'},
          {key:1,Id:'启用'},
        ],
        ruleForm: {
          groupName: '',
          remark: null,
          enabled: 1,
        },
        rules: {
          groupName: [
            { required: true, validator: validateNameGroup, trigger: 'blur' }
          ],
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ]
        }
      };
    },
    methods: {
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('groupName', this.ruleForm.groupName);
            param.append('remark', this.ruleForm.remark);
            param.append('enabled', this.ruleForm.enabled);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/operate/admin/group/save",
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
                this.$router.push('/collectionGroupManagement');
              }else if(res.data.msgCd=='ZYCASH-1009'){
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
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      /**
       * 获取催收群组列表
       */
      getProductList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/group/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: this.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body.list;
            this.ruleForm.enabled=res.data.body.enabled;
            this.ruleForm.groupName=res.data.body.groupName;
            this.ruleForm.id=res.data.body.id;
            this.ruleForm.remark=res.data.body.remark;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted:function () {
      this.id=this.$route.params.id;
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
