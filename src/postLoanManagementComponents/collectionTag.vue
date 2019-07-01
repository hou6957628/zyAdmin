<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>催收标签</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button v-if="hasPermissionCustom('order:label:save')" class="upLoadBtn" @click="toTag()" type="primary">添加催收标签<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
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
          label="标签编号"
          width="200">
        </el-table-column>
        <el-table-column
          prop="labelContent"
          label="标签内容"
          width="400">
        </el-table-column>
        <el-table-column
          label="操作"
          width="100">
          <template slot-scope="scope">
            <el-button v-if="hasPermissionCustom('order:label:update')" @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button v-if="hasPermissionCustom('order:label:delete')" @click="deleteProduct(scope.row)" type="text" size="medium">删除</el-button>
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
    <!--添加标签结构-->
    <el-dialog
      title="添加催收标签"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="标签内容:" prop="labelContent">
          <el-input v-model="ruleForm.labelContent"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitTag('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!--编辑标签结构-->
    <el-dialog
      title="编辑催收标签"
      :visible.sync="centerDialogVisible1"
      width="40%"
      center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="100px" class="demo-ruleForm">
        <el-form-item label="标签内容:" prop="labelContent">
          <el-input v-model="ruleForm2.labelContent"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="editTag('ruleForm2')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible1 = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
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
      /**
       * 催收标签列表接口
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       */
      getProductList(data1,data2){
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
            this.tableData=res.data.body.list;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //添加催收标签
      toTag(){
        this.ruleForm.labelContent='';
        this.centerDialogVisible=true;
      },
      //添加催收标签-保存按钮
      submitTag(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method:"post",
              url:"http://"+this.baseUrl+"/order/admin/label/save",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params:{
                labelContent: this.ruleForm.labelContent,
              }
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.centerDialogVisible=false;
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.getProductList(1,30);
              }else {
                this.$message.error(res.data.msgInfo);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        })
      },
      //编辑催收标签接口
      editProduct(row){
        this.centerDialogVisible1=true;
        this.ruleForm2.id=row.id;
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/order/admin/label/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm2=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //编辑催收标签-保存按钮
      editTag(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method:"post",
              url:"http://"+this.baseUrl+"/order/admin/label/update",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params:{
                id: this.ruleForm2.id,
                labelContent: this.ruleForm2.labelContent,
              }
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.centerDialogVisible1=false;
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.getProductList(1,30);
              }else {
                this.$message.error(res.data.msgInfo);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        })
      },
      //删除催收标签接口
      deleteProduct(row){
        this.$confirm('是否确认删除此标签?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }).then(() => {
          axios({
            method:"post",
            url:"http://"+this.baseUrl+"/order/admin/label/update",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              id: row.id,
              enable: false,
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.getProductList(1,30);
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },
    },
    mounted:function () {
      this.getProductList(1,30);
    },
    data() {
      return {
        tableData:[],
        electValue:'',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
        centerDialogVisible:false,
        centerDialogVisible1:false,
        ruleForm: {
          id: '',
          labelContent: '',
        },
        rules: {
          labelContent: [
            { required: true, message: '请输入标签内容', trigger: 'blur' }
          ],
        },
        ruleForm2: {
          id: '',
          labelContent: '',
        },
        rules2: {
          labelContent: [
            { required: true, message: '请输入标签内容', trigger: 'blur' }
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
