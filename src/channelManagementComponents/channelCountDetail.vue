<template>
    <div class="content">
      <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/channelCount' }">渠道统计列表</el-breadcrumb-item>
        <el-breadcrumb-item>详情</el-breadcrumb-item>
      </el-breadcrumb>
      <div class="operationContent">
        <el-col :span="9" style="height: 55px;">
          <template>
            时间：
            <el-date-picker style="margin-left: 0px"
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
        </el-col>
        <el-col :span="6" style="height: 55px;">
          产品：
          <el-select v-model="productCode" placeholder="请选择" @change="selectChange1()">
            <el-option
              v-for="item in productList"
              :key="item.productCode"
              :label="item.productName"
              :value="item.productCode">
            </el-option>
          </el-select>
        </el-col>
        <el-col :span="6" style="height: 55px;">
          主渠道：
          <el-select v-model="parentChannelCode" placeholder="请选择" @change="selectChange2()">
            <el-option
              v-for="item in parentChannelList"
              :key="item.parentChannelCode"
              :label="item.parentChannelName"
              :value="item.parentChannelCode">
            </el-option>
          </el-select>
        </el-col>
        <el-col :span="6" style="height: 55px;">
          子渠道：
          <el-select v-model="subChannelCode" placeholder="请选择">
            <el-option
              v-for="item in subChannelList"
              :key="item.subChannelCode"
              :label="item.subChannelName"
              :value="item.subChannelCode">
            </el-option>
          </el-select>
        </el-col>
        <el-button type="primary" icon="el-icon-search" @click="searchContent" style="margin-left: -800px;margin-top: 55px">搜索</el-button>
        <el-button type="primary" @click="daoBtn">导出<i class="el-icon-download el-icon--right"></i></el-button>&nbsp;&nbsp;&nbsp;&nbsp;
        消耗：<span>{{usedMoney}}</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        余额：<span>{{balance}}</span>
      </div>
      <template>
        <el-table
          :data="tableData"
          border
          highlight-current-row
          style="width: 100%">
          <el-table-column
            fixed
            prop="productName"
            label="产品名称"
            width="120">
          </el-table-column>
          <el-table-column
            prop="parentChannelName"
            label="主渠道名称"
            width="150">
          </el-table-column>
          <el-table-column
            prop="subChannelName"
            label="子渠道名称"
            width="150">
          </el-table-column>
          <el-table-column
            prop="statisticsDate"
            label="时间"
            width="160">
          </el-table-column>
          <el-table-column
            prop="uvPrice"
            label="UV单价"
            width="80">
          </el-table-column>
          <el-table-column
            prop="uvNum"
            label="UV渠道"
            width="80">
          </el-table-column>
          <el-table-column
            prop="viewNum"
            label="UV我们"
            width="80">
          </el-table-column>
          <el-table-column
            prop="cpaPrice"
            label="CPA单价"
            width="80">
          </el-table-column>
          <el-table-column
            prop="cpaNum"
            label="CPA数量"
            width="80">
          </el-table-column>
          <el-table-column
            prop="cpaFactNum"
            label="CPA实际"
            width="80">
          </el-table-column>
          <el-table-column
            prop="cpsPrice"
            label="CPS单价"
            width="80">
          </el-table-column>
          <el-table-column
            prop="cpsNum"
            label="CPS数量"
            width="80">
          </el-table-column>
          <el-table-column
            prop="cpsFactNum"
            label="CPS实际"
            width="80">
          </el-table-column>
          <el-table-column
            prop="countType"
            label="计价方式"
            :formatter="countTypeFormatter"
            width="80">
          </el-table-column>
          <el-table-column
            prop="currentCost"
            label="当日成本"
            width="80">
          </el-table-column>
          <el-table-column
            prop="remark"
            label="备注"
            width="300">
          </el-table-column>
          <el-table-column
            fixed="right"
            label="操作"
            width="100">
            <template slot-scope="scope">
              <el-button @click="editChannel(scope.row)" type="text" size="medium">编辑</el-button>
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
      //选择产品后查询产品对应的主渠道
      selectChange1(){
        this.parentChannelCode=null;
        this.subChannelCode=null;
        this.subChannelList=[];
        this.getParentChannelList(this.productCode);
      },
      //选择主渠道后查询对应的子渠道
      selectChange2(){
        this.subChannelCode=null;
        this.getSubChannelList(this.parentChannelCode);
      },
      //获取所有产品
      getFenList(){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/productManage/productMerchantInfo",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productList=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //通过产品code集合获取对应的主渠道
      getParentChannelList(data){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/channel/admin/account/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            productCodes:data
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.parentChannelList=res.data.body.list;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //通过主渠道code集合获取对应的子渠道
      getSubChannelList(data){
        let parentChannelCodes = [];
        parentChannelCodes.push(data);
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/channel/admin/channel/getSubChannel",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(parentChannelCodes),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.subChannelList=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //通过主渠道code获取账户余额
      getBalance(data){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/channel/admin/account/getByChannelCode",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            channelCode:data
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.balance=res.data.body.balance;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //通过3个code获取消耗
      getUsedMoney(data1,data2,data3,data4,data5){
        var data  = {
          'startDate':data1,
          'endDate':data2,
          'productCodes':data3,
          'parentChannelCodes':data4,
          'subChannelCodes':data5,
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/channel/admin/channel_statistics/channelConsume",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(data),
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.usedMoney=res.data.body;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //条件查询
      searchContent(){
        if (this.parentChannelCode==null) {
          this.$message({
            showClose: true,
            message: '请选择主渠道',
            type: 'warning'
          });
        } else if (this.subChannelCode==null) {
          this.$message({
            showClose: true,
            message: '请选择子渠道',
            type: 'warning'
          });
        } else {
          this.getProductList(this.pageNum,this.nowPageSizes,this.startTime,this.endTime,this.productCode,this.parentChannelCode,this.subChannelCode);
          this.getUsedMoney(null,null,this.productCode,this.parentChannelCode,this.subChannelCode);
          this.getBalance(this.parentChannelCode);
        }
      },
      //每页显示多少条
      handleSizeChange(val) {
        if (this.parentChannelCode==null) {
          this.$message({
            showClose: true,
            message: '请选择主渠道',
            type: 'warning'
          });
        } else if (this.subChannelCode==null) {
          this.$message({
            showClose: true,
            message: '请选择子渠道',
            type: 'warning'
          });
        } else {
          this.nowPageSizes=val;
          this.getProductList(this.pageNum,val,this.startTime,this.endTime,this.productCode,this.parentChannelCode,this.subChannelCode);
        }
      },
      //翻页
      handleCurrentChange(val) {
        if (this.parentChannelCode==null) {
          this.$message({
            showClose: true,
            message: '请选择主渠道',
            type: 'warning'
          });
        } else if (this.subChannelCode==null) {
          this.$message({
            showClose: true,
            message: '请选择子渠道',
            type: 'warning'
          });
        } else {
          this.getProductList(val,this.nowPageSizes,this.startTime,this.endTime,this.productCode,this.parentChannelCode,this.subChannelCode);
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
          this.startTime=null;
          this.endTime=null;
        }
      },
      //导出
      daoBtn(){
        var data  = {
          'startDate':this.startDate,
          'endDate':this.endDate,
          'productCodes':this.productCode,
          'parentChannelCodes':this.parentChannelCode,
          'subChannelCodes':this.subChannelCode,
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/channel/admin/export",
          headers:{
            "content-type":"application/octet-stream;charset=utf-8",
            "content-disposition":"attachment;filename=total.xls",
            'Authorization': localStorage.token
          },
          responseType: 'blob',
          data:JSON.stringify(data),
        }).then((res)=>{
          this.download1(res.data);
          this.$message({
            message: '导出成功',
            type: 'success'
          });
        })
      },
      // 下载文件
      download1(data) {
        console.log(data);
        if (!data) {
          return
        }
        let url = window.URL.createObjectURL(new Blob([data]));
        console.log(url + '-----');
        let link = document.createElement('a');
        link.style.display = 'none';
        link.href = url;
        link.setAttribute('download', '统计表详情.xls');
        document.body.appendChild(link);
        link.click()
      },
      /**
       * 列表查询
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 开始时间
       * @param data4 结束时间
       * @param data5 产品code集合
       * @param data6 主渠道code集合
       * @param data7 子渠道code集合
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7){
        var data  = {
          'pageNum':data1,
          'pageSize':data2,
          'startDate':data3,
          'endDate':data4,
          'productCodes':data5,
          'parentChannelCodes':data6,
          'subChannelCodes':data7,
        };
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/channel/admin/channel_statistics/list",
          headers:{
            'Content-Type':'application/json',
            'Authorization': localStorage.token
          },
          data:JSON.stringify(data),
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
      //下载文件
      download (data) {
        if (!data) {
          return
        }
        let url = "http://"+this.baseUrl+"/super/admin/productinfo/export?name="+this.value8+"&startDate="+this.startTime+"&endDate="+this.endTime+"&token="+this.token;
        let link = document.createElement('a');
        link.style.display = 'none';
        link.href = url;
        link.setAttribute('download', 'excel.xls');
        document.body.appendChild(link);
        link.click()
      },
      //详情
      detailChannel(){
        this.$router.push({
          path: `/operation/${id}/${countType}`,
        });
      },
      //编辑
      editChannel(row){
        let id = row.id;
        this.$router.push({
          path: `/editChannelStatisticsDetail/${id}`,
        });
      },
      //过滤计价方式
      countTypeFormatter(row){
        let countType = row.countType;
        if(countType === 1){
          return 'CPA '
        } else if (countType === 2){
          return 'CPS'
        } else if (countType === 3){
          return 'UV'
        } else {
          return ''
        }
      }
    },
    mounted:function () {
      this.productCode=this.$route.params.productCode;
      this.parentChannelCode=this.$route.params.parentChannelCode;
      this.subChannelCode=this.$route.params.subChannelCode;
      this.getFenList();
      this.getParentChannelList(this.productCode);
      this.getSubChannelList(this.parentChannelCode);
      this.getUsedMoney(null,null,this.productCode,this.parentChannelCode,this.subChannelCode);
      this.getBalance(this.parentChannelCode);
      this.getProductList(1,30,null,null,null,null,this.subChannelCode);
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
        value8:'',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:30,
        value7: '',
        startTime:null,
        endTime:null,
        productList:[],
        parentChannelList:[],
        subChannelList:[],
        productCode:null,
        parentChannelCode:null,
        subChannelCode:null,
        usedMoney:null,
        balance:null,
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
</style>
