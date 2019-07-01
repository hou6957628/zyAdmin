<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/financeProductList' }">金融产品列表</el-breadcrumb-item>
      <el-breadcrumb-item>金融产品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">创建金额产品<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      <el-input class="searchContent"
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
          prop="name"
          label="产品名称"
          min-width="120">
        </el-table-column>
        <el-table-column
          prop="description"
          label="说明"
          width="120">
        </el-table-column>
        <el-table-column
          prop="type"
          label="类型"
          :formatter="typeFormatter"
          width="120">
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
          prop="interestMethods"
          label="利息方式"
          :formatter="methodFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          prop="obtainedDate"
          label="利息金额"
          width="120">
        </el-table-column>
        <el-table-column
          prop="minCapital"
          label="合同金额"
          width="120">
        </el-table-column>
        <el-table-column
          prop="maxCapital"
          label="最大合同额"
          width="120">
        </el-table-column>
        <el-table-column
          prop="creator"
          label="创建人员"
          width="120">
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
          fixed="right"
          label="操作"
          width="250">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button @click="openWarningDelete(scope.row)" type="text" size="medium">删除</el-button>
            <el-button @click="copyProduct(scope.row)" type="text" size="medium">复制</el-button>
            <el-button @click="obtainedProduct(scope.row)" type="text" size="medium">停用</el-button>
            <el-button @click="obtainedProduct(scope.row)" type="text" size="medium">启用</el-button>
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
    <!--复制金融产品结构-->
    <el-dialog
      title="复制金融产品"
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
  import { Loading } from 'element-ui'
  export default {
    methods: {
      //查询金融产品
      searchContent(data){
        Loading.service();
        this.getProductList(1,20,this.finProduct);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,this.finProduct);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.finProduct);
      },
      //创建金融产品
      toAddProduct(){
        this.$router.push({
          path: `/addFinanceProduct`,
        });
      },
      /**
       * 获取金融产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 产品名称
       * @param data4 产品编号
       */
      getProductList(data1,data2,data3){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/product/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            name: data3,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body.list;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
            let loadingInstance = Loading.service();
            this.$nextTick(() => { // 以服务的方式调用的 Loading 需要异步关闭
              loadingInstance.close();
            });
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //编辑产品接口
      editProduct(row){
        var id=row.id;
        this.$router.push({
          path: `/editFinanceProduct/${id}`,
        });
      },
      //删除提示窗
      openWarningDelete(row){
        if (row.position==null) {
          this.$confirm('是否确认删除此产品信息?', '提示', {
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
        } else {
          this.$alert('此产品暂时无法删除，请联系管理员', '提示', {
            dangerouslyUseHTMLString: true
          });
        }
      },
      //删除产品接口
      deleteProduct(row){
        axios({
          method:"post",
          // url:"http://"+this.baseUrl+"/super/admin/product/obtainedProduct",
          url:"http://"+this.baseUrl+"/operate/admin/product/delOrStopProduct",
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
            this.dialogFormVisible = false;
            this.$message({
              message: '操作成功',
              type: 'success'
            });
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
      //确定复制产品接口
      copyCustomerTag(){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method:"post",
              url:"http://"+this.baseUrl+"/operate/admin/product/copyProduct",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params:{
                id: this.copyId,
                name: this.ruleForm.productName,
                description: this.ruleForm.description,
              }
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.centerDialogVisible = false;
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.getProductList(1,20,null,null);
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
      //停用产品接口
      obtainedProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/product/delOrStopProduct",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
            //status: 0,
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
      //过滤性别字段
      typeFormatter(row){
        let type = row.type;
        if(type == 0){
          return '现金贷'
        } else if (type == 1){
          return '分期'
        } else {
          return '未知'
        }
      },
      //过滤利息方式
      methodFormatter(row){
        let interestMethods = row.interestMethods;
        if(interestMethods == 0){
          return '每期等额本息'
        } else if (interestMethods == 1){
          return '每期付息到期还本'
        } else if (interestMethods == 2){
          return '到期还本付息'
        } else if (interestMethods == 3){
          return '等本等息'
        } else if (interestMethods == 4){
          return '等额本金'
        } else if (interestMethods == 5){
          return '放款时一次性收取'
        }
      },
    },
    mounted:function () {
      this.getProductList(1,20,null,null);
    },
    data() {
      return {
        tableData: [],
        finProduct: '',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
        centerDialogVisible: false,
        rowData: null,
        ruleForm: {
          productName: '',
          description: '',
        },
        rules: {
          productName: [
            { required: true, message: '请输入产品名称', trigger: 'blur' }
          ],
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
</style>
