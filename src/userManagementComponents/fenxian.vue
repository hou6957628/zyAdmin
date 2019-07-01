<template>
  <div>
    <h3>风险命中列表</h3>
    <div class="operationContent">
          <el-input class="searchContent"
                    placeholder="查询编号或触发名称"
                    v-model="finProduct"
                    clearable>
          </el-input>
          <el-button id="searchBtn" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
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
          label="风险命中ID编号"
          width="150">
        </el-table-column>
        <el-table-column
          prop="cusName"
          label="用户姓名"
          width="150">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="用户手机号"
          width="150">
        </el-table-column>
        <el-table-column
          prop="flowName"
          label="子流程名称"
          width="150">
        </el-table-column>
        <el-table-column
          prop="itemName"
          label="类型"
          width="150">
        </el-table-column>
        <el-table-column
          prop="memo"
          label="触发内容"
          width="150">
        </el-table-column>
        <el-table-column
          prop="value"
          label="取值"
          width="150">
        </el-table-column>
        <el-table-column
          prop="result"
          label="结果"
          width="100">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.result == 1 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.result == 0 ? '拒绝' : '通过'}}</el-tag>
          </template>
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
          label="操作"
          width="120">
          <!--<template slot-scope="scope">-->
            <!--<el-button @click="detailProduct(scope.row)" type="text" size="small">详情</el-button>-->
            <!--<el-button @click="editProduct(scope.row)" type="text" size="small">修改</el-button>-->
          <!--</template>-->
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
      //查询金融产品
      searchContent(data){
        this.getProductList(this.pageNum,this.pageSize,this.id,this.finProduct);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.id,this.finProduct);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.id,this.finProduct);
      },
      /**
       * 获取风险命中列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 用户id
       * @param data4 编号或触发名称
       */
      getProductList(data1,data2,data3,data4){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/risk/admin/flow_log/selectControlFlowLog",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            customerId:data3,
            name:data4,
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
        console.log(row.id);
        let id=row.id;
        this.$router.push({
          path: `/editFinanceProduct/${id}`,
        });
      },
    },
    mounted:function () {
      this.id=this.$route.params.id;
      this.getProductList(1,30,this.id,null);
    },
    data() {
      return {
        tableData: [],
        finProduct: '',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
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
        value7:'',
      }
    }
  }
</script>

<style scoped>
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
    margin-left:5px;
    width: 300px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
