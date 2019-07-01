<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>推送消息</el-breadcrumb-item>
      <el-breadcrumb-item>编辑</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="160px" class="demo-ruleForm">
      <el-form-item style="margin-top: 20px" label="产品：" prop="productId">
        <el-select v-model="ruleForm.productId" value-key="id" placeholder="请选择" @change="selectChange1($event,productList)">
          <el-option
            v-for="item in productList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="名称：" prop="name">
        <el-input v-model="ruleForm.name"></el-input>
      </el-form-item>
      <el-form-item style="margin-top: 20px" label="请选择分类：" prop="classifyId">
        <el-select v-model="ruleForm.classifyId" value-key="id" placeholder="请选择" @change="selectChange2($event,messageClassifyList)">
          <el-option
            v-for="item in messageClassifyList"
            :key="item.id"
            :label="item.classifyName"
            :value="item.id">
          </el-option>
        </el-select>
        <el-button style="position: absolute;right:-50px;top:1px;" @click="toAddProduct()" type="primary" plain>添加分类</el-button>
      </el-form-item>
      <el-form-item label="标题：" prop="title">
        <el-input v-model="ruleForm.title"></el-input>
      </el-form-item>
      <el-form-item label="请输入备注：">
        <el-input type="textarea" :rows="4" v-model="ruleForm.description"></el-input>
      </el-form-item>
      <el-form-item label="请输入内容：" prop="content">
        <el-input type="textarea" :rows="4" v-model="ruleForm.content"></el-input>
      </el-form-item>
      <el-form-item label="已有LOGO：" v-if="ruleForm.picture">
        <img style="height: 108px;width: 170px" :src="ruleForm.picture"/>
      </el-form-item>
      <el-form-item label="点此修改LOGO：">
        <a class="upload-file" href="javascript:;">{{ruleForm.fileName}}
          <input type="file" accept="image/png,image/gif,image/jpeg" value="上传弹窗图片" @change="tirggerFile($event)">
        </a>
        <el-alert :closable="false" style="padding: 3px 16px;" title="icon推荐尺寸（80*80）" type="success"></el-alert>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">保存</el-button>
        <el-button type="" @click="resetForm('ruleForm')">取消</el-button>
      </el-form-item>
    </el-form>
    <!--创建分类-->
    <el-dialog
      title="添加分类"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="80px" class="demo-ruleForm">
        <el-form-item label="名称:" prop="classifyName">
          <el-input v-model="ruleForm2.classifyName" placeholder="请填写名称"></el-input>
        </el-form-item>
        <el-form-item label="备注:" prop="description">
          <el-input type="textarea":rows="3" v-model="ruleForm2.description" placeholder="请填写备注"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="insertMessage('ruleForm2')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      //分类名称不可重复
      var validateName = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请填写名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/message/admin/message_classify/check",
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
        ruleForm: {
          id:'',
          productId:'',
          productName:'',
          productCode:'',
          name:'',
          classifyId: '',
          classifyName: '',
          title: '',
          description:'',
          content:'',
          file:'',
          fileName:'',
          modeName:'推送消息',
          modeCode:'push_message',
        },
        rules: {
          productId: [
            {required: true, message: '请选择产品', trigger: 'change'}
          ],
          name: [
            {required: true, message: '请填写名称', trigger: 'blur'}
          ],
          classifyId: [
            {required: true, message: '请选择分类', trigger: 'change' }
          ],
          title: [
            {required: true, message: '请输入标题', trigger: 'blur' }
          ],
          content: [
            {required: true, message: '请输入内容', trigger: 'blur'}
          ],
        },
        centerDialogVisible:false,
        ruleForm2: {
          classifyName: '',
          description: '',
        },
        rules2: {
          classifyName: [
            { required: true, validator: validateName, trigger: 'blur' }
          ],
        },
        productList:[],
        messageClassifyList:[],
      };
    },
    methods: {
      /**
       * 获取选择所属APP
       */
      getFenList(){
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
      //查询所有分类
      getProductList1() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_classify/findList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.messageClassifyList=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //查询消息详情
      getMessageById(id) {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //提交按钮
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();
            param.append('id', this.ruleForm.id);
            param.append('productId', this.ruleForm.productId);
            param.append('productName', this.ruleForm.productName);
            param.append('productCode', this.ruleForm.productCode);
            param.append('name', this.ruleForm.name);
            param.append('classifyId', this.ruleForm.classifyId);
            param.append('classifyName', this.ruleForm.classifyName);
            param.append('title', this.ruleForm.title);
            param.append('description', this.ruleForm.description);
            param.append('modeName', this.ruleForm.modeName);
            param.append('modeCode', this.ruleForm.modeCode);
            param.append('content', this.ruleForm.content);
            param.append('file', this.ruleForm.file);
            axios({
              method: "POST",
              url: "http://"+this.baseUrl+"/message/admin/message/update",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data: param,
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.$router.push('/pushMessage');
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
      //封装产品名称
      selectChange1(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.productId === vId;
        });
        this.ruleForm.productName=obj.productName;
        this.ruleForm.productCode=obj.productCode;
      },
      //封装分类名称
      selectChange2(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.ruleForm.classifyName=obj.classifyName;
      },
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      //创建分类
      toAddProduct(){
        this.centerDialogVisible=true;
      },
      //保存分类
      insertMessage(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('classifyName', this.ruleForm2.classifyName);
            param.append('description', this.ruleForm2.description);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/message/admin/message_classify/insert",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '保存成功',
                  type: 'success'
                });
                this.centerDialogVisible=false;
                this.getProductList1();
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
      //上传ICON图标
      tirggerFile($event,index){
        var file = $event.target.files[0];
        var name = $event.target.files[0].name;
        this.ruleForm.file = file;
        this.ruleForm.fileName = name;
        this.$forceUpdate();
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getMessageById(this.id);
      this.getProductList1();
      this.getFenList();
    },
  }
</script>

<style scoped>
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }

  .demo-ruleForm {
    width: 480px;
    text-align: left;
    margin-top: 50px;
  }

  .fs-16 {
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
    width: 92%;
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
