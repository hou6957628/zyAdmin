<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>消息管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <div style="margin-bottom: 20px">
        <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">创建分类&nbsp;<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      </div>
      <template>
        时间：
          <el-date-picker style="margin-left: 0px;margin-right: 15px;"
                          v-model="value5"
                          type="datetimerange"
                          align="right"
                          unlink-panels
                          range-separator="至"
                          start-placeholder="开始日期"
                          end-placeholder="结束日期"
                          :picker-options="pickerOptions2"
                          format="yyyy-MM-dd HH:mm:ss"
                          value-format="yyyy-MM-dd HH:mm:ss"
                          @change="logTimeChange()">
          </el-date-picker>
        </template>
      <el-input style="width: 350px;" class="searchContent"
                placeholder="输入名称或ID"
                v-model="finProduct"
                clearable>
        <el-button id="searchBtn" @click="searchContent(finProduct)" slot="append" icon="el-icon-search">查询</el-button>
      </el-input>
      <el-button type="primary" id="cancelBtn" @click="batchDel()" slot="append">批量删除</el-button>
    </div>
    <template>
      <el-table
        ref="multipleTable"
        :data="tableData"
        @selection-change="handleSelectionChange"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          type="selection"
          label="批量"
          width="55">
        </el-table-column>
        <el-table-column
          fixed
          prop="id"
          label="ID"
          width="80">
        </el-table-column>
        <el-table-column
          fixed
          prop="classifyName"
          label="分类名称"
          width="200">
        </el-table-column>
        <el-table-column
          prop="description"
          label="备注"
          width="300">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="创建时间"
          width="170">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="更新时间"
          width="170">
        </el-table-column>
        <el-table-column
          prop="creator"
          label="创建人"
          width="120">
        </el-table-column>
        <el-table-column
          label="操作"
          width="100">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
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
    <!--创建分类-->
    <el-dialog
      title="创建分类"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="80px" class="demo-ruleForm">
        <el-form-item label="名称:" prop="classifyName">
          <el-input v-model="ruleForm.classifyName" placeholder="请填写名称"></el-input>
        </el-form-item>
        <el-form-item label="备注:" prop="description">
          <el-input type="textarea":rows="3" v-model="ruleForm.description" placeholder="请填写备注"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="insertMessage('ruleForm')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!--编辑分类-->
    <el-dialog
      title="编辑分类"
      :visible.sync="centerDialogVisible1"
      width="40%"
      center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="100px" class="demo-ruleForm">
        <el-form-item label="名称:" prop="classifyName">
          <el-input v-model="ruleForm2.classifyName"></el-input>
        </el-form-item>
        <el-form-item label="请输入备注:" prop="description">
          <el-input type="textarea":rows="3" v-model="ruleForm2.description"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="saveMessage('ruleForm2')">保存<i class="el-icon-check el-icon--right"></i></el-button>
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
      //条件查询列表
      searchContent(data){
        this.getProductList(this.pageNum,this.nowPageSizes,this.startDate,this.endDate,data);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.startDate,this.endDate,this.finProduct);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.startDate,this.endDate,this.finProduct);
      },
      /**
       * 获取分类列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 开始时间
       * @param data4 结束时间
       * @param data5 名称和id
       */
      getProductList(data1,data2,data3,data4,data5){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_classify/find",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            startDate: data3,
            endDate: data4,
            condition: data5,
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
      //创建分类弹窗
      toAddProduct(){
        this.centerDialogVisible=true;
      },
      //创建分类
      insertMessage(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('classifyName', this.ruleForm.classifyName);
            param.append('description', this.ruleForm.description);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/message/admin/message_classify/insert",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '保存成功',
                  type: 'success'
                });
                this.centerDialogVisible=false;
                this.getProductList(1,10,null,null,null);
              } else {
                this.$message.error(res);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //编辑
      editProduct(row){
        this.centerDialogVisible1=true;
        this.ruleForm.id=row.id;
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_classify/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id:row.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm2.id=res.data.body.id;
            this.ruleForm2.classifyName=res.data.body.classifyName;
            this.ruleForm2.description=res.data.body.description;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        });
      },
      //编辑保存分类
      saveMessage(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('classifyName', this.ruleForm2.classifyName);
            param.append('description', this.ruleForm2.description);
            param.append('id', this.ruleForm2.id);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/message/admin/message_classify/update",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.centerDialogVisible1=false;
                this.getProductList(1,10,null,null,null);
              } else {
                this.$message.error(res);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //全选
      handleSelectionChange(val) {
        this.multipleSelection = val;
        let ids = []
        this.multipleSelection.map((item)=> {
          ids.push(item.id);
        })
        this.ids = ids;
      },
      //批量删除
      batchDel(){
        if (this.ids.length==0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条记录',
            type: 'warning'
          });
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/message/admin/message_classify/checkClassify",
            headers:{
              'Content-Type':'application/json',
              'Authorization': localStorage.token
            },
            data:JSON.stringify(this.ids),
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              this.$confirm('是否确定删除所选消息分类?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.deleteClassification(this.ids);
              });
            } else if (res.data.msgCd=='ZYCASH-70003') {
              this.$alert('分类使用中,不可删除', '提示', {
                confirmButtonText: '确定',
                type: 'warning',
                center: true
              })
            } else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }
      },
      //时间筛选
      logTimeChange(){
        if(this.value5==''||this.value5==null){
          this.startDate=null;
          this.endDate=null;
        }else {
          var startTime=this.value5[0];
          var endTime=this.value5[1];
          this.startDate=startTime;
          this.endDate=endTime;
        }
      },
      //单独删除分类接口
      deleteProduct(row){
        let id = [];
        id.push(row.id);
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_classify/checkClassify",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(id),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$confirm('是否确定删除此消息分类?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              this.deleteClassification(id);
            });
          }else if (res.data.msgCd=='ZYCASH-70003') {
            this.$alert('分类使用中,不可删除', '提示', {
              confirmButtonText: '确定',
              type: 'warning',
              center: true
            })
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //确认删除短信消息接口
      deleteClassification(ids){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_classify/delete",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(ids),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
            this.getProductList(1,10,null,null,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted:function () {
      this.getProductList(1,10,null,null,null);
    },
    data() {
      //分类名称不可重复
      var validateName = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请填写名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/message/admin/message_classify/check",
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
      };
      return {
        productList:[],
        multipleSelection:'',
        ids:[],
        finProduct:'',
        tableData:[],
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[10,20,30],
        nowPageSizes:10,
        pickerOptions2: {
          shortcuts: [{
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          }]
        },
        startDate:null,
        endDate:null,
        ruleForm: {
          classifyName: '',
          description: '',
        },
        rules: {
          classifyName: [
            { required: true, validator: validateName, trigger: 'blur' }
          ],
        },
        ruleForm2: {
          id:'',
          classifyName: '',
          description: '',
        },
        rules2: {
          classifyName: [
            { required: true, message: '请填写名称', trigger: 'blur' }
          ],
        },
        centerDialogVisible:false,
        centerDialogVisible1:false,
        value5:''
      }
    }
  }
</script>

<style scoped>
  .searchContent{
    width: 200px;
  }
  .el-col-8{
    height: 55px;
  }
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
    margin-right: 15px;
  }
  .operationContent .searchContent{
    margin-left:0;
    width: 200px;
    margin-right: 30px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
