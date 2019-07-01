<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/accountManagement' }">账号管理列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">添加账号<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      <el-input class="searchContent"
                placeholder="请输入姓名或手机号"
                v-model="finProduct"
                clearable>
        <el-button id="searchBtn" @click="searchContent(finProduct)" slot="append" icon="el-icon-search">查询</el-button>
      </el-input>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="id"
          label="编号"
          width="80">
        </el-table-column>
        <el-table-column
          prop="name"
          label="账号"
          width="150">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="150">
        </el-table-column>
        <el-table-column
          prop="roles"
          label="角色名称"
          :formatter="rolesFormatter"
          width="250">
        </el-table-column>
        <el-table-column
          prop="products"
          label="产品名称"
          width="250"
          :formatter="getAuto">
        </el-table-column>
        <el-table-column
          prop="enabled"
          label="状态"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.enabled == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enabled == true ? '启用' : '停用'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="150">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button v-if="scope.row.enabled" @click="obtainedProduct(scope.row)" type="text" size="medium">停用</el-button>
            <el-button v-if="!scope.row.enabled" @click="obtainedProduct(scope.row)" type="text" size="medium">启用</el-button>
            <el-button @click="deleteProduct(scope.row)" type="text" size="medium">删除</el-button>
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
  export default {
    methods: {
      //条件查询
      searchContent(data){
        this.getProductList(1,30,this.finProduct);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.finProduct);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.finProduct);
      },
      //创建角色
      toAddProduct(){
        this.$router.push({
          path: `/addAccount`,
        });
      },
      /**
       * 获取金融产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 名称编号
       */
      getProductList(data1,data2,data3){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/account/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            mobile: data3,
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
      //编辑产品接口
      editProduct(row){
        var id=row.id;
        this.$router.push({
          path: `/editAccount/${id}`,
        });
      },
      //删除产品接口
      deleteProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/product/deleteOrStop",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
            status: 1,
            enabled: row.enabled,
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
      //停用产品接口
      obtainedProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/account/deleteOrStop",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
            stopTag: !row.enabled,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,20);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //删除
      deleteProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/account/deleteOrStop",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
            deltetTag: true,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,20);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //过滤类型字段
      getAuto(row,column){
        var roleName=[];
        for(var i=0;i<row.products.length;i++){
          roleName.push(row.products[i].productName)
        }
        return roleName.join("、");
      },
      //过滤角色字段
      rolesFormatter(row){
        var roleName=[];
        for(var i=0;i<row.roles.length;i++){
          roleName.push(row.roles[i].roleName)
        }
        return roleName.join("、");
      },
    },
    mounted:function () {
      this.getProductList(1,20,null);
    },
    data() {
      return {
        tableData: [],
        finProduct: '',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[10,20,30],
        nowPageSizes:20,
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
