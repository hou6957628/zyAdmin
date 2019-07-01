<template>
  <div>
    <template>
      <h3>放款记录</h3>
      <el-table
        :data="count"
        border
        highlight-current-row
        style="width: 98%;margin-top: 20px;">
        <el-table-column
          fixed
          prop="orderId"
          label="订单ID"
          width="250">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label=" 手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="name"
          label="姓名"
          width="120">
        </el-table-column>
        <el-table-column
          prop="bankName"
          label="银行名称"
          width="120">
        </el-table-column>
        <el-table-column
          prop="bankNumber"
          label="银行卡号"
          width="200">
        </el-table-column>
        <el-table-column
          prop="tranFlowId"
          label="交易流水号"
          width="250">
        </el-table-column>
        <el-table-column
          prop="borrowingPaymentAmount"
          label="放款金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="status"
          label="状态"
          :formatter="statusFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="所属平台"
          width="100">
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
      //放款记录
      getCollection(id) {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/user_center/admin/payment/get",
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
    },
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
