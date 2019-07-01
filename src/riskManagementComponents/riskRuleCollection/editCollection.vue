<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/ruleSetConfiguration' }">风控规则集管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑规则集</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent" style="width: 500px">
      <p class="topText">规则集填写</p>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="规则集名称:" prop="collectionName">
          <el-input v-model="ruleForm.collectionName"></el-input>
        </el-form-item>
        <el-form-item label="规则集说明:">
          <el-input v-model="ruleForm.collectionDetail"></el-input>
        </el-form-item>
        <el-form-item label="规则集分类:" prop="classifyId">
          <el-select v-model="ruleForm.classifyId" placeholder="请选择" @change="changeClassify($event)">
            <el-option
              v-for="item in electData"
              :key="item.id"
              :label="item.classifyName"
              :value="item.id">
            </el-option>
          </el-select>&nbsp;&nbsp;
          <el-button @click="addTag()">添加规则集分类</el-button>
        </el-form-item>
        <el-form-item label="是否启用:" prop="enabled">
          <el-select v-model="ruleForm.enabled" disabled placeholder="请选择">
            <el-option
              v-for="item in electDataEnabled"
              :key="item.key"
              :label="item.Id"
              :value="item.key">
            </el-option>
          </el-select>
        </el-form-item>

    <el-row :gutter="20" style="width: 1200px">
      <el-col :span="6">创建时间:<div class="grid-content">{{ruleForm.createDate}}</div></el-col>
      <el-col :span="6">更新时间:<div class="grid-content">{{ruleForm.updateDate}}</div></el-col>
      <el-col :span="6">创建人员:<div class="grid-content">{{ruleForm.creator}}</div></el-col>
    </el-row>
    <div class="line"></div>
    <el-row :gutter="20">
      <h4 style="padding-left: 10px;padding-top: 20px;padding-bottom: 10px">使用中的风控流程</h4>
      <el-col  style="width: 1200px">对应使用的风控流程: <div class="grid-content">风控流程1、风控流程2、风控流程3、风控流程四</div><p style="float: right;color:red;margin-right: 50px">修改记录</p></el-col>
    </el-row>
    <div class="operationContent">
      <p class="topText">规则设置&nbsp;&nbsp;&nbsp;<el-button type="primary"  @click="addDomain" size="medium">添加规则</el-button></p>
      <div class="labelContent" style="width: 1200px">
        <el-form-item label-width="10px">
          <el-form-item class="labelList" v-for="(domain, index) in electDataList.domains" :key="index">
            <span>{{domain.itemAlias}}&nbsp;&nbsp;&nbsp;&nbsp;|</span>
            规则：
            <el-button @click="xuan(index)" :id="['btn'+index]" v-if="domain.ruleId">{{domain.ruleName}}</el-button>
            <el-button @click="xuan(index)" :id="['btn'+index]" v-if="!domain.ruleId">点击选择</el-button>
            <el-button @click.prevent="removeDomain(domain)">删除</el-button>
            <el-button @click.prevent="toDetailRule(domain)" type="primary" plain>查看规则详情</el-button>
          </el-form-item>
        </el-form-item >
        <p class="topText" style="margin: 60px 0 20px 0;">运算规则集设置&nbsp;&nbsp;&nbsp;<el-tag class="fs15">&&:且</el-tag>&nbsp;&nbsp;<el-tag class="fs15">|:或</el-tag>&nbsp;&nbsp;<el-tag class="fs15">{}:最后执行</el-tag>&nbsp;&nbsp;<el-tag class="fs15">():优先执行</el-tag></p>
        <el-form-item class="" prop="collectionRule" label-width="5px">
          <el-input style="width: 500px" v-model="ruleForm.collectionRule"></el-input>
          <el-button type="primary" @click="checkForDoc()">执行<i class="el-icon-upload el-icon--right"></i></el-button>
        </el-form-item>
        <p class="topText" style="margin-top: 60px">执行后规则说明</p>
        <div class="">
          <p style="width: 1000px;margin-bottom: 40px;background-color: #fff;font-size: 18px;padding: 20px" v-model="ruleForm.collectionRuleDesc">{{ruleForm.collectionRuleDesc}}</p>
          <el-button type="primary" @click="submitFormTips('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info" @click="resetForm('ruleForm')">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </div>
      </div>
    </div>
    </el-form>
    </div>
    <div v-if="bg" id="bg">
      <div v-for="(item,index) in tableData" style="display: inline-block;padding: 2px 5px;" :key="index">
        <el-radio class="radioType" v-model="radio" @change="changeHandler(item.ruleName,item.id)" border size="medium" :label="item.id">{{item.ruleName}}</el-radio>
      </div>
    </div>
    <!--添加规则及分类结构-->
    <el-dialog
      title="添加用户标签分类"
      :visible.sync="centerDialogVisible1"
      width="40%"
      center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="100px" class="demo-ruleForm">
        <el-form-item label="分类名称:" prop="classifyName" :rules="[ruleForm2.classifyName,{validator:checkName2, required: true, trigger:'blur'}]">
          <el-input v-model="ruleForm2.classifyName"></el-input>
        </el-form-item>
        <el-form-item label="分类说明:">
          <el-input v-model="ruleForm2.classifyDescription"></el-input>
        </el-form-item>
        <el-form-item label="是否启用:" prop="classifyEnabled">
          <el-select v-model="ruleForm2.classifyEnabled" placeholder="请选择" @change="selectChange()">
            <el-option
              v-for="item in electDataEnabled"
              :key="item.key"
              :label="item.Id"
              :value="item.key">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="addClassification('ruleForm2')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible1 = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>

