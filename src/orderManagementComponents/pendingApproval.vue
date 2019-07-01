<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>待审批列表</el-breadcrumb-item>
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
        新老户：
        <el-select v-model="reBorrow" placeholder="请选择">
          <el-option
            v-for="item in reBorrowList"
            :key="item.id"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        主渠道：<el-input class="searchContent" placeholder="主渠道" v-model="parentChannelName" clearable></el-input>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        子渠道：<el-input class="searchContent" placeholder="子渠道" v-model="childrenChannelName" clearable></el-input>
      </el-col>
      <el-col :span="5" style="height: 55px;">
        性别：<el-select v-model="sex" placeholder="请选择">
          <el-option
            v-for="item in sexList"
            :key="item.classifyId"
            :label="item.classifyName"
            :value="item.classifyId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
      手机号：<el-input class="searchContent" placeholder="用户手机号" v-model="mobile" clearable></el-input>
      </el-col>
      <!--<el-col :span="12" style="height: 55px;">-->
        <!--订单状态：-->
        <!--<el-select v-model="status" placeholder="请选择">-->
          <!--<el-option-->
            <!--v-for="item in statusList"-->
            <!--:key="item.id"-->
            <!--:label="item.classifyName"-->
            <!--:value="item.classifyId">-->
          <!--</el-option>-->
        <!--</el-select>-->
      <!--</el-col>-->
      <el-col :span="9" style="height: 55px;">
        <template>
          申请时间：
          <el-date-picker style="margin-left: 0px"
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
      </el-col>
      <el-button type="primary" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
      <el-button v-if="hasPermissionCustom('order:audit:batchApproval')" type="primary" id="cancelBtn" @click="batchAuditOrderTip()" slot="append">批量审批</el-button>
      <!--<el-button v-if="hasPermissionCustom('order:audit:batchApprovalMachine')" type="primary" @click="batchjsOrderTip()" slot="append">批量机审</el-button>-->
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
          width="55">
        </el-table-column>
        <el-table-column
          fixed
          prop="orderId"
          label="订单ID"
          width="260">
        </el-table-column>
        <el-table-column
          fixed
          prop="name"
          label="姓名"
          min-width="80">
        </el-table-column>
        <el-table-column
          prop="gender"
          label="性别"
          :formatter="genderFormatter"
          min-width="80">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="产品"
          width="120">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="borrowingPeriod"
          label="借款期限"
          width="100">
        </el-table-column>
        <el-table-column
          prop="borrowingCapital"
          label="借款金额"
          width="100">
        </el-table-column>
        <el-table-column
          prop="status"
          label="订单状态"
          :formatter="statusFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          prop="parentChannelName"
          label="主渠道"
          width="120">
        </el-table-column>
        <el-table-column
          prop="childrenChannelName"
          label="子渠道"
          width="120">
        </el-table-column>
        <el-table-column
          prop="reBorrow"
          label="用户标识"
          :formatter="reBorrowFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="100">
          <template slot-scope="scope">
            <el-button v-if="hasPermissionCustom('order:audit:customer:find')" @click="detailProduct(scope.row)" type="text" size="medium">审核</el-button>
            <!--<el-button v-if="hasPermissionCustom('order:audit:machine')" @click="jsProduct(scope.row)" type="text" size="medium">机审</el-button>-->
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
    <!--批量审核结构-->
    <el-dialog
      title="批量审批"
      :visible.sync="centerDialogVisible1"
      width="40%"
      center>
      <center><el-button type="primary" @click="batchAuditOrder('0')" size="medium">同意</el-button>
      <el-button type="danger" @click="batchAuditOrder('1')">拒绝</el-button>
      <el-button type="info"  @click="centerDialogVisible1 = false">取消</el-button></center>
    </el-dialog>
  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
      //批量审核弹窗
      batchAuditOrderTip(){
        if (this.orderIds.length==0) {
          this.$message({
            showClose: true,
            message: '请选择至少一个订单',
            type: 'warning'
          });
        } else {
          this.centerDialogVisible1=true;
        }
      },
      //批量机审弹窗
      batchjsOrderTip(){
        if (this.orderIds.length==0) {
          this.$message({
            showClose: true,
            message: '请选择至少一个订单',
            type: 'warning'
          });
        } else {
          this.$confirm('是否确定批量机审?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            center: true
          }).then(() => {
            this.batchjsOrder();
          });
        }
      },
      //批量机审
      batchjsOrder(){
        let orderIdsStr = this.orderIds.join(',');
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/audit/machineTrial",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            orderIds: orderIdsStr,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,30,null,null,null,null,null,null,this.startDate,this.endDate,this.status);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //批量审核订单
      batchAuditOrder(status){
        const loadingObj = this.$loading({
          lock: true,
          text: '提交中...',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)',
          target: document.querySelector('.div')
        });
        let orderIdsStr = this.orderIds.join(',');
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/audit/batchAuditOrder",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            orderIds: orderIdsStr,
            status: status,
            memo: null,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            loadingObj.close();
            this.centerDialogVisible1=false;
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(this.pageNum,this.pageSize,this.productId,this.reBorrow,this.parentChannelName,this.childrenChannelName,
              this.sex,this.mobile,this.startDate,this.endDate,this.status);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
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
      //条件查询列表
      searchContent(data){
        this.getProductList(this.pageNum,this.pageSize,this.productId,this.reBorrow,this.parentChannelName,this.childrenChannelName,
          this.sex,this.mobile,this.startDate,this.endDate,this.status);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.productId,this.reBorrow,this.parentChannelName,this.childrenChannelName,
          this.sex,this.mobile,this.startDate,this.endDate,this.status);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.productId,this.reBorrow,this.parentChannelName,this.childrenChannelName,
          this.sex,this.mobile,this.startDate,this.endDate,this.status);
      },
      /**
       * 获取待审批订单列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 产品id
       * @param data4 新老户
       * @param data5 主渠道名称
       * @param data6 子渠道名称
       * @param data7 性别
       * @param data8 手机号
       * @param data9 开始时间
       * @param data10 结束时间
       * @param data11 订单状态
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9,data10,data11){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/audit/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            productId: data3,
            reBorrow: data4,
            parentChannelName: data5,
            childrenChannelName: data6,
            gender: data7,
            mobile: data8,
            startDate: data9,
            endDate: data10,
            status: data11,
            sortColumn: 'create_date',
            direction: 'desc',
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$forceUpdate();
            this.tableData=res.data.body.list;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //审核订单
      detailProduct(row){
        let id=row.customerId;
        let orderId=row.orderId;
        this.$router.push({
          path: `/orderDetail/${id}/${orderId}`,
        });
      },
      //机审
      jsProduct(row){
        this.$confirm('是否确定机审?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }).then(() => {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/order/admin/audit/machineTrial",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              orderIds: row.orderId,
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.getProductList(1,30,null,null,null,null,null,null,this.startDate,this.endDate,this.status);
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        });
      },
      //全选
      handleSelectionChange(val) {
        this.multipleSelection = val;
        let ids = []
        this.multipleSelection.map((item)=> {
          ids.push(item.orderId);
        })
        this.orderIds = ids;
      },
      //过滤性别字段
      genderFormatter(row){
        let gender = row.gender;
        if(gender == false){
          return '男'
        } else if (gender == true){
          return '女'
        } else {
          return '未知'
        }
      },
      //过滤状态字段
      statusFormatter(row){
        let status = row.status;
        if(status === 0){
          return '待机审 '
        } else if (status === 1){
          return '机器审核中'
        } else if (status === 3){
          return '人工审核'
        }
      },
      //过滤用户标识字段
      reBorrowFormatter(row){
        let reBorrow = row.reBorrow;
        if(reBorrow === false){
          return '新户'
        } else if (reBorrow === true){
          return '老户'
        } else{
          return '---'
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
          this.startTime=startTime;
          this.endTime=endTime;
        }
      },
    },
    mounted:function () {
      this.startDate=this.dateFormatCustom(new Date(new Date().getFullYear(), new Date().getMonth()-1, new Date().getDate(), 0, 0, 0));
      this.endDate=this.dateFormatCustom(new Date());
      this.value5=[this.startDate,this.endDate];
      this.getProduct();
      this.getProductList(1,30,null,null,null,null,null,null,this.startDate,this.endDate,this.status);
    },
    data() {
      return {
        multipleSelection:[],
        orderIds:[],
        productList:[],
        reBorrowList: [
          {classifyId:null,classifyName:"全部状态"},
          {classifyId:0,classifyName:"新户"},
          {classifyId:1,classifyName:"老户"},
        ],
        sexList: [
          {classifyId:null,classifyName:"全部"},
          {classifyId:false,classifyName:"男"},
          {classifyId:true,classifyName:"女"},
        ],
        statusList: [
          {classifyId:null,classifyName:"全部状态"},
          // {classifyId:0,classifyName:"待机审"},
          // {classifyId:1,classifyName:"机器审核中"},
          {classifyId:3,classifyName:"人工审核"},
        ],
        tableData:[],
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:30,
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
        productId:null,
        reBorrow:null,
        parentChannelName:null,
        childrenChannelName:null,
        sex:null,
        mobile:null,
        status:3,
        value5:'',
        startDate:null,
        endDate:null,
        centerDialogVisible1:false,
      }
    }
  }
</script>

<style scoped>
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
