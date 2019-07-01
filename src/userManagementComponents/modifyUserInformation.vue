<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/userProductList' }">用户列表</el-breadcrumb-item>
      <el-breadcrumb-item>编辑用户</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="listContent">
      <table>
        <tr><td>用户ID</td><td>{{ruleForm.id}}</td></tr>
        <tr><td>手机号</td><td>{{ruleForm.mobile}}</td></tr>
        <tr><td>姓名</td><td>{{ruleForm.realName}}</td></tr>
        <tr><td>身份证号码</td><td>{{ruleForm.cardNumber}}</td></tr>
        <tr><td>状态</td><td>{{ruleForm.enabled=='false'?'禁用':'启用'}}</td></tr>
        <tr><td>性别</td><td>{{ruleForm.gender}}</td></tr>
        <tr><td>出生日期</td><td>{{ruleForm.birthDate}}</td></tr>
        <tr><td>创建日期</td><td>{{ruleForm.createDate}}</td></tr>
        <tr><td>更新日期</td><td>{{ruleForm.updateDate}}</td></tr>
      </table>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="120px" class="demo-ruleForm">
        <el-form-item label="修改登录密码:" prop="password">
          <el-input v-model="ruleForm.password"></el-input>
        </el-form-item>
        <el-form-item label="确认登录密码:" prop="confirmPassword">
          <el-input v-model="ruleForm.confirmPassword"></el-input>
        </el-form-item>
        <el-form-item label="是否启用:" prop="enabled">
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
          <el-button v-if="hasPermissionCustom('user:customer:update')" type="primary" @click="submitForm('ruleForm')">修改<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info" @click="resetForm('ruleForm')">关闭<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      var validatePass = (rule, value, callback) => {
        if (!value) {
          callback();
        } else if (value.toString().length < 6 || value.toString().length > 32) {
          callback(new Error('密码长度为6 - 32个字符'));
        } else {
          callback();
        }
      };
      var validatePass2 = (rule, value, callback) => {
        if (!value && this.ruleForm.password!=null && this.ruleForm.password!='') {
          callback(new Error('请再次输入密码'));
        } else if (value != this.ruleForm.password) {
          callback(new Error('两次输入密码不一致'));
        } else {
          callback();
        }
      };
      return {
        tableData:[],
        electValue:0,
        electData: [
          {key:false,Id:"不启用"},
          {key:true,Id:"启用"},
        ],
        ruleForm: {
          id: '',
          mobile: '',
          password: null,
          confirmPassword: '',
          gender: '',
          enabled: '',
        },
        rules: {
          password: [
            { required: true, validator: validatePass, trigger: 'blur' }
          ],
          confirmPassword: [
            { required: true, validator: validatePass2, trigger: 'blur' }
          ],
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ]
        }
      };
    },
    methods: {
      //提交按钮
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('id', this.ruleForm.id);  // 这个字段没有
            param.append('mobile', this.ruleForm.mobile); // 通过append向form对象添加数据
            param.append('password', this.ruleForm.password); // 添加0：用户标签 1：规则标签 2:规则集标签
            param.append('enabled', this.ruleForm.enabled);
            axios({
              method: "POST",
              url:"http://"+this.baseUrl+"/user_center/admin/customer/update",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data: param,
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '修改成功',
                  type: 'success'
                });
                this.$router.push('/userProductList');
              } else if (res.data.msgCd == 'ZYCASH-1009') {
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
      //通过id获取用户信息
      getUserById(id) {
        axios({
          method: "GET",
          url:"http://"+this.baseUrl+"/user_center/admin/customer/get",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.ruleForm = res.data.body;
            if(res.data.body.gender==true){
              this.ruleForm.gender='女';
            } else if(res.data.body.gender==false){
              this.ruleForm.gender='男';
            } else {
              this.ruleForm.gender='未知';
            }
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getUserById(this.id);
    }
  }
</script>

<style scoped>
  .demo-ruleForm{
    width: 400px;
    font-size: 16px;
    margin-top: 20px;
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
  table,table tr th, table tr td { border:1px solid #838383; }
  table { width: 40%; min-height: 40px; line-height: 40px; text-align: center; border-collapse: collapse; padding:10px 5px;margin-top: 20px}

</style>
