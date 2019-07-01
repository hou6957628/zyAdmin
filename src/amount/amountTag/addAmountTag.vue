<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/userTagsList' }">标签管理</el-breadcrumb-item>
      <el-breadcrumb-item>创建标签</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <p class="topText">额度标签</p>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="标签名称:" prop="tagName" :rules="[ruleForm.tagName,{validator:checkName, required: true, trigger:'blur'}]">
          <el-input v-model="ruleForm.tagName" style="width: 500px"></el-input>
        </el-form-item>
        <el-form-item label="标签说明:">
          <el-input v-model="ruleForm.description" style="width: 500px"></el-input>
        </el-form-item>
        <el-form-item label="是否启用:" prop="enabled">
          <el-select v-model="ruleForm.enabled" placeholder="请选择" @change="selectChange">
            <el-option
              v-for="item in electDataEnabled"
              :key="item.key"
              :label="item.Id"
              :value="item.key">
            </el-option>
          </el-select>
        </el-form-item>
    <div class="line"></div>
    <div class="operationContent">
      <p class="topText">字段设置&nbsp;&nbsp;&nbsp;<el-button type="primary"  @click="addDomain" size="medium">添加字段</el-button></p>
      <div class="labelContent">
        <el-form-item label-width="10px">
          <el-form-item class="labelList" v-for="(domain, index) in electDataList.domains" :key="index">
            <span>{{domain.tagItemAlias}}&nbsp;&nbsp;&nbsp;&nbsp;|</span>
            字段名称：<el-button @click="xuan(index)" :id="['btn'+index]">点击选择</el-button>
            <!--数字类型type=0：两个输入框-->
            <div class="noDisplay" :class="['type'+index]">
              <el-select v-model="domain.symbolCode1" @change="selectChange">
                <el-option
                  v-for="item in fieldCodeList0"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-input type="text" style="width: 200px" v-model="domain.fieldValue1" @keyup.native="numberCheck1(index)"></el-input>
              <el-select v-model="domain.symbolCode2" @change="selectChange">
                <el-option
                  v-for="item in fieldCodeList0"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-input type="text" style="width: 200px" v-model="domain.fieldValue2" @keyup.native="numberCheck2(index)"></el-input>
              <el-input v-if="domain.symbolCode2" type="hidden" style="width: 1px;" v-model="domain.operationalSymbolCode=domain.symbolCode1+','+domain.symbolCode2"></el-input>
              <el-input v-if="!domain.symbolCode2" type="hidden" style="width: 1px;" v-model="domain.operationalSymbolCode=domain.symbolCode1"></el-input>
              <el-input v-if="domain.fieldValue2" type="hidden" style="width: 1px;" v-model="domain.fieldValue=domain.fieldValue1+','+domain.fieldValue2"></el-input>
              <el-input v-if="!domain.fieldValue2" type="hidden" style="width: 1px;" v-model="domain.fieldValue=domain.fieldValue1"></el-input>
            </div>
            <el-button @click.prevent="removeDomain(domain)">删除</el-button>
          </el-form-item>
        </el-form-item >
        <p class="topText" style="margin: 60px 0 20px 0;">标签规则设置&nbsp;&nbsp;&nbsp;<el-tag class="fs15">&&:且</el-tag>&nbsp;&nbsp;<el-tag class="fs15">|:或</el-tag>&nbsp;&nbsp;<el-tag class="fs15">{}:最后执行</el-tag>&nbsp;&nbsp;<el-tag class="fs15">():优先执行</el-tag></p>
        <el-form-item class="" prop="tagRule" label-width="5px">
          <el-input style="width: 500px" v-model="ruleForm.tagRule"></el-input>
          <el-button type="primary" @click="checkForDoc()">执行<i class="el-icon-upload el-icon--right"></i></el-button>
        </el-form-item>
        <p class="topText" style="margin-top: 60px">执行后标签规则说明</p>
        <div class="">
          <p style="width: 1000px;margin-bottom: 40px;background-color: #fff;font-size: 18px;padding: 20px" v-model="ruleForm.tagRuleDesc">{{ruleForm.tagRuleDesc}}</p>
          <el-button type="primary" @click="submitForm('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info" @click="resetForm('ruleForm')">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </div>
      </div>
    </div>
    </el-form>
    </div>
    <div v-if="bg" id="bg">
      <div v-for="(item,index) in tableData" style="display: inline-block;padding: 2px 5px;">
        <el-radio class="radioType" v-model="radio" @change="changeHandler(item)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
    </div>
  </div>

</template>

