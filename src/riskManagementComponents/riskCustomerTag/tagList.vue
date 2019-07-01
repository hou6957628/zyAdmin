<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/tagList' }">用户标签</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">创建标签<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      <el-button class="upLoadBtn" @click="tolabelList()" type="primary">用户标签分类列表<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      <el-input @click="searchProduct" class="searchContent"
        placeholder="标签名称、编号搜索"
        v-model="finProduct"
        clearable>
        <el-button id="searchBtn" @click="searchContent(finProduct)" slot="append" icon="el-icon-search">查询</el-button>
      </el-input>
        <el-select v-model="electValue" placeholder="请选择" @change="selectChange">
          <el-option
            v-for="item in electData"
            :key="item.id"
            :label="item.classifyName"
            :value="item.id">
          </el-option>
        </el-select>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="code"
          label="标签编号"
          width="150">
        </el-table-column>
        <el-table-column
          prop="tagName"
          label="标签名称"
          width="120">
        </el-table-column>
        <el-table-column
          prop="tagTypeName"
          label="标签分类"
          width="120">
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
          prop="creator"
          label="创建人员"
          width="120">
        </el-table-column>
        <el-table-column
          prop="description"
          label="标签说明"
          width="120">
        </el-table-column>
        <el-table-column
          prop="enabled"
          label="是否有效"
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
          width="220">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">配置</el-button>
            <el-button @click="deleteProduct(scope.row)" type="text" size="medium">删除</el-button>
            <el-button v-if="scope.row.enabled" @click="obtainedProduct(scope.row)" type="text" size="medium">停用</el-button>
            <el-button v-if="!scope.row.enabled" @click="obtainedCustomerTag(scope.row)" type="text" size="medium">启用</el-button>
            <el-button @click="copyProduct(scope.row)" type="text" size="medium">复制</el-button>
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
    <!--复制用户标签结构-->
    <el-dialog
      title="复制用户标签"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="标签名称:" prop="tagName" :rules="[ruleForm.tagName,{validator:checkName2, required: true, trigger:'blur'}]">
          <el-input v-model="ruleForm.tagName"></el-input>
        </el-form-item>
        <el-form-item label="标签说明:">
          <el-input v-model="ruleForm.description"></el-input>
        </el-form-item>
        <el-form-item label="是否启用:" prop="enabled">
          <el-select v-model="ruleForm.enabled" placeholder="请选择">
            <el-option
              v-for="item in electDataEnabled"
              :key="item.key"
              :label="item.Id"
              :value="item.key">
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
      //条件查询用户标签列表
      searchContent(data){
        this.getProductList(1,20,this.finProduct,this.electValue);
      },
      //每页显示多少条
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.getProductList(this.pageNum,val,this.finProduct,this.electValue);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        console.log(this.nowPageSizes);
        this.getProductList(val,this.nowPageSizes,this.finProduct,this.electValue);
      },
      //创建标签
      toAddProduct(){
        this.$router.push({
          path: `/addTag`,
        });
      },
      //跳转标签分类列表
      tolabelList(){
        this.$router.push({
          path: `/tagClassifyList`,
        });
      },
      /**
       * 用户标签列表接口
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 标签名称
       * @param data4 分类名称
       */
      getProductList(data1,data2,data3,data4){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/risk/admin/customer_tag/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            tagName: data3,
            customerTagTypeId: data4,
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
      //编辑用户标签接口
      editProduct(row){
        var id=row.id;
        this.$router.push({
          path: `/editTag/${id}`,
        });
      },
      //提示删除用户标签接口
      deleteProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/customer_tag/checkRef",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            customerTag: row.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body.list.length != 0) {
              this.$alert('该标签已被风控流程引用，请在对应风控流程中修改为其他的用户标签，再进行删除操作', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              this.$confirm('是否确认删除此用户标签?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.deleteCustomerTag(row);
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
      //确认删除用户标签接口
      deleteCustomerTag(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/customer_tag/delete",
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
            this.getProductList(1,this.nowPageSizes,this.finProduct,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //提示复制用户标签接口
      copyProduct(row){
        this.centerDialogVisible=true;
        this.copyId = row.id;
      },
      //确认复制用户标签接口
      copyCustomerTag(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method:"post",
              url:"http://"+this.baseUrl+"/risk/admin/customer_tag/copy",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params:{
                id: this.copyId,
                tagName: this.ruleForm.tagName,
                description: this.ruleForm.description,
                enabled: this.ruleForm.enabled,
              }
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.centerDialogVisible=false;
                this.getProductList(1,20,this.finProduct,null);
              }else {
                this.$message.error(res.data.msgInfo);
              }
            })
          }
        })
      },
      //提示启用停用用户标签接口
      obtainedProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/customer_tag/checkRef",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            customerTag: row.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body.list.length != 0) {
              this.$alert('该标签已被风控流程引用，请在对应风控流程中修改为其他的用户标签，再进行停用操作', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              this.$confirm('是否确认停用此用户标签?', '提示', {
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
          url:"http://"+this.baseUrl+"/risk/admin/customer_tag/delete",
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
            this.getProductList(1,20,this.finProduct,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //下拉选择
      selectChange(row){
        this.finProduct='';
        this.getProductList(1,20,null,this.electValue);
      },
      //查询用户标签分类
      getCustomerTagType(customerTagType){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/risk/admin/classification/getByType",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            type: customerTagType,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.electData=res.data.body.list;
            this.electData.unshift({id: null, classifyName: '全部'});
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //验证标签名称是否重名
      checkName2(rule, value, callback) {
        if (!value) {
          callback(new Error('请输入名称'));
        } else if (value.length < 3 | value.length > 10) {
          callback(new Error('长度在 3 到 10 个字符'));
        } else {
          axios({
            method:"GET",
            url:"http://"+this.baseUrl+"/risk/admin/customer_tag/getByName",
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
      },
    },
    mounted:function () {
      this.getProductList(1,20,null,null);
      this.getCustomerTagType(0);
    },
    data() {
      return {
        electData: [ ],
        tableData:[],
        electValue:null,
        finProduct: '',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
        centerDialogVisible:false,
        electDataEnabled: [
          {key:1,Id:'启用'},
          {key:0,Id:'不启用'},
        ],
        ruleForm:{
          copyId:'',
          tagName:'',
          description:'',
          enabled:1,
        },
        rules: {
          enabled: [
            { required: true, message: '请选择是否启用', trigger: 'change' }
          ],
        },
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
