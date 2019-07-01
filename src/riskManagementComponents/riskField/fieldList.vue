<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/fieldManagement' }">字段管理</el-breadcrumb-item>
    </el-breadcrumb>
    <template>
      <p>{{listName[0]}}</p>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="fieldName"
          label="字段名称"
          width="200">
        </el-table-column>
        <el-table-column
          prop="description"
          label="字段说明"
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
          prop="fieldType"
          label="字段类型"
          width="200">
        </el-table-column>
        <el-table-column
          prop="nullPass"
          label="空值通过"
          width="150">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.nullPass == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.nullPass == true ? '通过' : '拒绝'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="enabled"
          label="状态"
          width="150">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.enabled == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enabled == true ? '启用' : '停用'}}</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </template>
  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
      //创建金融产品
      toAddProduct(){
        this.$router.push({
          path: `/editFinanceProduct`,
        });
      },
      /**
       * 字段列表接口
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 字段名称
       */
      getProductList(data1,data2,data3){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/risk/admin/field/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            // pageNum: data1,
            // pageSize: data2,
            // fieldName: data3,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableData=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted:function () {
      this.getProductList(1,200,null);
    },
    data() {
      return {
        tableData: [ ],
        listName:["基础字段","运营商字段","白骑士字段","天机命中字段"]
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
  p{
    height: 50px;
    line-height: 50px;
    text-align: left;
    font-size: 20px;
    font-weight: bold;
  }
  .content{
    margin-bottom: 30px;
  }
</style>
