<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/merchantProductList' }">商户列表</el-breadcrumb-item>
      <el-breadcrumb-item>商户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">添加商户<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="merchantCode"
          label="商户编号"
          width="150">
        </el-table-column>
        <el-table-column
          prop="merchantName"
          label="商户名称"
          width="200">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="创建时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="更新时间"
          width="200">
        </el-table-column>
        <el-table-column
          prop="enabled"
          label="状态"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.enabled == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enabled == true ? '使用中' : '已停用'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="180">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button @click="deleteProductTip(scope.row)" type="text" size="medium">删除</el-button>
            <el-button v-if="scope.row.enabled" @click="obtainedProduct(scope.row)" type="text" size="medium">停用</el-button>
            <el-button v-if="!scope.row.enabled" @click="obtainedCustomerTag(scope.row)" type="text" size="medium">启用</el-button>
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

  </div>
</template>


<script>
  import axios from 'axios'
  import qs from 'qs';
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
          path: `/addMerchant`,
        });
      },
      /**
       * 获取金融产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       */
      getProductList(data1,data2){
        var data = qs.stringify({
          pageNum: data1,
          pageSize: data2
        });
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/merchant/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
          }
          // data:data,
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
      //编辑产品接口
      editProduct(row){
        var id=row.id;
        this.$router.push({
          path: `/editMerchant/${id}`,
        });
      },
      //提示删除商户接口
      deleteProductTip(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/merchant/verifyMerchantBeforeDelete",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            merchantCode: row.merchantCode,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body == false) {
              this.$alert('商户使用中无法删除', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              this.$confirm('是否确认删除此商户?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.deleteProduct(row);
              }).catch(() => {
                this.$message({
                  type: 'info',
                  message: '已取消删除'
                });
              });
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //确认删除商户接口
      deleteProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/merchant/edit",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token,
            'merchantCode': row.merchantCode,
          },
          params:{
            enableDelete: true,
            merchantCode: row.merchantCode,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,20,this.finProduct,this.finProduct);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //提示启用停用用户标签接口
      obtainedProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/merchant/verifyMerchantBeforeDelete",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            merchantCode: row.merchantCode,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body == false) {
              this.$alert('商户使用中无法停用', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              this.$confirm('是否确认停用此商户?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.obtainedCustomerTag(row);
              }).catch(() => {
                this.$message({
                  type: 'info',
                  message: '已取消操作'
                });
              });
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //确认启用停用用户标签接口
      obtainedCustomerTag(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/merchant/edit",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            merchantCode: row.merchantCode,
            enabled: !row.enabled,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,20,this.finProduct);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //提交表单
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('merchantName', this.ruleForm.merchantName);
            param.append('enabled', this.ruleForm.enabled);
            param.append('enableDelete', 0);
            param.append('company', null);
            param.append('companyAddress', this.ruleForm.companyAddress);
            param.append('companyDetail', this.ruleForm.companyDetail);
            param.append('email', null);
            param.append('mobile', null);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/operate/admin/merchant/save",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.$router.push('/merchantProductList');
              }else if(res.data.msgCd=='ZYCASH-SUPERMARKET-1009'){
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
      //关闭表单
      noSubmitForm(){
        this.bg=false;
        this.ruleForm.merchantName='';
        this.ruleForm.companyAddress='';
        this.ruleForm.companyDetail='';
        this.ruleForm.enabled=1;
      },
    },
    mounted:function () {
      // this.finProduct=this.$route.params.name;
      this.getProductList(1,20);
    },
    data() {
      return {
        tableData: [],
        electData:[
          {key:0,Id:'不启用'},
          {key:1,Id:'启用'},
        ],
        bg:false,
        finProduct: '',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
        ruleForm: {
          merchantName: '',
          companyAddress: '',
          companyDetail: '',
          enabled: 1,
        },
        rules: {
          merchantName: [
            { required: true, message: '请填写商户名称', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
          ],
          companyAddress: [
            {  required: true, message: '请填写公司地址', trigger: 'change' }
          ],
          companyDetail: [
            { required: true, message: '请填写公司描述', trigger: 'change' }
          ]
        }
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
  .bg{
    display: block;
    position: absolute;
    top: 0%;
    left: 0%;
    width: 100%;
    height: 100%;
    text-align: left;
    z-index: 1001;
    background: rgba(0,0,0,0.8);
  }
  .demo-ruleForm{
    width: 500px;
    height: 300px;
    background-color: #fff;
    margin: 0 auto;
    margin-top: 100px;
    padding: 30px 30px 30px 0px;
    text-align: left;
  }
</style>
