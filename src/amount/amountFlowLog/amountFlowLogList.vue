<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>额度命中列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-col :span="6">
        <el-input placeholder="请输入姓名" clearable v-model="name">
          <template slot="prepend">姓名</template>
        </el-input>
      </el-col>
      <el-col :span="6">
        <el-input placeholder="请输入手机号" clearable v-model="mobile">
          <template slot="prepend">手机号</template>
        </el-input>
      </el-col>
      <el-button id="searchBtn" @click="searchContent()" slot="append" icon="el-icon-search" type="primary">查询</el-button>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 100%">
        <el-table-column
          fixed
          prop="id"
          label="ID"
          width="90">
        </el-table-column>
        <el-table-column
          prop="name"
          label="姓名"
          width="90">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="flowName"
          label="流程名称"
          width="200">
        </el-table-column>
        <el-table-column
          prop="tagName"
          label="标签名称"
          width="240">
        </el-table-column>
        <el-table-column
          prop="memo"
          label="触发内容"
          width="300">
        </el-table-column>
        <el-table-column
          prop="result"
          label="结果"
          width="100"
          :formatter="actionFormatter">
        </el-table-column>
        <el-table-column
          prop="actionValue"
          label="取值"
          width="90">
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
        <!--<el-table-column-->
          <!--label="操作"-->
          <!--width="220">-->
          <!--<template slot-scope="scope">-->
            <!--<el-button @click="editProduct(scope.row)" type="text" size="small">编辑</el-button>-->
            <!--<el-button @click="deleteProduct(scope.row)" type="text" size="small">删除</el-button>-->
            <!--<el-button v-if="scope.row.enabled" @click="obtainedProduct(scope.row)" type="text" size="small">停用</el-button>-->
            <!--<el-button v-if="!scope.row.enabled" @click="obtainedCustomerTag(scope.row)" type="text" size="small">启用</el-button>-->
            <!--<el-button @click="copyProduct(scope.row)" type="text" size="small">复制</el-button>-->
          <!--</template>-->
        <!--</el-table-column>-->
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
      //条件查询规则集列表
      searchContent(){
        this.getProductList(1,20,this.name,this.mobile);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,this.name,this.mobile);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.name,this.mobile);
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
          url:"http://"+this.baseUrl+"/quota/admin/amountFlowLog/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            name: data3,
            mobile: data4,
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
      //过滤状态字段
      actionFormatter(row){
        let action = row.action;
        if(action === 0){
          return '提额 '
        } else if (action === 1){
          return '降额'
        } else if (action === 2){
          return '维持原额度'
        }else if (action === 3){
          return '拉黑'
        }
      },
    },
    mounted:function () {
      this.getProductList(1,20,null,null);
    },
    data() {
      return {
        tableData:[],
        name: '',
        mobile: '',
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
  el-input {
    width: 130px;
    margin-bottom: 5px;
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
