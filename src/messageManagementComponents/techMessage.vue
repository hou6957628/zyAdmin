<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/messageConfigurationList' }">任务列表</el-breadcrumb-item>
      <el-breadcrumb-item>创建任务</el-breadcrumb-item><el-breadcrumb-item>技术人员</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="160px" class="demo-ruleForm">
      <el-form-item style="margin-top: 20px;width: 480px" label="选择产品：" prop="productId">
        <el-select style="width: 320px" v-model="ruleForm.productId" value-key="id" placeholder="请选择" @change="selectChange1($event,productList)">
          <el-option
            v-for="item in productList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="选择形式：" prop="modeId" style="margin-top: 20px;width: 480px">
        <el-select style="width: 320px" v-model="ruleForm.modeId" placeholder="请选择" @change="selectChange2($event,modeList)">
          <el-option
            v-for="item in modeList"
            :key="item.id"
            :label="item.modeName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="消息名称：" prop="messageName" style="margin-top: 20px;width: 480px;position: relative;">
        <el-input v-model="ruleForm.messageName" placeholder="请输入消息名称"></el-input>
      </el-form-item>
      <el-form-item label="bean名称：" prop="beanName" style="margin-top: 20px;width: 480px;position: relative;">
        <el-input v-model="ruleForm.beanName" placeholder="请输入bean名称"></el-input>
      </el-form-item>
      <el-form-item label="方法名称：" prop="methodName" style="margin-top: 20px;width: 480px;position: relative;">
        <el-input v-model="ruleForm.methodName" placeholder="请输入方法名称"></el-input>
      </el-form-item>
      <el-form-item label="参数：" prop="params" style="margin-top: 20px;width: 480px;position: relative;">
        <el-input v-model="ruleForm.params" placeholder="请输入参数"></el-input>
      </el-form-item>
      <el-form-item label="cron表达式：" prop="cron" style="margin-top: 20px;width: 480px;position: relative;">
        <el-input v-model="ruleForm.cron" placeholder="请输入cron表达式"></el-input>
      </el-form-item>
      <el-form-item label="备注：">
        <el-input type="textarea" :rows="4" placeholder="请输入备注" v-model="ruleForm.description"></el-input>
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
        modeCode:'short_message',
        productList:[],
        modeList:[],
        messageClassifyList:[],
        ruleForm: {
          productId:'',
          productName:'',
          productCode:'',
          modeId:'',
          modeName:'',
          messageName:'',
          beanName:'',
          methodName:'',
          params:'',
          cron:'',
          description:'',
        },
        rules: {
          productId: [
            {required: true, message: '请选择产品', trigger: 'change'}
          ],
          modeId: [
            {required: true, message: '请选择形式', trigger: 'change'}
          ],
          messageName: [
            {required: true, message: '请输入消息名称', trigger: 'change'}
          ],
          beanName: [
            {required: true, message: '请输入bean名称', trigger: 'change'}
          ],
          methodName: [
            {required: true, message: '请输入方法名称', trigger: 'change'}
          ],
          params: [
            {required: true, message: '请输入参数', trigger: 'change'}
          ],
          cron: [
            {required: true, message: '请输入cron表达式', trigger: 'change'}
          ],
        },
        tableData:[],
        taskList:[],
        msg:'',
      };
    },
    methods: {
      //查询所有产品
      getProduct() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getProductList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productList=res.data.body;
          } else if (res.data.msgCd=='ZYCASH-1005') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else if (res.data.msgCd=='SYS00001') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else {
            this.$message.error(res);
          }
        })
      },
      //查询所有形式
      getModeList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_mode/findList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.modeList=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //封装产品名称
      selectChange1(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.productId === vId;
        });
        this.ruleForm.productName=obj.productName;
        this.ruleForm.productCode=obj.productCode;
      },
      //封装形式名称
      selectChange2(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.ruleForm.modeName=obj.modeName;
      },
      //保存任务
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();
            param.append('productId', this.ruleForm.productId);
            param.append('productName', this.ruleForm.productName);
            param.append('productCode', this.ruleForm.productCode);
            param.append('modeId', this.ruleForm.modeId);
            param.append('modeName', this.ruleForm.modeName);
            param.append('messageName', this.ruleForm.messageName);
            param.append('beanName', this.ruleForm.beanName);
            param.append('methodName', this.ruleForm.methodName);
            param.append('params', this.ruleForm.params);
            param.append('cron', this.ruleForm.cron);
            param.append('description', this.ruleForm.description);
            param.append('type', 3);
            axios({
              method: "POST",
              url: "http://"+this.baseUrl+"/message/admin/save/task",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.$router.push('/messageConfigurationList');
              } else {
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
    },
    mounted: function () {
      this.getProduct();
      this.getModeList();
    },
  }
</script>

<style scoped>
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }

  .demo-ruleForm {
    width: 800px;
    text-align: center;
    margin-top: 50px;

  }

  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .searchContent{
    width: 200px;
  }
  .operationContent{
    text-align: left;
    margin: 25px 30px 15px 0;
  }
  .operationContent .upLoadBtn{
    margin-right: 15px;
  }
  .operationContent .searchContent{
    margin-left:0px;
    width: 200px;
    margin-right: 30px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
  .bankBox p{
    height: 35px;
    line-height: 25px;
  }
  .bankBox p span{
    display: inline-block;
    width: 100px;
    text-align: center;
  }
</style>
