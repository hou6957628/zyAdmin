<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/roleList' }">角色列表</el-breadcrumb-item>
      <el-breadcrumb-item>编辑</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="角色名称:" prop="roleName">
        <el-input v-model="ruleForm.roleName"></el-input>
      </el-form-item>
      <el-form-item label="角色说明:" prop="description">
        <el-input v-model="ruleForm.description"></el-input>
      </el-form-item>
      <p style="font-size: 16px;padding-bottom: 20px">角色权限配置</p>
      <el-tree
        :data="electData"
        show-checkbox
        default-expand-all
        node-key="id"
        ref="tree"
        highlight-current
        :props="defaultProps"
        :default-checked-keys="electData2"
        style="padding-left: 15px;">
      </el-tree>
      <el-form-item style="margin-top: 20px" label="是否启用:" prop="enabled">
        <el-select v-model="ruleForm.enabled" placeholder="请选择" @change="selectChange">
          <el-option
            v-for="item in electData1"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">保存</el-button>
        <!--<el-button type="primary" @click="getCheckedNodes()">获取</el-button>-->
        <!--<el-button type="primary" @click="getCheckedKeys()">获取</el-button>-->
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
        electData:[],
        electData2:[],
        electData1: [
          {key: false, Id: "不启用"},
          {key: true, Id: "启用"},
        ],
        electValue: '',
        ruleForm: {
          roleId: '',
          roleName: '',
          description: '',
          enabled: '',
          authorities:[],
        },
        rules: {
          roleName: [
            {required: true, message: '请输入角色名称', trigger: 'blur'},
          ],
          description: [
            {required: true, message: '请输入角色说明', trigger: 'blur'}
          ],
          enabled: [
            {required: true, message: '请选择是否启用', trigger: 'blur'}
          ]
        },
        defaultProps: {
          children: 'list',
          label: 'name'
        },
        roleCode:''
      };
    },
    methods: {
      //提交按钮
      submitForm(formName) {
        this.getCheckedKeys();
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var data  = {
              'roleId':this.ruleForm.roleId,
              'roleName':this.ruleForm.roleName,
              'description':this.ruleForm.description,
              'enabled':this.ruleForm.enabled,
              'authoritiyIds':this.ruleForm.authorities,
            };
            axios({
              method: "POST",
              url:"http://"+this.baseUrl+"/operate/admin/role/save",
              headers: {
                'Content-Type': 'application/json',
                'Authorization': localStorage.token
              },
              data:JSON.stringify(data),
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.$router.push('/roleList');
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
      selectChange() {
        this.$forceUpdate();
      },
      //获取角色详情
      getProductList(data) {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/operate/admin/role/get",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            roleCode: data,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.ruleForm = res.data.body;
            this.ruleForm.enabled = res.data.body.enable;
            //默认选中
            var _this = this;
            if (this.ruleForm.authorities.length == 0) {
              this.electData2=null;
            } else {
              this.ruleForm.authorities.forEach(function (item,index) {
                if (item.type == 3) {
                  _this.electData2.push(item.id);
                }
              })
            }
            this.getAuthorityList();
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //取消按钮
      resetForm() {
        this.$router.go(-1);
      },
      //获取所有权限
      getAuthorityList() {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/operate/admin/authority/get",
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
            this.electData = res.data.body;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //获取选中的内容
      getCheckedKeys() {
        /*返回选中的id组成的数组*/
        let vv1 = this.$refs.tree.getHalfCheckedKeys();
        let vv2 = this.$refs.tree.getCheckedKeys();
        this.ruleForm.authorities = vv1.concat(vv2);
      },
    },
    mounted: function () {
      this.roleCode=this.$route.params.id;
      this.getProductList(this.roleCode);
    }
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

