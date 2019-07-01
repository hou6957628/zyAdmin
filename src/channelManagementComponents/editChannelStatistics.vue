<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/channelCount' }">渠道统计列表</el-breadcrumb-item>
      <el-breadcrumb-item>编辑</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="计价方式:">
        <el-select v-model="countType" placeholder="请选择">
          <el-option
            v-for="item in countTypeList"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
        <div class="jiegou" v-if="countType==1" label="cpa计价">
          <p style="margin-bottom: 10px;text-align: center;font-size: 18px">CPA计价</p>
          <el-form-item label="产品名称">
            <el-input v-model="ruleForm.productName" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="对应日期">
            <el-input v-model="ruleForm.statisticsDate" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="CPA单价" prop="cpaPrice">
            <el-input v-model="ruleForm.cpaPrice"></el-input>
          </el-form-item>
          <el-form-item label="CPA数量" prop="cpaNum" >
            <el-input v-model="ruleForm.cpaNum"></el-input>
          </el-form-item>
          <el-form-item label="备注" prop="remark">
            <el-input v-model="ruleForm.remark"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="CPAForm(1,'ruleForm')">保存</el-button>
            <el-button type="info" @click="cancel()">返回</el-button>
          </el-form-item>
        </div>
        <div class="jiegou" v-if="countType==2" label="cps计价">
          <p style="margin-bottom: 10px;text-align: center;font-size: 18px">CPS计价</p>
          <el-form-item label="产品名称">
            <el-input v-model="ruleForm.productName" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="对应日期">
            <el-input v-model="ruleForm.statisticsDate" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="CPS单价" prop="cpsPrice">
            <el-input v-model="ruleForm.cpsPrice"></el-input>
          </el-form-item>
          <el-form-item label="CPS数量" prop="cpsNum">
            <el-input v-model="ruleForm.cpsNum"></el-input>
          </el-form-item>
          <el-form-item label="备注" prop="remark">
            <el-input v-model="ruleForm.remark"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="CPSForm(2,'ruleForm')">保存</el-button>
            <el-button type="info" @click="cancel()">返回</el-button>
          </el-form-item>
        </div>
        <div class="jiegou" v-if="countType==3" label="uv计价">
          <p style="margin-bottom: 10px;text-align: center;font-size: 18px">UV计价</p>
          <el-form-item label="产品名称">
            <el-input v-model="ruleForm.productName" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="对应日期">
            <el-input v-model="ruleForm.statisticsDate" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="UV单价" prop="uvPrice">
            <el-input v-model="ruleForm.uvPrice"></el-input>
          </el-form-item>
          <el-form-item label="UV数量" prop="uvNum">
            <el-input v-model="ruleForm.uvNum"></el-input>
          </el-form-item>
          <el-form-item label="备注" prop="remark">
            <el-input v-model="ruleForm.remark"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="UVForm(3,'ruleForm')">保存</el-button>
            <el-button type="info" @click="cancel()">返回</el-button>
          </el-form-item>
        </div>
    </el-form>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      return {
        countTypeList:[
          {key:1,Id:'按照cpa计价'},
          {key:2,Id:'按照cps计价'},
          {key:3,Id:'按照uv计价'},
        ],
        ruleForm: {
          cpaPrice:'',
          cpaNum:'',
          cpsPrice:'',
          cpsNum:'',
          uvPrice:'',
          uvNum:'',
          remark:'',
        },
        rules: {
          cpaPrice: [
            { required: true, message: '请填写CPA单价', trigger: 'blur' },
          ],
          cpaNum: [
            { required: true, message: '请填写CPA数量', trigger: 'blur' },
          ],
          cpsPrice: [
            { required: true, message: '请填写CPS单价', trigger: 'blur' },
          ],
          cpsNum: [
            { required: true, message: '请填写CPS数量', trigger: 'blur' },
          ],
          uvPrice: [
            { required: true, message: '请填写UV单价', trigger: 'blur' },
          ],
          uvNum: [
            { required: true, message: '请填写UV数量', trigger: 'blur' },
          ],
        },
        cpaPrice:'',
        cpsPrice:'',
        uvPrice:'',
        countType:null,
      };
    },
    methods: {
      //查询渠道统计信息
      getProductList(data1){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/channel/admin/channel_statistics/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: this.id
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm=res.data.body;
            this.countType=res.data.body.countType;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      CPAForm(data,formName){
        this.$refs[formName].validate((valid) => {
            if (valid) {

            } else {
              console.log('error submit!!');
              return false;
            }
        });
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/channel/admin/channel_statistics/update",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: this.id,
            countType:data,
            cpaPrice:this.ruleForm.cpaPrice,
            cpaNum:this.ruleForm.cpaNum,
            remark:this.ruleForm.remark,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.$router.push('/channelCount');
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      CPSForm(data,formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {

          } else {
            console.log('error submit!!');
            return false;
          }
        });

        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/channel/admin/channel_statistics/update",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: this.id,
            countType:data,
            cpsPrice:this.ruleForm.cpsPrice,
            cpsNum:this.ruleForm.cpsNum,
            remark:this.ruleForm.remark,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.$router.push('/channelCount');
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      UVForm(data,formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {

          } else {
            console.log('error submit!!');
            return false;
          }
        });

        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/channel/admin/channel_statistics/update",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: this.id,
            countType:data,
            uvPrice:this.ruleForm.uvPrice,
            uvNum:this.ruleForm.uvNum,
            remark:this.ruleForm.remark,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.$router.push('/channelCount');
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //取消
      cancel(){
        this.$router.go(-1);
      }
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getProductList(1,10);
    },
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
  .jiegou{
    background-color: #fff;
    padding-right: 40px;
    padding-bottom: 20px;
    padding-top: 20px;
  }
</style>
