<template>
  <div>
    <template>
      <h3>还款记录</h3>
      <el-table
        :data="count"
        border
        highlight-current-row
        style="width: 98%;margin-top: 20px;">
        <el-table-column
          fixed
          prop="id"
          label="ID"
          width="110">
        </el-table-column>
        <el-table-column
          fixed
          prop="memo"
          label="还款方式"
          :formatter="methodFormatter"
          min-width="150">
        </el-table-column>
        <el-table-column
          prop="gender"
          label="还款通道"
          :formatter="genderFormatter"
          min-width="120">
        </el-table-column>
        <el-table-column
          prop="status"
          label="状态"
          :formatter="statusFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          prop="name"
          label="姓名"
          width="120">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="tranFlowId"
          label="交易流水号"
          width="300">
        </el-table-column>
        <el-table-column
          prop="borrowingCapital"
          label="合同金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="lagInterest"
          label="逾期费用"
          width="100">
        </el-table-column>
        <el-table-column
          prop="amount"
          label="总费用"
          width="100">
        </el-table-column>
        <el-table-column
          prop="discountAmount"
          label="已减免金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="paymentAmount"
          label="实际还款金额"
          width="110">
        </el-table-column>
        <el-table-column
          prop="partial"
          label="是否部分还款"
          width="100">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.method == 4 || scope.row.method == 7 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.method == 4 || scope.row.method == 7 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="partialRepayment"
          label="部分还款应还金额"
          :formatter="amountFormatter"
          width="100">
        </el-table-column>
        <el-table-column
          prop="defer"
          label="是否展期"
          width="100">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.method == 2 || scope.row.method == 3 || scope.row.method == 6 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.method == 2 || scope.row.method == 3 || scope.row.method == 6 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="repaymentDefer"
          label="展期还款金额"
          :formatter="amountFormatter2"
          width="110">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="收款账户"
          width="110">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="收款平台"
          width="100">
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
      </el-table>
    </template>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        count: [],
      };
    },
    methods: {
      //查询还款记录
      getCollection(id) {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/user_center/admin/repayment/get",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.count = res.data.body;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //过滤还款方式字段
      methodFormatter(row){
        let method = row.method;
        if(method == 0){
          return '线下平账'
        } else if (method == 1){
          return '单独扣款'
        } else if (method == 2){
          return '线下展期平账'
        } else if (method == 3){
          return '线上展期扣款'
        } else if (method == 4){
          return '线下部分还款'
        } else if (method == 5){
          return '主动还款'
        } else if (method == 6){
          return 'APP线上展期'
        } else if (method == 7){
          return 'APP线上部分还款'
        } else if (method == 8){
          return '批扣'
        }
      },
      //过滤还款通道字段
      genderFormatter(row){
        return '合利宝'
      },
      //过滤部分还款应还金额
      amountFormatter(row){
        let method = row.method;
        if(method == 4 || method == 7) {
          return row.amount;
        }
      },
      //过滤展期应还金额
      amountFormatter2(row){
        let method = row.method;
        if (method == 2 || method == 3|| method == 6) {
          return row.amount;
        }
      },
      //过滤状态字段
      statusFormatter(row){
        let status = row.status;
        if(status === 0){
          return '订单生成 '
        } else if (status === 1){
          return '付款交易进行中'
        } else if (status === 2){
          return '已完成'
        } else if (status === 3){
          return '已取消'
        } else if (status === 4){
          return '失败'
        }
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getCollection(this.id);
    }
  }
</script>

<style scoped>
  .listBox{
    display: inline-block;
    float: left;
    padding: 10px 20px;
    margin: 0 5px;
    font-size: 17px;
    color: #000;
    background-color: #dedede;
    border: 1px solid #b0b0b0;
    border-radius: 10px;
    cursor: pointer;
  }
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }
  .listContent{
    clear: both;
    height: 80px;
  }

  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
</style>
