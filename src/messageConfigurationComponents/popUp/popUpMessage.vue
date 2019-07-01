<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>消息管理</el-breadcrumb-item>
      <el-breadcrumb-item>弹窗消息</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <div style="margin-bottom: 20px">
        <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">创建弹窗&nbsp;<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
        <el-button class="upLoadBtn" @click="toMessageClassify()" type="primary">分类列表&nbsp;<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      </div>
      类型：<el-select v-model="classifyId" placeholder="请选择">
        <el-option
          v-for="item in messageClassifyList"
          :key="item.id"
          :label="item.classifyName"
          :value="item.id">
        </el-option>
      </el-select>&nbsp;&nbsp;
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
      <el-input style="width: 350px;" class="searchContent"
                placeholder="输入名称或ID"
                v-model="condition"
                clearable>
        <el-button id="searchBtn" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
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
          prop="productName"
          label="APP"
          width="110">
        </el-table-column>
        <el-table-column
          fixed
          prop="name"
          label="名称"
          width="160">
        </el-table-column>
        <el-table-column
          prop="classifyName"
          label="分类"
          width="100">
        </el-table-column>
        <el-table-column
          prop="positionName"
          label="弹出位置"
          width="150">
        </el-table-column>
        <el-table-column
          prop="popupCount"
          label="次数"
          width="80">
        </el-table-column>
        <el-table-column
          prop="content"
          label="文字内容"
          width="250">
        </el-table-column>
        <el-table-column
          prop="description"
          label="备注"
          width="200">
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
          fixed="right"
          label="操作"
          width="210">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button @click="deleteProduct(scope.row)" type="text" size="medium">删除</el-button>
            <el-button @click="copeProductTip(scope.row)" type="text" size="medium">复制</el-button>
            <el-button @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
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
    <!--复制弹窗消息-->
    <el-dialog
      title="复制弹窗消息"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="80px" class="demo-ruleForm">
        <el-form-item label="名称：" prop="name">
          <el-input v-model="ruleForm2.name" placeholder="请输入名称"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="copeProduct('ruleForm2')">保存<i class="el-icon-check el-icon--right"></i></el-button>
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
      //查询所有分类
      getMessageClassifyList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_classify/findList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.messageClassifyList=res.data.body;
            this.messageClassifyList.unshift({id: null, classifyName: '全部分类'});
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //条件查询列表
      searchContent(data){
        this.getProductList(1,10,this.startDate,this.endDate,this.condition,this.classifyId,this.modeCode);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.startDate,this.endDate,this.condition,this.classifyId,this.modeCode);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.startDate,this.endDate,this.condition,this.classifyId,this.modeCode);
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
            url:"http://"+this.baseUrl+"/message/admin/message/checkMessage",
            headers:{
              'Content-Type':'application/json',
              'Authorization': localStorage.token
            },
            data:JSON.stringify(this.ids),
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              this.$confirm('是否确定删除所选消息?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.deleteClassification(this.ids);
              });
            }else if (res.data.msgCd=='ZYCASH-70004') {
              this.$alert('消息使用中,不可删除', '提示', {
                confirmButtonText: '确定',
                type: 'warning',
                center: true
              })
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }
      },
      /**
       * 获取消息列表查询
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 开始时间
       * @param data4 结束时间
       * @param data5 名称和id
       * @param data6 分类id
       * @param data7 消息类型 1-短信消息 2-提醒消息 3-弹窗消息 4-推送消息
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message/find",
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
            classifyId: data6,
            modeCode: data7,
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
      //创建弹窗
      toAddProduct(){
        this.$router.push({
          path: `/createPopUpMessage`,
        });
      },
      //详情
      detailProduct(row){
        let id = row.id;
        this.$router.push({
          path: `/detailPopUpMessage/${id}`,
        });
      },
      //编辑短信
      editProduct(row){
        let id = row.id;
        this.$router.push({
          path: `/editPopUpMessage/${id}`,
        });
      },
      //复制短信
      copeProductTip(row){
        this.copyId = row.id;
        this.centerDialogVisible=true;
        this.ruleForm2.name='';
      },
      //分类列表
      toMessageClassify(){
        this.$router.push({
          path: `/messageClassify`,
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
      //提示删除分类接口
      deleteProduct(row){
        let id = [];
        id.push(row.id);
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message/checkMessage",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(id),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$confirm('是否确定删除所选消息?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
              center: true
            }).then(() => {
              this.deleteClassification(id);
            });
          }else if (res.data.msgCd=='ZYCASH-70004') {
            this.$alert('消息使用中,不可删除', '提示', {
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
          url:"http://"+this.baseUrl+"/message/admin/message/delete",
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
            this.getProductList(1,10,null,null,null,null,this.modeCode);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //复制接口
      copeProduct(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('id', this.copyId);
            param.append('name', this.ruleForm2.name);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/message/admin/message/copy",
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
                this.centerDialogVisible=false;
                this.getProductList(1,10,null,null,null,null,this.modeCode);
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
    },
    mounted:function () {
      this.getMessageClassifyList();
      this.getProductList(1,10,null,null,null,null,this.modeCode);
    },
    data() {
      return {
        productList:[],
        copyId:'',
        multipleSelection:'',
        ids:[],
        messageClassifyList:[],
        classifyId:null,
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
        value5:'',
        startDate:null,
        endDate:null,
        condition:null,
        modeCode:'popup_message',
        centerDialogVisible:false,
        ruleForm2: {
          name: '',
        },
        rules2: {
          name: [
            { required: true, message: '请输入名称', trigger: 'blur' }
          ],
        },
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
    margin-left:0px;
    width: 200px;
    margin-right: 30px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
