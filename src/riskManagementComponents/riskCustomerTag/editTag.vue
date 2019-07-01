<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/tagList' }">用户标签管理</el-breadcrumb-item>
      <el-breadcrumb-item>创建标签</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent" style="width: 500px">
      <p class="topText">用户标签</p>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="标签名称:" prop="tagName">
          <el-input v-model="ruleForm.tagName">{{ruleForm.tagName}}</el-input>
        </el-form-item>
        <el-form-item label="标签说明:">
          <el-input v-model="ruleForm.description"></el-input>
        </el-form-item>
        <el-form-item label="标签分类:" prop="tagTypeId">
          <el-select v-model="ruleForm.tagTypeId" placeholder="请选择" @change="changeClassify($event)">
            <el-option
              v-for="item in electData"
              :key="item.id"
              :label="item.classifyName"
              :value="item.id">
            </el-option>
          </el-select>&nbsp;&nbsp;
          <el-button @click="addTag()">添加标签</el-button>
        </el-form-item>
        <el-form-item label="是否启用:" prop="enabled">
          <el-select v-model="ruleForm.enabled" disabled placeholder="请选择" @change="selectChange">
            <el-option
              v-for="item in electDataEnabled"
              :key="item.key"
              :label="item.Id"
              :value="item.key">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
    </div>
    <div class="line"></div>
    <el-form>
    <div class="operationContent">
      <p class="topText">标签设置&nbsp;&nbsp;&nbsp;<el-button type="primary"  @click="addDomain" size="medium">添加标签</el-button></p>
      <div class="labelContent">
        <el-form-item>
          <el-form-item class="labelList" v-for="(domain, index) in electDataList.domains" :key="index">
            <span>{{domain.tagItemAlias}}&nbsp;&nbsp;&nbsp;&nbsp;|</span>
            字段名称：<el-button @click="xuan(index)" :id="['btn'+index]" v-if="domain.fieldName">{{domain.fieldName}}</el-button>
                      <el-button @click="xuan(index)" :id="['btn'+index]" v-if="!domain.fieldName">点击选择</el-button>
            <!--数字类型type=0：两个输入框-->
            <div v-if="domain.fieldDataType-1==0" style="display: inline-block;height: 1px">
              <el-select v-model="domain.symbolCode1" @change="changeSelect()">
                <el-option
                  v-for="item in fieldCodeList0"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-input type="text" style="width: 200px" v-model="domain.fieldValue1" @keyup.native="numberCheck1(index)"></el-input>
              <el-select v-model="domain.symbolCode2" @change="changeSelect()">
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
            <!--boolean类型type=1-->
            <div v-if="domain.fieldDataType-1==1" style="display: inline-block;height: 1px">
              <el-select v-model="domain.symbolCode" @change="changeSelect2($event,index,fieldCodeList1)">
                <el-option
                  v-for="item in fieldCodeList1"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-select v-model="domain.fieldValue">
                <el-option
                  v-for="item in symbolCodeList"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
            </div>
            <!--单选类型type=2-->
            <div v-if="domain.fieldDataType-1==2" style="display: inline-block;height: 1px">
              <el-select v-model="domain.operationalSymbolCode" @change="changeSelect2($event,index,fieldCodeList2)">
                <el-option
                  v-for="item in fieldCodeList2"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-select v-model="domain.fieldValue" @focus="selectChange(domain.fieldCode,index)">
                <el-option
                  v-for="item in domain.selectValues"
                  :key="item.value"
                  :label="item.description"
                  :value="item.value">
                </el-option>
              </el-select>
            </div>
            <!--多选类型type=3-->
            <div v-if="domain.fieldDataType-1==3" style="display: inline-block;height: 1px">
              <el-select v-model="domain.operationalSymbolCode" @change="changeSelect2($event,index,fieldCodeList3)">
                <el-option
                  v-for="item in fieldCodeList3"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-select v-model="domain.fieldValue" multiple style="width: 500px" @focus="selectChange(domain.fieldCode,index)">
                <el-option
                  v-for="item in domain.selectValues"
                  :key="item.value"
                  :label="item.description"
                  :value="item.value">
                </el-option>
              </el-select>
            </div>
            <!--时间类型type=4：两个输入框-->
            <div v-if="domain.fieldDataType-1==4" style="display: inline-block;height: 1px">
              <el-select v-model="domain.symbolCode1" @change="changeSelect()">
                <el-option
                  v-for="item in fieldCodeList0"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-date-picker type="date" style="width: 200px" v-model="domain.fieldValue1" @blur="changeSelect()"
                              placeholder="选择日期" value-format="yyyy-MM-dd" format="yyyy 年 MM 月 dd 日"></el-date-picker>
              <el-select v-model="domain.symbolCode2" @change="changeSelect()">
                <el-option
                  v-for="item in fieldCodeList0"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-date-picker type="date" style="width: 200px" v-model="domain.fieldValue2" @blur="changeSelect()"
                              placeholder="选择日期" value-format="yyyy-MM-dd" format="yyyy 年 MM 月 dd 日"></el-date-picker>
              <el-input v-if="domain.symbolCode2" type="hidden" style="width: 1px;" v-model="domain.operationalSymbolCode=domain.symbolCode1+','+domain.symbolCode2"></el-input>
              <el-input v-if="!domain.symbolCode2" type="hidden" style="width: 1px;" v-model="domain.operationalSymbolCode=domain.symbolCode1"></el-input>
              <el-input v-if="domain.fieldValue2" type="hidden" style="width: 1px;" v-model="domain.fieldValue=domain.fieldValue1+','+domain.fieldValue2"></el-input>
              <el-input v-if="!domain.fieldValue2" type="hidden" style="width: 1px;" v-model="domain.fieldValue=domain.fieldValue1"></el-input>
            </div>
            <!--字符串类型type=5-->
            <div v-if="domain.fieldDataType-1==5" style="display: inline-block;height: 1px">
              <el-select v-model="domain.operationalSymbolCode" @change="changeSelect2($event,index,fieldCodeList5)">
                <el-option
                  v-for="item in fieldCodeList5"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-input type="text" style="width: 200px" v-model="domain.fieldValue"></el-input>
            </div>
            <el-button @click.prevent="removeDomain(domain)">删除</el-button>
          </el-form-item>
        </el-form-item >
        <p class="topText" style="margin: 60px 0 20px 0;">标签规则设置&nbsp;&nbsp;&nbsp;<el-tag class="fs15">&&:且</el-tag>&nbsp;&nbsp;<el-tag class="fs15">|:或</el-tag>&nbsp;&nbsp;<el-tag class="fs15">{}:最后执行</el-tag>&nbsp;&nbsp;<el-tag class="fs15">():优先执行</el-tag></p>
        <div class="">
          <el-input style="width: 500px" v-model="ruleForm.tagRule"></el-input>
          <el-button type="primary" @click="checkForDoc()">执行<i class="el-icon-upload el-icon--right"></i></el-button>
        </div>
        <p class="topText" style="margin-top: 60px">执行后标签规则说明</p>
        <div class="">
          <p style="width: 1000px;margin-bottom: 40px;background-color: #fff;font-size: 18px;padding: 20px" v-model="ruleForm.tagRuleDesc">{{ruleForm.tagRuleDesc}}</p>
          <el-button type="primary" @click="submitFormTips('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info" @click="resetForm('ruleForm')">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </div>
      </div>
    </div>
    </el-form>
    <div v-if="bg" id="bg">
      <p style="text-align: center;margin: 5px 0">----基本字段----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='基本字段'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
      <p style="text-align: center;margin: 5px 0">----运营商字段----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='运营商字段'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
      <p style="text-align: center;margin: 5px 0">----白骑士字段----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='白骑士字段'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
      <p style="text-align: center;margin: 5px 0">----天机命中字段----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='天机命中字段'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
      <p style="text-align: center;margin: 5px 0">----天机多头报告----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='天机多头报告'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
      <p style="text-align: center;margin: 5px 0">----通库黑名单----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='通库黑名单'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
      <p style="text-align: center;margin: 5px 0">----通讯录字段----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='通讯录字段'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
      <p style="text-align: center;margin: 5px 0">----face++----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='face++'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
      <p style="text-align: center;margin: 5px 0">----新颜运营商字段----</p>
      <div v-for="(item,index) in tableData" v-if="item.fieldType=='新颜运营商字段'" style="display: inline-block;padding: 2px 3px;width: 17%;">
        <el-radio :dataType="item.id" class="radioType" v-model="radio" @change="changeHandler(item.fieldName,item.id)" border size="medium" :label="item.id">{{item.fieldName}}</el-radio>
      </div>
    </div>
    <!--添加标签结构-->
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
          <el-select v-model="ruleForm2.classifyEnabled" placeholder="请选择" @change="selectChange">
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
      //验证标签分类名称是否重名
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
              id:0
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
      //提示保存标签弹窗
      submitFormTips(formName){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/customer_tag/checkRef",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            customerTag: this.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body.list.length != 0) {
              this.$alert('修改用户标签，对应风控流程也会发生改变，是否确定修改？', '提示', {
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
                  'customerTag':this.ruleForm,
                  'customerTagItems':this.electDataList.domains
                };
                axios({
                  method:"POST",
                  url:"http://"+this.baseUrl+"/risk/admin/customer_tag/update",
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
                    this.$router.push('/tagList');
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
          url:"http://"+this.baseUrl+"/risk/admin/field/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body;
            for(var i=0;i<this.tableData.length;i++){
              console.log(this.tableData[i].fieldType);
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //选择字段的事件
      changeHandler(key,value) {
        this.bg=false;
        this.radio="";
        $("#btn"+this.count).html(key);
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/risk/admin/field/id",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: value
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.electDataList.domains[this.count].operationalSymbolCode='';
            this.electDataList.domains[this.count].operationalSymbolName='';
            this.electDataList.domains[this.count].symbolCode1='';
            this.electDataList.domains[this.count].symbolCode2='';
            // this.electDataList.domains[this.count].fieldValue='';
            this.electDataList.domains[this.count].fieldValue1='';
            this.electDataList.domains[this.count].fieldValue2='';
            var type=res.data.body.rcField.dataType - 1;
            this.electDataList.domains[this.count].fieldCode=res.data.body.rcField.fieldCode;
            this.electDataList.domains[this.count].fieldName=res.data.body.rcField.fieldName;
            this.electDataList.domains[this.count].fieldType=res.data.body.rcField.fieldType;
            this.electDataList.domains[this.count].fieldDataType=res.data.body.rcField.dataType;
            if (type == 2 || type == 3) {
              this.selectValues.push(res.data.body.selectValues);
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        });
      },
      //查询用户标签分类
      getCustomerTagType(customerTagType){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/risk/admin/classification/getByType",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            type: customerTagType,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.electData=res.data.body.list;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //执行：生成文本描述
      checkForDoc(){
        var data  = {
          'customerTag':{
            'tagRule': this.ruleForm.tagRule,
          },
          'customerTagItems':this.electDataList.domains
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/risk/admin/customer_tag/checkForDoc",
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
      //根据id查询用户标签
      getCustomerTag(customerTagId){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/risk/admin/customer_tag/id",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: customerTagId,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm = res.data.body.customerTag;
            this.ruleForm.tagName = res.data.body.customerTag.tagName;
            this.ruleForm.description = res.data.body.customerTag.description;
            this.ruleForm.tagTypeId = res.data.body.customerTag.tagTypeId;
            this.ruleForm.enabled = res.data.body.customerTag.enabled;
            this.ruleForm.tagRule = res.data.body.customerTag.tagRule;
            this.ruleForm.tagRuleDesc = res.data.body.customerTag.tagRuleDesc;
            this.electDataList.domains=res.data.body.customerTagItems;
            localStorage.num=this.electDataList.domains.length-1;
            var _this=this;
            this.electDataList.domains.forEach(function (item,index) {
              if (item.fieldCode=='nation') {
                let nationList=item.fieldValue.split(',');
                _this.electDataList.domains[index].fieldValue=nationList;
              }
              if (item.fieldDataType==1 || item.fieldDataType==5) {
                if (item.operationalSymbolCode.indexOf(',')!=-1) {
                  _this.electDataList.domains[index].symbolCode1=item.operationalSymbolCode.split(',')[0];
                  _this.electDataList.domains[index].symbolCode2=item.operationalSymbolCode.split(',')[1];
                  _this.electDataList.domains[index].fieldValue1=item.fieldValue.split(',')[0];
                  _this.electDataList.domains[index].fieldValue2=item.fieldValue.split(',')[1];
                } else {
                  _this.electDataList.domains[index].symbolCode1=item.operationalSymbolCode;
                  _this.electDataList.domains[index].fieldValue1=item.fieldValue;
                }
              }
            });
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //下拉选择
      selectChange(data,index){
        this.kai=!this.kai;
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/risk/admin/field/getFieldByCode",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            fieldCode: data
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.electDataList.domains[index].fieldCode=res.data.body.rcField.fieldCode;
            this.electDataList.domains[index].fieldName=res.data.body.rcField.fieldName;
            this.electDataList.domains[index].fieldType=res.data.body.rcField.fieldType;
            this.electDataList.domains[index].fieldDataType=res.data.body.rcField.dataType;
            this.electDataList.domains[index].selectValues=res.data.body.selectValues;
            this.$forceUpdate();
          }else {
            this.$message.error(res.data.msgInfo);
          }
        });
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
      //强制刷新视图层
      changeSelect(){
        this.$forceUpdate();
      },
      //过滤类型字段
      typeFormatter(row){

      },
      //添加标签
      addTag(){
        this.centerDialogVisible1=true;
      },
      //下拉选择
      changeClassify(vId){
        let obj = {};
        obj = this.electData.find((item)=>{
          return item.id === vId;
        });
        this.ruleForm.classifyName=obj.classifyName;
      },
      //添加标签提交
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
                classifyType: 0,
              }
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.centerDialogVisible1 = false;
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.getCustomerTagType(0);
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
      },
      //只能输入数字
      numberCheck1(index) {
        // data.target.value=data.target.value.replace(/[^\.\d]/g,'');
        // data.target.value=data.target.value.replace('.','');
        var data = this.electDataList.domains[index].fieldValue1;
        let boo = new RegExp("^[0-9][0-9]*$").test(data);
        if (!boo) {
          this.$message.error("请输入正整数");
          this.electDataList.domains[index].fieldValue1=data.replace(/[^\d]/g,'');
        }
      },
      numberCheck2(index) {
        // data.target.value=data.target.value.replace(/[^\.\d]/g,'');
        // data.target.value=data.target.value.replace('.','');
        var data = this.electDataList.domains[index].fieldValue2;
        let boo = new RegExp("^[0-9][0-9]*$").test(data);
        if (!boo) {
          this.$message.error("请输入正整数");
          this.electDataList.domains[index].fieldValue2=data.replace(/[^\d]/g,'');
        }
      }
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getCustomerTag(this.id);
      this.getCustomerTagType(0);
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
          tagName:'',
          description:'',
          tagTypeId:'',
          enabled:'',
          tagRule:'',
          tagRuleDesc:"",
        },
        rules: {
          tagName: [
            { required: true, message: '请输入标签名称', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
          ],
          tagTypeId: [
            { required: true, message: '请选择标签分类', trigger: 'change' }
          ],
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ],
          tagRule: [
            { required: true, message: '请输入标签规则', trigger: 'blur' }
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
        electValue:'',
        tableData:[],
        bg:false,
        num:'',
        kai:true,
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
            tagItemAlias: "A",fieldValue:'',fieldCode:'',fieldName:'',fieldType:'',fieldDataType:'',selectValues: [ ],
            operationalSymbolCode: '',operationalSymbolName: '',symbolCode1: '',fieldValue1:'',symbolCode2: '',fieldValue2:'',
          }]
        },
        radio:"",
        symbolCodeList:[
          {key:1,Id:"是"},
          {key:0,Id:"否"},
        ],
        fieldCodeList0:[
          {key:"EQ",Id:"等于"},
          {key:"NEQ",Id:"不等于"},
          {key:"LT",Id:"小于"},
          {key:"GT",Id:"大于"},
          {key:"ELT",Id:"小于等于"},
          {key:"EGT",Id:"大于等于"},
        ],
        fieldCodeList1:[
          {key:"EQ",Id:"等于"},
        ],
        fieldCodeList2:[
          {key:"EQ",Id:"等于"},
          {key:"NEQ",Id:"不等于"},
        ],
        fieldCodeList3:[
          {key:"IN",Id:"包含"},
          {key:"NIN",Id:"不包含"},
        ],
        fieldCodeList5:[
          {key:"EQ",Id:"等于"},
          {key:"NEQ",Id:"不等于"},
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
    width: 98%;
    height: 1px;
    background-color: #999;
  }
  .labelContent{
    padding: 20px;
    border: 1px solid #999;
  }
</style>
