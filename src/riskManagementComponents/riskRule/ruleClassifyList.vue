<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/tagList' }">规则分类列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button class="upLoadBtn" @click="toTag()" type="primary">添加规则分类<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="classifyCode"
          label="分类编号"
          width="400">
        </el-table-column>
        <el-table-column
          prop="classifyName"
          label="规则分类名称"
          width="200">
        </el-table-column>
        <el-table-column
          prop="description"
          label="分类说明"
          width="200"
        >
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
          width="220">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button @click="deleteProduct(scope.row)" type="text" size="medium">删除</el-button>
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
  export default {
    methods: {
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,1,null);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,1,null);
      },
      //添加规则分类
      toTag(){
        this.$router.push({
          path: `/addRuleClassify`,
        });
      },
      /**
       * 规则分类列表接口
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 标签名称
       * @param data4 分类名称
       */
      getProductList(data1,data2,data3,data4){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/risk/admin/classification/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            classifyType: data3,
            classifyName: data4,
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
      //编辑规则分类
      editProduct(row){
        var id=row.id;
        this.$router.push({
          path: `/editRuleClassify/${id}`,
        });
      },
      //提示删除分类接口
      deleteProduct(row){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/risk/admin/classification/getRuleByClassId",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body != 0) {
              this.$alert('该规则分类已被规则引用，不可删除', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              this.$confirm('是否确认删除此规则分类?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.deleteClassification(row);
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
      //确认删除分类接口
      deleteClassification(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/classification/delete",
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
              message: '删除成功',
              type: 'success'
            });
            this.getProductList(1,10,1,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //提示启用停用分类接口
      obtainedProduct(row){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/risk/admin/classification/getRuleByClassId",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body != 0) {
              this.$alert('该规则分类已被规则引用，不可停用', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              this.$confirm('是否确认停用此规则分类?', '提示', {
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
      //确认启用停用分类接口
      obtainedCustomerTag(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/classification/delete",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
            enabled: !row.enabled,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,20,1,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //过滤类型字段
      typeFormatter(row){
        let status = row.type;
        if(status === 0){
          return '用户标签'
        }else if(status === 1){
          return '规则标签'
        }else {
          return '规则集标签'
        }
      },
    },
    mounted:function () {
      this.getProductList(1,20,1,null);
    },
    data() {
      return {
        tableData:[],
        electValue:'',
        finProduct: '',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
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
