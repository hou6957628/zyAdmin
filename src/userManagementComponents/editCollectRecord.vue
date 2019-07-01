<template>
  <div>
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>用户催收记录</el-breadcrumb-item>
      <el-breadcrumb-item>详情</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="用户类型:" prop="collectionLabelId">
        <el-select v-model="ruleForm.collectionLabelId" placeholder="请选择" @change="changeSelect1($event,labelList)">
          <el-option
            v-for="item in labelList"
            :key="item.id"
            :label="item.labelContent"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="通话类型:" prop="callTypeId">
        <el-select v-model="ruleForm.callTypeId" placeholder="请选择" @change="changeSelect2($event,callTypeList)">
          <el-option
            v-for="item in callTypeList"
            :key="item.id"
            :label="item.callContent"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="备注:">
        <el-input type="textarea" v-model="ruleForm.memo"></el-input>
      </el-form-item>
      <el-form-item v-for="(domain, index) in picList" label="图片:" :key="index">
        <img class="ID1" style="height: 108px;width: 170px" :src="'http://'+baseUrl + '/credit/api/down?file='+domain.collectRecordImage"/>
      </el-form-item>
      <el-form-item label="上传图片:" v-for="(domain, index) in fileList" :key="index">
        <a class="upload-file" href="javascript:;" v-model="domain.filename">{{domain.image}}<input type="file"
          accept="image/png,image/gif,image/jpeg" value="上传弹窗图片"  @change="tirggerFile($event,index)"></a>
        <input class="fileBox" type="hidden" v-model="domain.filename">
      </el-form-item>
      <el-form-item style="margin-top: 10px">
        <el-button @click="addDomain" size="medium">添加图片</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
        <el-button type="info" @click="resetForm('ruleForm')">取消<i class="el-icon-close el-icon--right"></i></el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      return {
        labelList: [],
        callTypeList: [],
        files: [],
        picList: [],
        fileList: [
          { filename: '', image: '',}
        ],
        ruleForm: {
          id:null,
          collectionLabelId: '',
          collectionLabelName: '',
          callTypeId: '',
          callTypeName: '',
          memo: '',
          productCode: '',
        },
        rules: {
          collectionLabelId: [
            { required: true, message: '请选择用户类型', trigger: 'change' }
          ],
          callTypeId: [
            { required: true, message: '请选择通话类型', trigger: 'change' }
          ],
        }
      };
    },
    methods: {
      //所有催收标签
      getLabelList(data1,data2){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/label/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.labelList=res.data.body.list;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //所有通话类型
      getCallTypeList(data1,data2){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/callType/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.callTypeList=res.data.body.list;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //封装名称
      changeSelect1(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.ruleForm.collectionLabelName=obj.labelContent;
      },
      //封装名称
      changeSelect2(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.ruleForm.callTypeName=obj.callContent;
      },
      //选择图片
      tirggerFile($event,index){
        var file = $event.target.files[0];
        var name = $event.target.files[0].name;
        this.fileList[index].filename = file;
        this.fileList[index].image = name;
        this.files[index]=file;
      },
      //添加图片
      addDomain(){
        this.fileList.push({
          filename: '',
          image: '',
        });
      },
      //提交按钮
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            this.files.forEach(function (item,index) {
              param.append('files', item);
            });
            param.append('id', this.ruleForm.id);
            param.append('collectionLabelId', this.ruleForm.collectionLabelId);
            param.append('collectionLabelName', this.ruleForm.collectionLabelName);
            param.append('callTypeId', this.ruleForm.callTypeId);
            param.append('callTypeName', this.ruleForm.callTypeName);
            param.append('memo', this.ruleForm.memo);
            param.append('productCode', this.ruleForm.productCode);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/order/admin/collect/update",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.$router.go(-1);
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
      //催收记录详情
      getCollectRecord(data){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/collect/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: data,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm=res.data.body.record;
            this.picList=res.data.body.fileList;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getLabelList(1,200);
      this.getCallTypeList(1,200);
      this.getCollectRecord(this.id);
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
  .ID1{
    transition: All 0.4s ease-in-out;
    -webkit-transition: All 0.4s ease-in-out;
    -moz-transition: All 0.4s ease-in-out;
    -o-transition: All 0.4s ease-in-out;
    z-index: 1000;
    cursor: pointer;
  }
  .ID1:hover{
    transform: translate(-200px,0) scale(4);
    -webkit-transform: translate(-200px,0) scale(4);
    -moz-transform: translate(-200px,0) scale(4);
    -o-transform: translate(-200px,0) scale(4);
    -ms-transform: translate(-200px,0) scale(4);
    z-index: 3000;
  }
</style>
