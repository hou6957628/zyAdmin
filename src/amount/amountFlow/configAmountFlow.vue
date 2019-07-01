<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/ruleSetConfiguration' }">额度流程管理</el-breadcrumb-item>
      <el-breadcrumb-item>配置额度流程</el-breadcrumb-item>
    </el-breadcrumb>
    <h3 class="mt_20">额度流程</h3>
    <el-row :gutter="20">
      <el-col :span="6">创建时间:<div class="grid-content">{{ruleForm.createDate}}</div></el-col>
      <el-col :span="6">更新时间:<div class="grid-content">{{ruleForm.updateDate}}</div></el-col>
      <el-col :span="6">创建人员:<div class="grid-content">{{ruleForm.creator}}</div></el-col>
    </el-row>
    <div class="line"></div>
    <el-row :gutter="20">
      <h4 style="padding-left: 10px;padding-top: 20px;padding-bottom: 10px">说明</h4>
      <el-col>使用此额度流程的APP: <div class="grid-content">{{this.borrowingProductUsed==null?'无':this.borrowingProductUsed}}</div><p style="float: right;color:red;margin-right: 50px">修改记录</p></el-col>
    </el-row>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" >
      <div class="operationContent">
        <p class="topText">流程管理&nbsp;&nbsp;&nbsp;<el-button type="primary"  @click="addDomain" size="medium">添加子项</el-button></p>
        <div class="labelContent">
          <el-form-item>
            <el-form-item class="labelList" v-for="(domain, index) in electDataList.domains" :key="index">
              <span>{{domain.flowItemAlias}}&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;</span>
              标签名称：<el-button @click="xuan(index)" :id="['btn'+index]" v-if="domain.id">{{domain.tagName}}</el-button>
                        <el-button @click="xuan(index)" :id="['btn'+index]" v-if="!domain.id">点击选择</el-button>
              <!--规则-->
              <div v-if="btype==1" class="noDisplay">
                <el-select v-model="domain.action" class="select">
                  <el-option
                    v-for="item in actionList"
                    :key="item.key"
                    :label="item.Id"
                    :value="item.key">
                  </el-option>
                </el-select>
                <el-input type="text" style="width: 200px" v-model="domain.actionValue" @keyup.native="numberCheck1(index)"></el-input>
                <span style="margin-left: 15px">用户可选额度: </span>
                <el-input type="text" style="width: 200px" v-model="domain.amountApp"></el-input>
              </div>
              <el-button @click.prevent="removeDomain(domain)">删除</el-button>
            </el-form-item>
          </el-form-item >
          <p class="topText" style="margin: 60px 0 20px 0;">运算规则设置&nbsp;&nbsp;&nbsp;<el-tag class="fs15">&&:且</el-tag>&nbsp;&nbsp;<el-tag class="fs15">|:或</el-tag>&nbsp;&nbsp;<el-tag class="fs15">{}:最后执行</el-tag>&nbsp;&nbsp;<el-tag class="fs15">():优先执行</el-tag></p>
          <el-form-item class="" prop="flowRule">
            <el-input style="width: 500px" v-model="ruleForm.flowRule"></el-input>
            <el-button type="primary" @click="checkForDoc()">执行<i class="el-icon-upload el-icon--right"></i></el-button>
          </el-form-item>
          <p class="topText" style="margin-top: 60px">执行后标签规则说明</p>
          <div class="">
              <p style="width: 1000px;margin-bottom: 40px;background-color: #fff;font-size: 18px;padding: 20px" v-model="ruleForm.flowRuleDesc">{{ruleForm.flowRuleDesc}}</p>
            <el-button type="primary" @click="submitForm('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
            <el-button type="info" @click="resetForm('ruleForm')">取消<i class="el-icon-close el-icon--right"></i></el-button>
          </div>
        </div>
      </div>
    </el-form>
    <div v-if="bg" id="bg">
      <div v-for="(item,index) in tableData" style="display: inline-block;padding: 2px 5px;" :key="index">
        <el-radio class="radioType" v-model="radio" @change="changeHandler(item)" border size="medium" :label="item.id">{{item.tagName}}</el-radio>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    methods: {
      //添加数据
      addDomain() {
        if (this.electDataList.domains.length==0) {
          this.electDataList.domains.push({
            flowItemAlias: this.labelData[localStorage.num],
          });
          localStorage.num++;
        } else {
          this.electDataList.domains.push({
            flowItemAlias: this.labelData[localStorage.num],
          });
          localStorage.num++;
        }
      },
      //删除
      removeDomain(item) {
        var index = this.electDataList.domains.indexOf(item)
        if (index !== -1) {
          this.electDataList.domains.splice(index, 1)
        }
      },
      //选择字段
      xuan(index){
        this.count=index;
        this.bg=true;
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/quota/admin/amountTag/tagList",
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
      //选择字段的事件
      changeHandler(item) {
        this.bg=false;
        this.radio="";
        this.btype=1;
        this.electDataList.domains[this.count].tagId = item.id;
        $("#btn"+this.count).html(item.tagName);
        this.electDataList.domains[this.count].tagName = item.tagName;
      },
      //根据id查询流程
      getFlowById(ruleId){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/quota/admin/amountFlow/id",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: ruleId,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm = res.data.body.amountFlow;
            if (res.data.body.amountFlowItems.length != 0) {
              this.btype=1;
              this.electDataList.domains=res.data.body.amountFlowItems;
            }
            localStorage.num=this.electDataList.domains.length;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //执行：生成文本描述
      checkForDoc(){
        var data  = {
          'amountFlow':{
            'flowRule': this.ruleForm.flowRule,
          },
          'amountFlowItems':this.electDataList.domains
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/quota/admin/amountFlow/checkForDoc",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(data),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm.flowRuleDesc=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //保存标签
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            console.log(this.electDataList.domains);
            if (this.electDataList.domains[0].action==null) {
              this.$alert('请填写子项', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              var data  = {
                'amountFlow':this.ruleForm,
                'amountFlowItems':this.electDataList.domains
              };
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/quota/admin/amountFlow/config",
                headers:{
                  'Content-Type':'application/json',
                  'Authorization': localStorage.token
                },
                data:JSON.stringify(data),
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.$message({
                    message: '配置成功',
                    type: 'success'
                  });
                  this.$router.push('/amountFlowList');
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
        })
      },
      //只能输入数字
      numberCheck1(index) {
        var data = this.electDataList.domains[index].actionValue;
        let boo = new RegExp("^[0-9][0-9]*$").test(data);
        if (!boo) {
          this.$message.error("请输入正整数");
          this.electDataList.domains[index].actionValue='';
        }
      },
    },
    mounted:function () {
      localStorage.num=0;
      this.id=this.$route.params.id;
      this.getFlowById(this.id);
    },
    data() {
      return {
        electData: [ ],
        ruleForm:{
          id:'',
          flowRule:'',
          flowRuleDesc:'',
        },
        rules: {
          flowRule: [
            { required: true, message: '请输入运算规则', trigger: 'change' }
          ]
        },
        tableData:[],
        bg:false,
        num:'',
        index:'',
        count:0,
        btype:null,
        labelData:["A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z"
          ,"A1","A2","A3","A4","A5","A6","A7","A8","A9","B1","B2","B3","B4","B5","B6","B7","B8","B9",
          ,"C1","C2","C3","C4","C5","C6","C7","C8","C9","D1","D2","D3","D4","D5","D6","D7","D8","D9",
          ,"E1","E2","E3","E4","E5","E6","E7","E8","E9","F1","F2","F3","F4","F5","F6","F7","F8","F9",
          ,"G1","G2","G3","G4","G5","G6","G7","G8","G9","H1","H2","H3","H4","H5","H6","H7","H8","H9",
        ],
        electDataList: {
          domains: [{
            flowItemAlias: "A",tagId:'',tagName:'',action:null,actionValue:'',amountApp:'',id:'',
          }]
        },
        radio:"",
        actionList:[
          {key:0,Id:"增加额度"},
          {key:1,Id:"减少额度"},
          {key:2,Id:"维持原额度"},
          {key:3,Id:"拉黑"},
        ],
      }
    }
  }
</script>

<style scoped>
  .mt_20{
    margin-bottom: 20px;
  }
  .grid-content{
    display: inline-block;
    padding: 10px 0;
    padding-left: 10px;
  }
  .fs15{
    font-size: 15px;
  }
  .noDisplay{
    display: inline-block;
    height: 1px;
    margin-left: 15px;
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
    bottom: 500px;
    z-index: 1001;
  }
  .labelList{
    padding: 5px 0 10px 0;
    border-bottom: 2px solid #ccc;
    margin-bottom: 10px;
  }
  .select{
    margin-left: 5px;
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
    width: 98%;
    height: 1px;
    background-color: #999;
    margin-top: 10px;
  }
  .labelContent{
    padding: 20px;
    border: 1px solid #999;
  }
</style>
