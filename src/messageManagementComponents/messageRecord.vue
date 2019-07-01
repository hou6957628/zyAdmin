<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>消息配置</el-breadcrumb-item>
      <el-breadcrumb-item>日志列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-col :span="6" style="height: 55px;">
        产品：
        <el-select v-model="productId" placeholder="请选择">
          <el-option
            v-for="item in productList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        形式：
        <el-select v-model="modeId" placeholder="请选择">
          <el-option
            v-for="item in modeList"
            :key="item.id"
            :label="item.modeName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        分类：
        <el-select v-model="classifyId" placeholder="请选择">
          <el-option
            v-for="item in messageClassifyList"
            :key="item.id"
            :label="item.classifyName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-col>
      <el-input style="width: 350px;" class="searchContent"
                placeholder="输入名称或ID"
                v-model="conditionName"
                clearable>
      </el-input>
      <el-button type="primary" id="searchBtn" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
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
          label="ID"
          width="100">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="所属APP"
          width="120">
        </el-table-column>
        <el-table-column
          prop="name"
          label="消息名称"
          width="200">
        </el-table-column>
        <el-table-column
          prop="modeName"
          label="消息形式"
          width="100">
        </el-table-column>
        <el-table-column
          prop="classifyName"
          label="消息分类"
          width="180">
        </el-table-column>
        <el-table-column
          prop="description"
          label="备注"
          width="300">
        </el-table-column>
        <el-table-column
          prop="status"
          label="状态"
          width="100">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.status == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.status == true ? '成功' : '失败'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="执行时间"
          width="200">
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
      //查询所有产品
      getProduct() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getProductList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productList=res.data.body;
            this.productList.unshift({productId: null, productName: '全部产品'});
          } else if (res.data.msgCd=='ZYCASH-1005') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else if (res.data.msgCd=='SYS00001') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else {
            this.$message.error(res);
          }
        })
      },
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
      //查询所有形式
      getModeList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_mode/findList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.modeList=res.data.body;
            this.modeList.unshift({id: null, modeName: '全部形式'});
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
        this.getProductList(this.pageNum,this.pageSize,this.productId,this.modeId,this.classifyId,this.conditionName);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.productId,this.modeId,this.classifyId,this.conditionName);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.pageSize,this.productId,this.modeId,this.classifyId,this.conditionName);
      },
      /**
       * 获取日志列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 产品id
       * @param data4 形式id
       * @param data5 分类id
       * @param data6 名称或id
       */
      getProductList(data1,data2,data3,data4,data5,data6){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/messagelog/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            productId: data3,
            modeId: data4,
            classifyId: data5,
            conditionName: data6,
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
    },
    mounted:function () {
      this.getProduct();
      this.getModeList();
      this.getMessageClassifyList();
      this.getProductList(1,20,null,null,null,null);
    },
    data() {
      return {
        productList:[],
        modeList:[],
        messageClassifyList:[],
        productId:null,
        modeId:null,
        classifyId:null,
        conditionName:null,
        tableData:[],
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
