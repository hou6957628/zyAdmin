<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/blackList' }">黑名单列表</el-breadcrumb-item>
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
        <template>主渠道：
          <el-input class="searchContent" placeholder="主渠道名称" v-model="channelName" clearable></el-input>
        </template>
      </el-col>
      <el-col :span="6" style="height: 55px;">
        <template>子渠道：
          <el-input class="searchContent" placeholder="子渠道名称" v-model="subChannelName" clearable></el-input>
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
        </template>
    </div>
    <div style="margin-bottom: 20px">
      <el-button style="float: left;margin-bottom: 10px" id="searchBtn" type="primary" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
      <el-button v-if="this.hasPermissionCustom('user:black:add')" style="min-width: 50px;" type="primary" id="blackBtn" @click="blackBtn()" slot="append" icon="">添加黑名单</el-button>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 98%">
        <el-table-column
          fixed
          prop="customerId"
          label="用户ID"
          width="80">
        </el-table-column>
        <el-table-column
          prop="realName"
          label="姓名"
          width="100">
        </el-table-column>
        <el-table-column
          prop="cardNumber"
          label="身份证号"
          width="180">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="应用"
          width="120">
        </el-table-column>
        <el-table-column
          prop="channelName"
          label="主渠道"
          width="120">
        </el-table-column>
        <el-table-column
          prop="subChannelName"
          label="子渠道"
          width="120">
        </el-table-column>
        <el-table-column
          prop="description"
          label="拉黑原因"
          width="200">
        </el-table-column>
        <el-table-column
          prop="isBlackList"
          label="是否是黑名单"
          width="110">
          <template slot-scope="scope">
            <el-tag
              :type="'danger'"
              disable-transitions>是</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="reBorrow"
          label="用户标识"
          :formatter="reBorrowFormatter"
          width="80">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="120">
          <template slot-scope="scope">
            <el-button v-if="hasPermissionCustom('user::black:find')" @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
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
    <!--展期还款结构-->
    <el-dialog
      title="添加黑名单"
      :visible.sync="centerDialogVisible"
      width="40%"
      center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="手机号：" prop="mobile">
          <el-input v-model="ruleForm.mobile"></el-input>
        </el-form-item>
        <el-form-item label="备注：">
          <el-input v-model="ruleForm.description"></el-input>
        </el-form-item>
        <el-button style="margin-left: 200px" type="primary" @click="addBlack('ruleForm')">拉黑</el-button>
        <el-button type="info"  @click="centerDialogVisible = false">取消</el-button>
      </el-form>
    </el-dialog>
  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
      //根据产品查询
      selectChange() {
        this.getProductList(1,30,this.realName,this.mobile,this.cardNumber,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId);
      },
      //查询金融产品
      searchContent(data){
        this.getProductList(1,30,this.realName,this.mobile,this.cardNumber,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,this.realName,this.mobile,this.cardNumber,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.realName,this.mobile,this.cardNumber,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId);
      },
      //拉黑弹窗
      blackBtn(){
        this.centerDialogVisible=true;
      },
      //拉黑
      addBlack() {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/user_center/admin/black/add",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            mobile: this.ruleForm.mobile,
            description: this.ruleForm.description,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.centerDialogVisible=false;
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,20,null,null,null,null,null,null,null,null);
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      /**
       * 获取黑名单列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 用户姓名
       * @param data4 用户手机号
       * @param data5 身份证号
       * @param data6 主渠道
       * @param data7 子渠道
       * @param data8 开始时间
       * @param data9 结束时间
       * @param data10 产品id
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9,data10){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/user_center/admin/black/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            realName: data3,
            mobile: data4,
            cardNumber: data5,
            channelName: data6,
            subChannelName: data7,
            startDate: data8,
            endDate: data9,
            productId: data10,
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
      //查询产品接口
      searchProduct(){
        this.getProductList(1,20,this.finProduct,this.finProduct);
      },
      //编辑产品接口
      editProduct(row){
        let id=row.id;
        this.$router.push({
          path: `/editFinanceProduct/${id}`,
        });
      },
      //详情接口
      detailProduct(row){
        let id=row.customerId;
        if (id == null) {
          this.$message.warning('没有详情');
        } else {
          this.$router.push({
            path: `/userDetail/${id}`,
          });
        }
      },
      //过滤用户标识字段
      reBorrowFormatter(row){
        let reBorrow = row.reBorrow;
        if(reBorrow == false){
          return '新户'
        } else if (reBorrow == true){
          return '老户'
        } else{
          return '---'
        }
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
    },
    mounted:function () {
      this.startTime=this.dateFormatCustom(new Date(new Date().getFullYear(), new Date().getMonth()-1, new Date().getDate(), 0, 0, 0));
      this.endTime=this.dateFormatCustom(new Date());
      this.value7=[this.startTime,this.endTime];
      this.getProductList(1,30,null,null,null,null,null,this.startTime,this.endTime,null);
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
        realName:'',
        mobile:'',
        cardNumber:'',
        channelName:'',
        subChannelName:'',
        value7:'',
        startTime:'',
        endTime:'',
        centerDialogVisible:false,
        ruleForm: {
          mobile:null,
          description:null,
        },
        rules: {
          mobile: [
            { required: true, message: '请填写手机号', trigger: 'blur' }
          ],
        }
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
