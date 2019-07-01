<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/accountCenter' }">账户中心</el-breadcrumb-item>
      <el-breadcrumb-item>充值列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <div class="block">
        <el-button type="primary" icon="el-icon-plus" @click="addCashTip">充值</el-button>
        <el-date-picker style="margin-left: 25px"
          v-model="value7"
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
        </el-date-picker>&nbsp;&nbsp;&nbsp;&nbsp;
        <el-button type="primary" icon="el-icon-search" @click="searchContent">搜索</el-button>
      </div>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 100%">
        <el-table-column
          fixed
          prop="parentChannelName"
          label="渠道名称"
          width="150">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="时间"
          width="180">
        </el-table-column>
        <el-table-column
          prop="balance"
          label="余额"
          width="150">
        </el-table-column>
        <el-table-column
          prop="rechargeAmount"
          label="充值金额"
          width="150">
        </el-table-column>
        <el-table-column
          prop="description"
          label="备注"
          width="300">
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
    <!--充值结构-->
    <el-dialog title="充值" :visible.sync="dialogFormVisible" width="40%" center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="80px" class="demo-ruleForm">
        <el-form-item label="充值金额" prop="rechargeAmount">
          <el-input v-model="ruleForm.rechargeAmount" autocomplete="off" placeholder="填写充值金额"></el-input>
        </el-form-item>
        <el-form-item label="备注">
          <el-input v-model="ruleForm.description" autocomplete="off" placeholder="填写备注"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCash('ruleForm')">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    methods: {
      //条件查询
      searchContent(data){
        this.getProductList(1,30,this.channelName,this.startDate,this.endDate);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.channelName,this.startDate,this.endDate);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.channelName,this.startDate,this.endDate);
      },
      //充值列表
      getProductList(data1,data2,data3,data4,data5){
        axios({
          method:"get",
          url:"http://"+this.baseUrl+"/channel/admin/account/recharge/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            channelName: data3,
            startDate: data4,
            endDate: data5,
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
      //充值弹窗
      addCashTip(){
        this.dialogFormVisible=true;
        this.ruleForm.rechargeAmount='';
        this.ruleForm.description='';
      },
      //充值
      addCash(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method: "post",
              url: "http://" + this.baseUrl + "/channel/admin/account/recharge/add",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params: {
                accountId: this.accountId,
                rechargeAmount: this.ruleForm.rechargeAmount,
                description: this.ruleForm.description,
              }
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.dialogFormVisible = false;
                this.getProductList(1,30,this.channelName,this.startDate,this.endDate);
              } else {
                this.$message.error(res.data.msgInfo);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        })
      },
      //日期变化
      logTimeChange(){
        if(this.value7!='' && this.value7!=null){
          var startTime=this.value7[0];
          var endTime=this.value7[1];
          this.startDate=startTime;
          this.endDate=endTime;
        } else {
          this.startDate=null;
          this.endDate=null;
        }
      },
    },
    mounted:function () {
      this.channelName=this.$route.params.channelName;
      this.accountId=this.$route.params.id;
      this.getProductList(1,30,this.channelName,null,null);
    },
    data() {
      return {
        tableData: [],
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
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:30,
        channelName: '',
        accountId: '',
        value7:'',
        startDate:'',
        endDate:'',
        remark: '',
        dialogFormVisible:false,
        ruleForm: {
          rechargeAmount: '',
          description: '',
        },
        rules: {
          rechargeAmount: [
            {required: true, message: '请输入充值金额', trigger: 'blur'},
          ],
        },
      }
    }
  }
</script>

<style scoped>
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
  .dsp{
    display: inline-block;
    margin-left: 40px;
  }
  .block span{
    font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
    letter-spacing: 1px;
  }
</style>
