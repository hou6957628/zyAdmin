<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/accountManagement' }">账号管理列表</el-breadcrumb-item>
      <el-breadcrumb-item>编辑账户</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="角色名称:" prop="name">
        <el-input v-model="ruleForm.name"></el-input>
      </el-form-item>
      <el-form-item label="手机号:" prop="mobile">
        <el-input v-model="ruleForm.mobile"></el-input>
      </el-form-item>
      <el-form-item label="修改密码:">
        <el-input v-model="ruleForm.password" placeholder="请输入修改的密码"></el-input>
      </el-form-item>
      <el-form-item style="margin-top: 20px" label="角色名称:" prop="roleId">
        <el-select v-model="ruleForm.roleId" value-key="index" placeholder="请选择" @change="selectChange1($event,electData1)">
          <el-option
            v-for="(item,index) in electData1"
            :key="item.id"
            :label="item.roleName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item style="margin-top: 20px" label="管理产品:" prop="productIds">
        <el-select v-model="ruleForm.productIds" value-key="id" multiple placeholder="请选择">
          <el-option
            v-for="item in electData2"
            :key="item.index"
            :label="item.productName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item style="margin-top: 20px" label="状态:" prop="enabled">
        <el-select v-model="ruleForm.enabled" value-key="key" placeholder="请选择">
          <el-option
            v-for="item in electData"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">保存</el-button>
        <el-button type="" @click="resetForm('ruleForm')">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        ruleForm: {
          name: '',
          mobile: '',
          password: '',
          rePassword: '',
          roleId: '',
          roleName: '',
          productIds: [],
          enabled: '',
        },
        rules: {
          name: [
            {required: true, message: '请输入角色名称', trigger: 'blur'}
          ],
          mobile: [
            {required: true, message: '请输入手机号', trigger: 'blur'}
          ],
          password: [
            {required: true, message: '请输入密码', trigger: 'blur' }
          ],
          roleId: [
            {required: true, message: '请选择角色', trigger: 'change'}
          ],
          productIds: [
            {required: true, message: '请选择产品', trigger: 'blur'}
          ],
          enabled: [
            {required: true, message: '请选择是否启用', trigger: 'change'}
          ],
        },
        electData: [
          {key:true,Id:"启用"},
          {key:false,Id:"停用"},
        ],
        electData1: [],
        electData2: [],
        tableData:[],
        products:[],
      };
    },
    methods: {
      //提交按钮
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();
            param.append('id', this.id);
            param.append('name', this.ruleForm.name);
            param.append('mobile', this.ruleForm.mobile);
            param.append('password', this.ruleForm.password);
            param.append('roleId', this.ruleForm.roleId);
            param.append('enabled', this.ruleForm.enabled);
            param.append('productIds', this.ruleForm.productIds);
            axios({
              method: "POST",
              url: "http://"+this.baseUrl+"/operate/admin/account/save",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data: param,
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.$router.push('/accountManagement');
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
      //下拉选择
      selectChange1(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.ruleForm.roleName=obj.roleName;
      },
      //获取账户详情
      getProductList() {
        axios({
          method: "POST",
          url: "http://"+this.baseUrl+"/operate/admin/account/get",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: this.id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.ruleForm = res.data.body;
            this.ruleForm.roleId = res.data.body.roles[0].id;
            this.ruleForm.password = null;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      //获取管理产品列表
      getProductList1(){
        axios({
          method: "POST",
          url: "http://"+this.baseUrl+"/operate/admin/productManage/list",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            pageNum: 1,
            pageSize: 100,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.electData2 = res.data.body.list;
            // for(var i=0;i<this.electData2.length;i++){
            //   this.products.push({
            //     id:this.electData2[i].id.toString(),
            //     productName:this.electData2[i].productName
            //   });
            // }
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //获取角色列表
      getRoleList(){
        axios({
          method: "POST",
          url: "http://"+this.baseUrl+"/operate/admin/role/list",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            pageNum: 1,
            pageSize: 100,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.electData1 = res.data.body.list;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        });
      }
    },
    mounted: function () {
       this.id=this.$route.params.id;
       this.getProductList();
       this.getProductList1();
       this.getRoleList();
    },

  }
</script>

<style scoped>
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }

  .demo-ruleForm {
    width: 600px;
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