<script>
  import axios from 'axios'
  export default {
    methods: {
      //验证标签名称是否重名
      checkName(rule, value, callback) {
        if (!value) {
          callback(new Error('请输入名称'));
        } else {
          axios({
            method:"GET",
            url:"http://"+this.baseUrl+"/quota/admin/amountTag/getByName",
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
      //保存标签
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            if (this.electDataList.domains.length==0) {
              this.$alert('请填写子项', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              if (this.electDataList.domains[0].fieldValue.length==0 | this.electDataList.domains[0].operationalSymbolCode == null) {
                this.$alert('请填写子项', '提示', {
                  confirmButtonText: '确定',
                  center: true,
                  type: 'warning'
                });
              } else {
                var data  = {
                  'amountTag':this.ruleForm,
                  'amountTagItems':this.electDataList.domains
                };
                axios({
                  method:"POST",
                  url:"http://"+this.baseUrl+"/quota/admin/amountTag/add",
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
                    this.$router.push('/amountTagList');
                  }else if(res.data.msgCd=='ZYCASH-1009'){
                    this.$message.error(res.data.msgInfo);
                  }
                  else {
                    this.$message.error(res);
                  }
                })
              }
            }
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //强制刷新
      selectChange(){
        this.$forceUpdate();
      },
      //添加数据
      addDomain() {
        localStorage.num++;
        this.electDataList.domains.push({
          tagItemAlias: this.labelData[localStorage.num],
          fieldCode: null,
          fieldName: null,
          fieldType: null,
        });
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
          url:"http://"+this.baseUrl+"/quota/admin/amountField/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body.fieldList;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //选择字段的事件
      changeHandler(item) {
        this.bg=false;
        this.radio="";
        this.electDataList.domains[this.count].operationalSymbolCode='';
        this.electDataList.domains[this.count].operationalSymbolName='';
        this.electDataList.domains[this.count].symbolCode1='';
        this.electDataList.domains[this.count].symbolCode2='';
        this.electDataList.domains[this.count].fieldValue='';
        this.electDataList.domains[this.count].fieldValue1='';
        this.electDataList.domains[this.count].fieldValue2='';
        $("#btn"+this.count).html(item.fieldName);
        this.electDataList.domains[this.count].fieldCode=item.fieldCode;
        this.electDataList.domains[this.count].fieldName=item.fieldName;
        $(".noDisplay").css("display","inline-block");
      },
      //执行：生成文本描述
      checkForDoc(){
        var data  = {
          'amountTag':{
            'tagRule': this.ruleForm.tagRule,
          },
          'amountTagItems':this.electDataList.domains
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/quota/admin/amountTag/checkForDoc",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(data),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm.tagRuleDesc=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //封装label值
      changeSelect2(vId,index,list){
        this.$forceUpdate();
        let obj = {};
        obj = list.find((item)=>{
          return item.key === vId;
        });
        this.electDataList.domains[index].operationalSymbolName=obj.Id;
      },
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      //只能输入数字
      numberCheck1(index) {
        var data = this.electDataList.domains[index].fieldValue1;
        let boo = new RegExp("^[0-9][0-9]*$").test(data);
        if (!boo) {
          this.$message.error("请输入正整数");
          this.electDataList.domains[index].fieldValue1='';
        }
      },
      numberCheck2(index) {
        var data = this.electDataList.domains[index].fieldValue2;
        let boo = new RegExp("^[0-9][0-9]*$").test(data);
        if (!boo) {
          this.$message.error("请输入正整数");
          this.electDataList.domains[index].fieldValue2='';
        }
      }
    },
    mounted:function () {
      localStorage.num=0;
    },
    data() {
      return {
        electDataEnabled: [
          {key:1,Id:'启用'},
          {key:0,Id:'不启用'},
        ],
        ruleForm:{
          tagName:'',
          description:'',
          enabled:1,
          tagRule:'',
          tagRuleDesc:"",
        },
        rules: {
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ],
          tagRule: [
            { required: true, message: '请输入标签规则', trigger: 'blur' }
          ]
        },
        electValue:'',
        tableData:[],
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
            tagItemAlias: "A",fieldValue:'',fieldCode:'',fieldName:'',
            operationalSymbolCode: null,operationalSymbolName: '',symbolCode1: '',fieldValue1:'',symbolCode2: '',fieldValue2:'',
          }]
        },
        radio:"",
        fieldCodeList0:[
          {key:"EQ",Id:"等于"},
          {key:"NEQ",Id:"不等于"},
          {key:"LT",Id:"小于"},
          {key:"GT",Id:"大于"},
          {key:"ELT",Id:"小于等于"},
          {key:"EGT",Id:"大于等于"},
        ],
      }
    }
  }
</script>

<style scoped>
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
    bottom: 500px;
    z-index: 1001;
  }
  .labelList{
    padding: 5px 0 10px 0;
    border-bottom: 2px solid #ccc;
    margin-bottom: 10px;
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
  }
  .labelContent{
    padding: 20px;
    border: 1px solid #999;
  }
</style>
