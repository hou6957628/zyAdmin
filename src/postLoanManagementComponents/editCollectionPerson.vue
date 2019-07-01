<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/collectionPersonManagement' }">催收人员管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加催收人员</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="姓名:" prop="name">
        <el-input v-model="ruleForm.name"></el-input>
      </el-form-item>
      <el-form-item label="手机号:" prop="mobile">
        <el-input v-model="ruleForm.mobile"></el-input>
      </el-form-item>
      <el-form-item label="密码:" prop="password">
        <el-input v-model="ruleForm.password"></el-input>
      </el-form-item>
      <el-form-item label="确认密码:" prop="rePassword">
        <el-input v-model="ruleForm.rePassword"></el-input>
      </el-form-item>
      <el-form-item label="所属群组" prop="groupId">
        <el-select v-model="ruleForm.groupId" value-key="id" placeholder="请选择">
          <el-option
            v-for="item in operationGroupList"
            :key="item.id"
            :label="item.groupName"
            :value="item.id">
          </el-option>
        </el-select>
        <el-button v-if="hasPermissionCustom('operate:collection:addGroup')" @click="addGroup()" style="margin-left: 50px">创建群组</el-button>
      </el-form-item>
      <el-form-item label="角色" prop="roleId">
        <el-select v-model="ruleForm.roleId" value-key="id" placeholder="请选择">
          <el-option
            v-for="item in operationRoleList"
            :key="item.id"
            :label="item.roleName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="组长或组员" prop="groupRole">
        <el-select v-model="ruleForm.groupRole" value-key="id" placeholder="请选择">
          <el-option
            v-for="item in groupRoleList"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="催收产品" prop="productIds">
        <el-select v-model="ruleForm.productIds" value-key="productId" multiple placeholder="请选择" @change="selectChange">
          <el-option
            v-for="item in operationMerchantProductList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="是否启用" prop="enabled">
        <el-select v-model="ruleForm.enabled" placeholder="请选择" value-key="key">
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
    <!--添加群组结构-->
    <el-dialog
      title="创建群组"
      :visible.sync="centerDialogVisible1"
      width="45%"
      center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="100px" class="demo-ruleForm">
        <el-form-item label="群组名称:" prop="groupName">
          <el-input v-model="ruleForm2.groupName"></el-input>
        </el-form-item>
        <el-form-item label="备注:">
          <el-input v-model="ruleForm2.remark"></el-input>
        </el-form-item>
        <el-form-item label="状态:" prop="enabled">
          <el-select v-model="ruleForm2.enabled" placeholder="请选择">
            <el-option
              v-for="item in electData"
              :key="item.key"
              :label="item.Id"
              :value="item.key">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="addGroupSubmit('ruleForm2')">保存<i class="el-icon-check el-icon--right"></i></el-button>
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
      //催收员姓名最多10个汉字姓名不可重复
      var validateName = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入催收员姓名'));
        }  else if (value.length > 10) {
          callback(new Error('长度须小于10个字符'));
        } else {
          callback();
        }
      };
      //验证密码长度
      var validatePass = (rule, value, callback) => {
        if (!value) {
          callback();
        }  else if (value.length < 6) {
          callback(new Error('长度最少6个字符'));
        } else {
          callback();
        }
      };
      //验证确认密码
      var validatePass2 = (rule, value, callback) => {
        if (!value && !this.ruleForm.password) {
          callback();
        }  else if (value !== this.ruleForm.password) {
          callback(new Error('两次输入密码不一致!'));
        } else {
          callback();
        }
      };
      //手机号验证
      var checkPhone = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('手机号不能为空'));
        } else {
          const reg = /^1[3|4|5|7|8][0-9]\d{8}$/
          console.log(reg.test(value));
          if (reg.test(value)) {
            callback();
          } else {
            return callback(new Error('请输入正确的手机号'));
          }
        }
      };
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
        centerDialogVisible1:false,
        electData:[
          {key:false,Id:'不启用'},
          {key:true,Id:'启用'},
        ],
        operationRoleList: [],
        groupRoleList:[
          {key:1,Id:'组长'},
          {key:2,Id:'组员'},
        ],
        operationGroupList: [],
        operationMerchantProductList: [],
        operationMerchantProductList1: [],
        ruleForm: {
          id:'',
          name: '',
          mobile: '',
          password: '',
          rePassword: null,
          groupId: '',
          roleId: '',
          groupRole: '',
          productIds:[],
          enabled: ''
        },
        rules: {
          name: [
            { required: true, validator: validateName, trigger: 'blur' }
          ],
          mobile: [
            {  required: true, validator: checkPhone, trigger: 'blur' }
          ],
          password: [
            {  required: true, validator: validatePass, trigger: 'blur' }
          ],
          rePassword: [
            {  required: true, validator: validatePass2, trigger: 'blur' }
          ],
          groupId: [
            {  required: true, message: '请选择所属群组', trigger: 'change' }
          ],
          roleId: [
            {  required: true, message: '请选择角色', trigger: 'change' }
          ],
          groupRole: [
            {  required: true, message: '请选择组长或组员', trigger: 'change' }
          ],
          productIds: [
            {  required: true, message: '请选择催收产品', trigger: 'change' }
          ],
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ]
        },
        ruleForm2: {
          groupName:'',
          remark:'',
          enabled:true,
        },
        rules2: {
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
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/operate/admin/collection/save",
              headers:{
                'Content-Type':'application/json',
                'Authorization': localStorage.token
              },
              data:JSON.stringify(this.ruleForm),
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.$router.push('/collectionPersonManagement');
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
      selectChange(row){
        // console.log(this.ruleForm.productId);
        console.log(typeof (this.ruleForm.productId));

      },
      /**
       * 获取群组，角色，产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 催收员姓名或者手机号
       */
      getProductList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/collection/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{}
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.operationRoleList=res.data.body.operationRoleList;
            this.operationGroupList=res.data.body.operationGroupList;
            this.operationMerchantProductList1=res.data.body.operationMerchantProductList;
            for(var i=0;i<this.operationMerchantProductList1.length;i++){
              this.operationMerchantProductList.push({
                productId:this.operationMerchantProductList1[i].productId,
                productName:this.operationMerchantProductList1[i].productName
              });
              this.$forceUpdate();
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //查询催收员信息
      getCollectionPersonById(){
        var _this=this;
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/collection/find",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id:this.id
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm=res.data.body.accountManager;
            this.ruleForm.productIds=[];
            this.ruleForm.groupId=res.data.body.accountManager.operationGroup.id;
            this.ruleForm.roleId=res.data.body.accountManager.roles[0].id;
            let products = res.data.body.accountManager.products;
            products.forEach(function (item,index) {
              _this.ruleForm.productIds.push(item.id);
            });
            console.log(this.ruleForm.groupRole);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //添加群组
      addGroup(){
        this.centerDialogVisible1=true;
      },
      //添加群组-提交
      addGroupSubmit(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('groupName', this.ruleForm2.groupName);
            param.append('remark', this.ruleForm2.remark);
            param.append('enabled', this.ruleForm2.enabled);
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
                this.centerDialogVisible1=false;
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.getProductList();
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
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getProductList();
      this.getCollectionPersonById();
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
