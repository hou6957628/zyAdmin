<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/productList' }">产品中心</el-breadcrumb-item>
      <el-breadcrumb-item>产品列表</el-breadcrumb-item>
      <el-breadcrumb-item>产品配置</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="120px" class="ruleForm">
      <span v-for="(item,index) in modeList">
          <el-button :id="item.module" @click="handleClick(item.module,item.moduleName,index)" style="width: 150px;margin-bottom: 20px;">{{item.moduleName}}</el-button>
      </span>
      <p style="padding: 20px 0;text-align: center;font-size: 18px; width: 740px;background-color: #fff;">{{moduleAttribute[0].moduleName}}配置</p>
      <div v-for="(item,index) in moduleAttribute">
        <div class="jiegou" v-if="ruleForm.mode==item.module" style="width: 700px">
          <el-form-item :label="item.nameText" prop="nameText">
            <div v-if="ruleForm.count==7">
              <el-select v-model="item.valueId" placeholder="请选择" @change="changeSelect1($event,index,selectList)">
                <el-option
                  v-for="item in selectList"
                  :key="item.id"
                  :label="item.flowName"
                  :value="item.id">
                </el-option>
              </el-select>
            </div>
            <div v-else-if="ruleForm.count==8">
              <el-select v-model="item.valueId" placeholder="请选择" @change="changeSelect2($event,index,selectList)">
                <el-option
                  v-for="item in selectList"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id">
                </el-option>
              </el-select>
            </div>
            <div v-else-if="ruleForm.count==13">
              <el-select v-if="index==0" v-model="item.valueId" placeholder="请选择">
                <el-option
                  v-for="item in enableList"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-select v-if="index==1" v-model="item.valueId" placeholder="请选择" @change="changeSelect1($event,index,amountList)">
                <el-option
                  v-for="item in amountList"
                  :key="item.id"
                  :label="item.flowName"
                  :value="item.id">
                </el-option>
              </el-select>
              <el-select v-if="index==2" v-model="item.valueId" placeholder="请选择">
                <el-option
                  v-for="item in enableList"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-select v-if="index==3" v-model="item.valueId" placeholder="请选择" @change="changeSelect1($event,index,orderFlowList)">
                <el-option
                  v-for="item in orderFlowList"
                  :key="item.id"
                  :label="item.flowName"
                  :value="item.id">
                </el-option>
              </el-select>
              <el-select v-if="index==4" v-model="item.valueId" placeholder="请选择">
                <el-option
                  v-for="item in enableList"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-input type="text" v-if="index==5" v-model="item.value"></el-input>
              <!--<el-select v-if="index==5" v-model="item.valueId" placeholder="请选择" @change="changeSelect3($event,index,selectList)">-->
                <!--<el-option-->
                  <!--v-for="item in selectList"-->
                  <!--:key="item.id"-->
                  <!--:label="item.name"-->
                  <!--:value="item.id">-->
                <!--</el-option>-->
              <!--</el-select>-->
              <el-select v-if="index==6" v-model="item.valueId" placeholder="请选择">
                <el-option
                  v-for="item in enableList"
                  :key="item.key"
                  :label="item.Id"
                  :value="item.key">
                </el-option>
              </el-select>
              <el-select v-if="index==7" v-model="item.valueId" placeholder="请选择" @change="changeSelect3($event,index,productList)">
                <el-option
                  v-for="item in productList"
                  :key="item.id"
                  :label="item.productName"
                  :value="item.id">
                </el-option>
              </el-select>
              <el-input type="textarea" v-if="index==8" v-model="item.value"></el-input>
              <!--<el-input type="textarea" v-if="index==9" v-model="item.value"></el-input>-->
            </div>
            <div v-else>
              <el-input v-model="item.value"></el-input>
            </div>
          </el-form-item>
      </div>
      </div>
        <el-form-item label-width="340px">
          <el-button type="primary" @click="submitForm(moduleAttribute[0].module,moduleAttribute[0].moduleName)">保存<i class="el-icon-check el-icon--right"></i></el-button>
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
        selectList:[],
        amountList:[],
        orderFlowList:[],
        productList:[],
        modeList:[],
        moduleAttribute:[
          {module:'',moduleName:'',name:'',nameText:'',value: '',valueId: '',}
        ],
        enableList:[
          {key:1,Id:'是'},
          {key:0,Id:'否'},
        ],
        ruleForm: {
          mode:'',
          moduleName:'',
          name:'',
          nameText:"",
          count:0,
        },
        rules: {
          cpaPrice: [
            { required: true, message: 'CPA单价', trigger: 'blur' },
            { min: 1, max: 5, message: '', trigger: 'blur' }
          ],
        },

      };
    },
    methods: {
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      selectChange(key){
        console.log(key);
        if (key == 0) {
          this.selectList=["1111"];
        }
      },
      //封装label值
      changeSelect1(vId,index,list){
        this.$forceUpdate();
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.moduleAttribute[index].value=obj.flowName;
      },
      //封装label值
      changeSelect2(vId,index,list){
        this.$forceUpdate();
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.moduleAttribute[index].value=obj.name;
      },
      //封装label值
      changeSelect3(vId,index,list){
        this.$forceUpdate();
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.moduleAttribute[index].value=obj.productName;
      },
      //查询所有配置
      loaderModelTable() {
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/loaderModelTable",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.modeList= res.data.body.list;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //查询每个配置的子项
      handleClick(key,id,num) {
        this.ruleForm.count=num;
        this.ruleForm.mode=key;
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/loaderModel",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            merchantId:this.merchantId,
            productId:this.id,
            mode:key,
            modeName:id
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.moduleAttribute= res.data.body.list;
            if (num == 7) {
              //获取所有风控流程
              this.getRiskList();
            } else if (num == 8) {
              //获取所有金融产品
              this.getBorrowingProductList();
            } else if (num == 13) {
              //获取所有提额流程
              this.getAmountList();
              //获取所有展期流程
              this.getOrderFlowList();
              //获取所有产品
              this.getProductList();
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //获取所有风控流程
      getRiskList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/getRiskList",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: 1,
            pageSize: 100,
          }
        }).then((res)=>{
          this.selectList=res.data.list;
        })
      },
      //获取所有提额流程
      getAmountList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/getAmountList",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: 1,
            pageSize: 100,
          }
        }).then((res)=>{
          this.amountList=res.data.list;
        })
      },
      //获取所有展期流程
      getOrderFlowList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/orderFlow/list",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: 1,
            pageSize: 100,
          }
        }).then((res)=>{
          console.log(res.data);
          this.orderFlowList=res.data.body.list;
          console.log(this.orderFlowList);
        })
      },
      //获取所有产品
      getProductList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: 1,
            pageSize: 200,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productList=res.data.body.list;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //获取金融产品列表
      getBorrowingProductList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/product/getList",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.selectList=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          } else {
            this.$message.error(res);
          }
        })
      },

      //保存
      submitForm(module,moduleName){
        var data  = {
          'merchantId': this.merchantId,
          'productId': this.id,
          'module': module,
          'moduleName': moduleName,
          'modValues': this.moduleAttribute,
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/saveModel",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data: JSON.stringify(data),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '配置成功',
              type: 'success'
            });
            this.$router.push('/productProductList');
          }else if(res.data.msgCd=='ZYCASH-SUPERMARKET-1009'){
            this.$message.error(res.data.msgInfo);
          } else {
            this.$message.error(res);
          }
        })
      }
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.merchantId=this.$route.params.merchantId;
      this.handleClick('Face','face++');
      this.loaderModelTable();
    },
  }
</script>

<style scoped>
  /*.content .el-form-item{*/
    /*margin-bottom: 0;*/
  /*}*/
  .el-form-item__content{
    line-height: 30px;
  }
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
    padding-bottom: 2px;
    padding-top: 20px;
  }
</style>