</template>

<script>
  import axios from 'axios'
  export default {
    methods: {
      //验证规则集分类名称是否重名
      checkName2(rule, value, callback) {
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
              id:2
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
      //根据id查询规则集
      getRuleCollectionById(ruleId){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/risk/admin/rule_collection/id",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: ruleId,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm = res.data.body.rcRuleCollection;
            this.electDataList.domains=res.data.body.rcRuleCollectionItem;
            localStorage.num=this.electDataList.domains.length-1;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //提示保存规则集弹窗
      submitFormTips(formName){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/rule_collection/checkRef",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: this.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body.count != 0) {
              this.$confirm('修改规则集，对应风控流程也会发生改变，是否确定修改？', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                center: true,
                type: 'warning'
              }).then(() => {
                this.submitForm(formName);
              }).catch(() => {
                this.$message({
                  type: 'info',
                  message: '已取消修改'
                });
              });
            } else {
              this.$confirm('是否确定保存?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.submitForm(formName);
              }).catch(() => {
                this.$message({
                  type: 'info',
                  message: '已取消修改'
                });
              });
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //编辑保存规则集
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            if (this.electDataList.domains[0]==null) {
              this.$alert('请填写子项', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              var data  = {
                'rcRuleCollection':this.ruleForm,
                'rcRuleCollectionItem':this.electDataList.domains
              };
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/risk/admin/rule_collection/update",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                data:JSON.stringify(data),
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.$message({
                    message: '修改成功',
                    type: 'success'
                  });
                  this.$router.push('/collectionList');
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            }
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //执行：生成文本描述
      checkForDoc(){
        var data  = {
          'rcRuleCollection':{
            'collectionRule': this.ruleForm.collectionRule,
          },
          'rcRuleCollectionItem':this.electDataList.domains
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/risk/admin/rule_collection/checkForDoc",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(data),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm.collectionRuleDesc=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //下拉选择
      changeClassify(vId){
        let obj = {};
        obj = this.electData.find((item)=>{
          return item.id === vId;
        });
        this.ruleForm.classifyName=obj.classifyName;
      },
      //添加数据
      addDomain() {
        localStorage.num++;
        this.electDataList.domains.push({
          itemAlias: this.labelData[localStorage.num],
          ruleId: null,
          ruleName: null,
        });
      },
      //删除
      removeDomain(item) {
        var index = this.electDataList.domains.indexOf(item)
        if (index !== -1) {
          this.electDataList.domains.splice(index, 1)
        }
      },
      //查看规则详情
      toDetailRule(item) {
        var id=item.ruleId;
        this.$router.push({
          path: `/editRule/${id}`,
        });
      },
      //点击选择所有规则
      xuan(index){
        this.count=index;
        this.bg=true;
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/risk/admin/rule/ruleList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //选择规则的事件
      changeHandler(key,value) {
        this.bg=false;
        this.radio="";
        $("#btn"+this.count).html(key);
        this.electDataList.domains[this.count].ruleId = value;
        this.electDataList.domains[this.count].ruleName = key;
      },
      //查询规则集分类列表
      getRuleClassify(ruleTypeId){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/risk/admin/classification/getByType",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            type: ruleTypeId,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.electData=res.data.body.list;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //添加规则集分类
      addTag(){
        this.centerDialogVisible1=true;
      },
      //添加规则集分类提交
      addClassification(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method:"post",
              url:"http://"+this.baseUrl+"/risk/admin/classification/add",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params:{
                classifyName: this.ruleForm2.classifyName,
                description: this.ruleForm2.classifyDescription,
                enabled: this.ruleForm2.classifyEnabled,
                classifyType: 2,
              }
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.centerDialogVisible1 = false;
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.getRuleClassify(2);
              }else {
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
      }
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getRuleCollectionById(this.id);
      this.getRuleClassify(2);
    },
    data() {
      return {
        centerDialogVisible1:false,
        electData: [ ],
        electDataEnabled: [
          {key:true,Id:'启用'},
          {key:false,Id:'不启用'},
        ],
        ruleForm:{
          id:'',
          collectionName:'',
          collectionDetail:'',
          classifyId:'',
          classifyName:'',
          enabled:'',
          collectionRule:'',
          collectionRuleDesc:"",
          createDate:'',
          updateDate:'',
          creator:'',
        },
        rules: {
          collectionName: [
            { required: true, message: '请输入规则集名称', trigger: 'blur' },
          ],
          classifyId: [
            { required: true, message: '请选择规则集分类', trigger: 'change' }
          ],
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ],
          collectionRule: [
            { required: true, message: '请输入运算规则', trigger: 'blur' }
          ]
        },
        ruleForm2:{
          classifyName:'',
          classifyDescription:'',
          classifyEnabled:true,
          classifyType:'',
        },
        rules2: {
          classifyEnabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ],
        },
        tableData:[],
        electValue:'',
        finProduct: '',
        bg:false,
        num:'',
        type:'',
        index:'',
        count:0,
        labelData:["A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z"
          ,"A1","A2","A3","A4","A5","A6","A7","A8","A9","B1","B2","B3","B4","B5","B6","B7","B8","B9",
          ,"C1","C2","C3","C4","C5","C6","C7","C8","C9","D1","D2","D3","D4","D5","D6","D7","D8","D9",
          ,"E1","E2","E3","E4","E5","E6","E7","E8","E9","F1","F2","F3","F4","F5","F6","F7","F8","F9",
          ,"G1","G2","G3","G4","G5","G6","G7","G8","G9","H1","H2","H3","H4","H5","H6","H7","H8","H9",
        ],
        electDataList: {
          domains: [{
            itemAlias: "A",ruleId:'',
          }]
        },
        radio:"",
      }
    }
  }
</script>

<style scoped>
  .grid-content{
    display: inline-block;
    padding: 10px 0;
    padding-left: 10px;
  }
  .fs15{
    font-size: 15px;
  }
  .noDisplay{
    display: none;
  }
  .radioType{
    margin: 5px;
  }
  #bg{
    background-color: #fff;
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    height: 100%;
    z-index: 1001;
    overflow-y: scroll;
  }
  .labelList{
    padding: 5px 0 10px 0;
    border-bottom: 2px solid #ccc;
    margin-bottom: 10px;
  }
  .select{
    margin-left: 20px;
  }
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }
  .fs-16{
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .operationContent{
    text-align: left;
    margin: 10px 30px 15px 0;
  }
  .operationContent .upLoadBtn{

  }
  .operationContent .searchContent{
    margin-left: 20px;
    width: 300px;
    margin-right: 30px;
  }
  .topText{
    font-size: 16px;
    font-weight: bold;
    height: 60px;
    line-height: 60px;
  }
  .line{
    width: 1240px;
    height: 1px;
    background-color: #999;
  }
  .labelContent{
    padding: 20px;
    border: 1px solid #999;
  }
</style>
