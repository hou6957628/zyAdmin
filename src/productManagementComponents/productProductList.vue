<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/financeProductList' }">产品列表</el-breadcrumb-item>
      <el-breadcrumb-item>产品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">
        创建产品&nbsp;<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      <el-button class="upLoadBtn" @click="toMerchantProductList()" type="primary">
        商户列表&nbsp;<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      <el-input @click="searchProduct" class="searchContent"
        placeholder="产品名称、编号搜索"
        v-model="finProduct"
        clearable>
        <el-button id="searchBtn" @click="searchContent(finProduct)" slot="append" icon="el-icon-search"></el-button>
      </el-input>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="productCode"
          label="产品编号"
          width="150">
        </el-table-column>
        <el-table-column
          fixed
          prop="productName"
          label="产品名称"
          min-width="120">
        </el-table-column>
        <el-table-column
          prop="description"
          label="产品说明"
          width="120">
        </el-table-column>
        <el-table-column
          prop="merchantName"
          label="商户名称"
          width="120">
        </el-table-column>
        <el-table-column
          prop="riskControlFlow"
          label="风控流程"
          width="120">
        </el-table-column>
        <el-table-column
          prop="finaceProduct"
          label="金融产品"
          width="200">
        </el-table-column>
        <el-table-column
          prop="enabledRaise"
          label="是否提额"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.enabledRaise == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enabledRaise == true ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="enabledSuperLoan"
          label="是否展示贷超"
          width="120">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.enabledSuperLoan == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enabledSuperLoan == true ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="创建时间"
          width="180">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="更新时间"
          width="180">
        </el-table-column>
        <el-table-column
          prop="creator"
          label="创建人员"
          width="80">
        </el-table-column>
        <el-table-column
          prop="enabled"
          label="状态"
          width="80">
          <template slot-scope="scope" style="text-align: center">
            <el-tag
              :type="scope.row.enabled == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enabled == true ? '使用中' : '已停用'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="220">
          <template slot-scope="scope">
            <el-button @click="configureProduct(scope.row)" type="text" size="medium">配置</el-button>
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button @click="copyProduct(scope.row)" type="text" size="medium">复制</el-button>
            <el-button v-if="scope.row.enabled" @click="obtainedProduct(scope.row)" type="text" size="medium">停用</el-button>
            <el-button v-if="!scope.row.enabled" @click="obtainedProduct(scope.row)" type="text" size="medium">启用</el-button>
          </template>
        </el-table-column>
      </el-table>
    </template>
    <div class="block">
      <el-pagination class="paginationBox"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :unique-opened="true"
        :current-page="pageNum"
        :page-sizes="pageSizes"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="proTotal">
      </el-pagination>
    </div>
    <!--复制产品结构-->
    <el-dialog
      title="复制产品"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="产品名称:" prop="productName">
          <el-input v-model="ruleForm.productName"></el-input>
        </el-form-item>
        <el-form-item label="产品说明:">
          <el-input v-model="ruleForm.description"></el-input>
        </el-form-item>
        <el-form-item label="商户名称:" prop="merchantId">
          <el-select v-model="ruleForm.merchantId" placeholder="请选择">
            <el-option
              v-for="item in electData"
              :key="item.id"
              :label="item.merchantName"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="copyCustomerTag('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    methods: {
      //查询金融产品
      searchContent(data){
        if(data==""){
          this.getProductList(1,20,null,null);
          // this.$message.error('搜索内容不可以为空');
        }else {
          this.getProductList(1,20,data,this.finProduct);
          console.log(data);
        }
      },
      //每页显示多少条
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.getProductList(this.pageNum,val,this.finProduct,this.finProduct);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        console.log(this.nowPageSizes);
        this.getProductList(val,this.nowPageSizes,this.finProduct,this.finProduct);
      },
      //创建金融产品
      toAddProduct(){
        this.$router.push({
          path: `/addProduct`,
        });
      },
      //商户列表
      toMerchantProductList(){
        this.$router.push({
          path: `/merchantProductList`,
        });
      },
      /**
       * 获取产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 商户名称
       * @param data4 商户编号
       */
      getProductList(data1,data2,data3,data4){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            productName: data3,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body.list;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //查询产品接口
      searchProduct(){
        this.getProductList(1,20,this.finProduct,this.finProduct);
      },
      //配置产品接口
      configureProduct(row){
        var id=row.id;
        var merchantId=row.merchantId;
        this.$router.push({
          path: `/configProduct/${id}/${merchantId}`,
        });
      },
      //编辑产品接口
      editProduct(row){
        var id=row.id;
        this.$router.push({
          path: `/editProduct/${id}`,
        });
      },
      //删除产品接口
      deleteProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/delOrStopProduct",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
            status: 1,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.dialogFormVisible = false;
            this.getProductList(1,10,this.finProduct,this.finProduct);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //提示复制产品接口
      copyProduct(row){
        this.copyId = row.id;
        this.centerDialogVisible=true;
      },
      //确认复制用户标签接口
      copyCustomerTag(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            console.log(this.type);
            var param = new FormData();  // 创建form对象
            param.append('id', this.copyId)  // 通过append向form对象添加数据
            param.append('productName', this.ruleForm.productName)  // 通过append向form对象添加数据
            param.append('description', this.ruleForm.description) // 添加form表单中其他数据
            param.append('merchantId', this.ruleForm.merchantId) // 添加form表单中其他数据
            if (!this.type) {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/operate/admin/productManage/copyProduct",
                headers:{
                  'Content-Type':'application/x-www-form-urlencoded',
                  'Authorization': localStorage.token
                },
                data:param,
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.centerDialogVisible=false;
                  this.$message({
                    message: '复制成功',
                    type: 'success'
                  });
                  this.getProductList(1,20,null,null);
                }else if(res.data.msgCd=='ZYCASH-1009'){
                  this.$message.error(res.data.msgInfo);
                }
                else {
                  this.$message.error(res);
                }
              })
            } else {
              axios({
                method:"POST",
                url:"http://"+this.baseUrl+"/operate/admin/productManage/saveProduct",
                headers:{
                  'Content-Type':'application/x-www-form-urlencoded',
                  'Authorization': localStorage.token
                },
                data:param,
              }).then((res)=>{
                if(res.data.msgCd=='ZYCASH-200'){
                  this.$message({
                    message: '修改成功',
                    type: 'success'
                  });
                  this.$router.push('/productProductList');
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
      //停用产品接口
      obtainedProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/delOrStopProduct",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
            status: 0,
            enabled: !row.enabled,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,20,this.finProduct,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //获取商户列表
      getMerchantList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/merchant/getNameList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.electData= res.data.body.list;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted:function () {
      this.getMerchantList();
      this.getProductList(1,20,null,null);
    },
    data() {
      //验证产品名称是否重复
      var validateName = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入产品名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/operate/admin/productManage/checkRepeateName",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              productName: value
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body == false) {
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
        tableData: [],
        finProduct: '',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
        centerDialogVisible:false,
        ruleForm: {
          productName: '',
          description: '',
          merchantId: null,
        },
        rules: {
          productName: [
            { required: true, validator: validateName, trigger: 'blur' }
          ],
          merchantId: [
            { required: true, message: '请选择商户名称', trigger: 'change' }
          ]
        },
        electData:[],
        dialogFormVisible: false
      }
    }
  }
</script>

<style scoped>
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
    margin: 25px 30px 15px 0;
  }
  .operationContent .upLoadBtn{

  }
  .operationContent .searchContent{
    margin-left: 20px;
    width: 300px;
    margin-right: 30px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
