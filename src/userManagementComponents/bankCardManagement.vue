<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/bankCardManagement' }">银行卡管理列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-col :span="6" style="height: 55px;">
        产品名称：
        <el-select v-model="productId" placeholder="请选择" @change="selectChange">
          <el-option
            v-for="item in productListData"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        <template>姓名：
          <el-input class="searchContent" placeholder="用户姓名" v-model="realName" clearable> </el-input>
        </template>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        <template>手机号：
          <el-input class="searchContent" placeholder="用户手机号" v-model="mobile" clearable></el-input>
        </template>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        <template>身份证号：
          <el-input class="searchContent" placeholder="用户身份证号" v-model="cardNumber" clearable></el-input>
        </template>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        <template>开户银行：
          <el-input class="searchContent" placeholder="开户银行" v-model="bankName" clearable></el-input>
        </template>
      </el-col>
      <template>
        时间筛选:
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
        </el-date-picker>
      </template>&nbsp;&nbsp;&nbsp;
      <el-button id="searchBtn" type="primary" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
      <el-button v-if="this.hasPermissionCustom('credit:bank:deleteBatch')" type="primary" @click="batchDelTip()" slot="append" icon="el-icon-delete">批量删除</el-button>
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
          label="银行卡ID"
          width="80">
        </el-table-column>
        <el-table-column
          prop="name"
          label="姓名"
          width="100">
        </el-table-column>
        <el-table-column
          prop="registerMobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="type==0?'储蓄卡':'信用卡'"
          label="类型"
          width="100">
          <template slot-scope="scope">
            <el-tag disable-transitions>{{scope.row.type == 0 ? '储蓄卡' : '信用卡'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="productName"
          label="应用"
          width="120">
        </el-table-column>
        <el-table-column
          prop="cardNumber"
          label="银行卡号"
          width="200">
        </el-table-column>
        <el-table-column
          prop="bankName"
          label="开户行"
          width="150">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="预留手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="purpose"
          label="是否是放款卡"
          width="110">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.purpose == 1 || scope.row.purpose == 3 || scope.row.purpose == 7? 'primary' : 'danger'"
              disable-transitions>{{scope.row.purpose == 1 || scope.row.purpose == 3 || scope.row.purpose == 7? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="purpose"
          label="是否是快捷支付"
          width="120">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.purpose == 4 ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.purpose == 4 ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="创建时间"
          width="180">
        </el-table-column>
        <el-table-column
          prop="number"
          label="身份证号"
          width="180">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="120">
          <template slot-scope="scope">
            <el-button v-if="hasPermissionCustom('credit:bank:delete')" @click="editProduct(scope.row)" type="text" size="medium">删除</el-button>
            <el-button v-if="hasPermissionCustom('credit:bank:getBankCardInfoById')" @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
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
  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
      //根据产品查询
      selectChange() {
        this.getProductList(1,30,this.realName,this.mobile,this.cardNumber,this.bankName,this.startTime,this.endTime,this.productId);
      },
      //查询金融产品
      searchContent(data){
        this.getProductList(1,30,this.realName,this.mobile,this.cardNumber,this.bankName,this.startTime,this.endTime,this.productId);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,this.realName,this.mobile,this.cardNumber,this.bankName,this.startTime,this.endTime,this.productId);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.realName,this.mobile,this.cardNumber,this.bankName,this.startTime,this.endTime,this.productId);
      },
      /**
       * 获取金融产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 用户姓名
       * @param data4 手机号
       * @param data5 身份证号
       * @param data6 开户行
       * @param data7 开始时间
       * @param data8 结束时间
       * @param data9 产品id
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/credit/admin/bank/getBankCardInfoByParams",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            name: data3,
            registerMobile: data4,
            number: data5,
            bankName: data6,
            startTime: data7,
            endTime: data8,
            productId: data9,
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
      //单独删除银行卡提示
      editProduct(row){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/isHaveUnPaymentList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            customerId:row.customerId,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body.isHaveUnPaymentList) {
              this.$alert('此用户有未还完订单，不能进行删卡操作', '提示', {
                confirmButtonText: '确定',
                type: 'warning',
                center: true
              })
            } else {
              this.$confirm('是否确认删除选择的银行卡?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.deleteBankCard(row.id);
              });
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //确认单独删除银行卡接口
      deleteBankCard(id){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/credit/admin/bank/deleteBankCardInfo",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id:id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '删除成功',
              type: 'success'
            });
            this.getProductList(1,30,null,null,null,null,null,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //批量删除银行卡提示
      batchDelTip(row){
        if (this.customerId == null) {
          this.$message({
            showClose: true,
            message: '请至少选择一条记录',
            type: 'warning'
          });
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/order/admin/borrowing/isHaveUnPaymentList",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              customerId:this.customerId,
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body.isHaveUnPaymentList) {
                this.$alert('此用户有未还完订单，不能进行删卡操作', '提示', {
                  confirmButtonText: '确定',
                  type: 'warning',
                  center: true
                })
              } else {
                this.$confirm('是否确认删除选择的银行卡?', '提示', {
                  confirmButtonText: '确定',
                  cancelButtonText: '取消',
                  type: 'warning',
                  center: true
                }).then(() => {
                  this.batchDel();
                });
              }
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }
      },
      //确认批量删除银行卡接口
      batchDel(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/credit/admin/bank/batchDeleteBankCard",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(this.ids),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '删除成功',
              type: 'success'
            });
            this.getProductList(1,30,null,null,null,null,null,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //详情接口
      detailProduct(row){
        let id=row.id;
        this.$router.push({
          path: `/bankCardDetail/${id}`,
        });
      },
      //时间筛选
      logTimeChange(){
        if(this.value7!='' && this.value7!=null){
          var startTime=this.value7[0];
          var endTime=this.value7[1];
          this.startTime=startTime;
          this.endTime=endTime;
        } else {
          this.startTime='';
          this.endTime='';
        }
      },
      //查询产品列表
      getProducts(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getProductList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productListData=res.data.body;
            this.productListData.unshift({productId: null, productName: '全部产品'});
          } else if (res.data.msgCd=='ZYCASH-1005') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else if (res.data.msgCd=='SYS00001') {
            this.$message.error('登陆信息失效，请重新登陆');
            this.$router.push({path: `/login`,});
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //全选
      handleSelectionChange(val) {
        this.multipleSelection = val;
        let ids = []
        this.multipleSelection.map((item)=> {
          ids.push(item.id);
        })
        this.ids = ids;
        if (this.multipleSelection.length == 0) {
          this.customerId = null;
        } else {
          this.customerId = this.multipleSelection[0].customerId;
        }
      },
    },
    mounted:function () {
      this.getProductList(1,30,null,null,null,null,null,null,null);
      this.getProducts();
    },
    data() {
      return {
        productListData: [],
        productId:null,
        tableData: [],
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
        realName:'',
        mobile:'',
        cardNumber:'',
        bankName:'',
        value7:'',
        startTime:'',
        endTime:'',
        ids:[],
        customerId:null,
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
    margin-left:5px;
    width: 200px;
    margin-right: 30px;
  }
  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
