<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">创建角色<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="id"
          label="角色ID"
          width="80">
        </el-table-column>
        <el-table-column
          prop="roleName"
          label="角色名称"
          width="150">
        </el-table-column>
        <el-table-column
          prop="description"
          label="角色说明"
          width="150">
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="authorities"
          label="角色权限"
          width="500"
          :formatter="getAuto">
        </el-table-column>
        <el-table-column
          prop="enable"
          label="状态"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.enable == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enable == true ? '启用' : '停用'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="创建时间"
          width="160">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="更新时间"
          width="160">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="180">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button v-if="scope.row.enable" @click="obtainedProductTip(scope.row)" type="text" size="medium">停用</el-button>
            <el-button v-if="!scope.row.enable" @click="obtainedProduct(scope.row)" type="text" size="medium">启用</el-button>
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
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes);
      },
      //创建角色
      toAddProduct(){
        this.$router.push({
          path: `/addRole`,
        });
      },
      /**
       * 获取金融产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       */
      getProductList(data1,data2){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/role/list",
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
        var id=row.roleCode;
        this.$router.push({
          path: `/editRole/${id}`,
        });
      },
      //删除产品接口
      deleteProduct(row){
        this.$confirm('是否确定删除此角色?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }).then(() => {
          axios({
            method:"post",
            url:"http://"+this.baseUrl+"/operate/admin/role/edit",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              roleCode: row.roleCode,
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
        });
      },
      //提示停用产品接口
      obtainedProductTip(row){
        this.$confirm('是否确定停用此角色?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }).then(() => {
          this.obtainedProduct(row);
        });
      },
      //停用产品接口
      obtainedProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/operate/admin/role/edit",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            roleCode: row.roleCode,
            stopTag: !row.enable,
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
        if (row.list != null) {
          for(var i=0;i<row.list.length;i++){
            roleName.push(row.list[i].authorityName)
          }
          return roleName.join("、");
        } else {
          return roleName.join('');
        }
      },
    },
    mounted:function () {
      this.getProductList(1,20);
    },
    data() {
      return {
        tableData: [],
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
