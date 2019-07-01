<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-button v-if="this.hasPermissionCustom('user:customer:batchUnlock')" type="primary" @click="batchUnlock()" slot="append">批量解锁</el-button>
    <div class="operationContent">
      <el-col :span="6" style="height: 55px;">
        产品：
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
        锁定状态：
        <el-select v-model="locked" placeholder="请选择">
          <el-option
            v-for="item in lockedList"
            :key="item.key"
            :label="item.Id"
            :value="item.key">
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
      </template>&nbsp;&nbsp;
      <el-button id="searchBtn" type="primary" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
    </div>
    <template>
      <el-table
        :data="tableData"
        @selection-change="handleSelectionChange"
        border
        highlight-current-row
        style="width:98%">
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column
          fixed
          prop="id"
          label="用户ID"
          width="80">
        </el-table-column>
        <el-table-column
          prop="realName"
          label="姓名"
          width="100">
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
          prop="createDate"
          label="注册时间"
          width="180">
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
          prop="updateDate"
          label="最近登录应用时间"
          width="180">
        </el-table-column>
        <el-table-column
          prop="locked"
          label="是否锁定"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.locked == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.locked == true ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="isActiveApp"
          label="激活APP"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.isActiveApp == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.isActiveApp == true ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="isBlackList"
          label="是否是黑名单"
          width="110">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.isBlackList == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.isBlackList == true ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="reBorrow"
          label="用户标识"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.reBorrow == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.reBorrow == true ? '老户' : '新户'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="hasIncreaseAmount"
          label="是否提额"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.hasIncreaseAmount == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.hasIncreaseAmount == true ? '是' : '否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="totalAmount"
          label="现在额度"
          width="120">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="120">
          <template slot-scope="scope">
            <el-button v-if="hasPermissionCustom('user:customer:find')" @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
            <el-button v-if="hasPermissionCustom('user:customer:get')" @click="editProduct(scope.row)" type="text" size="medium">修改</el-button>
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
        this.getProductList(1,20,this.realName,this.mobile,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId,this.locked);
      },
      //条件查询
      searchContent(){
        this.getProductList(1,20,this.realName,this.mobile,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId,this.locked);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,this.realName,this.mobile,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId,this.locked);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.realName,this.mobile,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId,this.locked);
      },
      /**
       * 获取金融产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 用户姓名
       * @param data4 用户手机号
       * @param data5 主渠道
       * @param data6 子渠道
       * @param data7 开始时间
       * @param data8 结束时间
       * @param data9 产品id
       * @param data10 锁定状态
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8,data9,data10){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/user_center/admin/customer/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            realName: data3,
            mobile: data4,
            channelName: data5,
            subChannelName: data6,
            startDate: data7,
            endDate: data8,
            productId: data9,
            locked: data10,
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
      //修改接口
      editProduct(row){
        let id=row.id;
        this.$router.push({
          path: `/modifyUserInformation/${id}`,
        });
      },
      //详情接口
      detailProduct(row){
        let id=row.id;
        this.$router.push({
          path: `/userDetail/${id}`,
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
      },
      //批量解锁用户
      batchUnlock(){
        if (this.ids.length==0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条记录',
            type: 'warning'
          });
        } else {
          this.$confirm('是否确定解锁选定的用户?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            center: true
          }).then(() => {
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/user_center/admin/customer/batchUnlock",
              headers:{
                'Content-Type':'application/json',
                'Authorization': localStorage.token
              },
              data:JSON.stringify(this.ids),
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.multipleSelection = [];
                this.getProductList(1,20,this.realName,this.mobile,this.channelName,this.subChannelName,this.startTime,this.endTime,this.productId,this.locked);
              }else {
                this.$message.error(res.data.msgInfo);
              }
            })
          });
        }
      },
    },
    mounted:function () {
      this.getProductList(1,20,null,null,null,null,null,null,null,null);
      this.getProducts();
    },
    data() {
      return {
        tableData: [],
        productListData: [],
        productId:null,
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
        channelName:'',
        subChannelName:'',
        value7:'',
        startTime:'',
        endTime:'',
        locked:null,
        lockedList:[
          {key:null,Id:'所有状态'},
          {key:1,Id:'已锁定'},
          {key:0,Id:'未锁定'},
        ],
        ids:[],
        multipleSelection:[]
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
