<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>风控命中列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-col :span="6">
        <el-input placeholder="请输入姓名" clearable v-model="cusName">
          <template slot="prepend">姓名</template>
        </el-input>
      </el-col>
      <el-col :span="6" style="margin-left: 10px">
        <el-input placeholder="请输入手机号" clearable v-model="mobile">
          <template slot="prepend">手机号</template>
        </el-input>
      </el-col>
      <el-col :span="6" style="margin-left: 10px">
        <el-input placeholder="请输入类型" clearable v-model="itemName">
          <template slot="prepend">类型</template>
        </el-input>
      </el-col>
      <el-col :span="8" style="margin-left: -20px;margin-top: 10px;margin-bottom: 10px">
        <el-date-picker style="margin-left: 20px"
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
      </el-col>
      <el-button id="searchBtn" @click="searchContent()" style="margin-left: -570px;margin-top: 49px" icon="el-icon-search" type="primary">查询</el-button>
      <el-button @click="searchContent()" icon="el-icon-download el-icon-right" type="primary">导出拒绝原因</el-button>
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
          label="ID"
          width="100">
        </el-table-column>
        <el-table-column
          prop="cusName"
          label="姓名"
          width="100">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="手机号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="flowName"
          label="子流程名称"
          width="180">
        </el-table-column>
        <el-table-column
          prop="itemName"
          label="类型"
          width="150">
        </el-table-column>
        <el-table-column
          prop="memo"
          label="触发内容"
          width="250">
        </el-table-column>
        <el-table-column
          prop="value"
          label="取值"
          width="100">
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
      //导出
      daoBtn(){
        var data  = {
          'cusName':this.cusName,
          'mobile':this.mobile,
          'startDate':this.startDate,
          'endDate':this.endDate,
          'itemName':this.itemName,
        };
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/risk/admin/flow_log/export",
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
        if (!data) {
          return
        }
        let url = window.URL.createObjectURL(new Blob([data]));
        let link = document.createElement('a');
        link.style.display = 'none';
        link.href = url;
        link.setAttribute('download', '拒绝原因.xlsx');
        document.body.appendChild(link);
        link.click()
      },
      //条件查询规则集列表
      searchContent(){
        this.getProductList(1,20,this.cusName,this.mobile,this.startDate,this.endDate,this.itemName);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,this.cusName,this.mobile,this.startDate,this.endDate,this.itemName);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.cusName,this.mobile,this.startDate,this.endDate,this.itemName);
      },
      /**
       * 用户标签列表接口
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 姓名
       * @param data4 手机号
       * @param data5 开始时间
       * @param data6 结束时间
       * @param data7 类型
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/risk/admin/flow_log/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            cusName: data3,
            mobile: data4,
            startDate: data5,
            endDate: data6,
            itemName: data7,
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
        this.getProductList(1,20,this.finProduct,this.finProduct,null);
      },
      //提示删除规则接口
      deleteProduct(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/rule_collection/checkRef",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            if (res.data.body.count != 0) {
              this.$alert('该规则集已被规则流程引用，不可删除', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              this.$confirm('是否确认删除此规则集?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.deleteCustomerTag(row);
              }).catch(() => {
                this.$message({
                  type: 'info',
                  message: '已取消删除'
                });
              });
            }
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //确认删除规则接口
      deleteCustomerTag(row){
        axios({
          method:"post",
          url:"http://"+this.baseUrl+"/risk/admin/rule_collection/deleteOrStop",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.id,
            status: 1,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '删除成功',
              type: 'success'
            });
            this.getProductList(1,20,null,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //时间筛选
      logTimeChange(){
        if(this.value7==''||this.value7==null){
          this.startDate=startTime;
          this.endDate=endTime;
        }else {
          var startTime=this.value7[0];
          var endTime=this.value7[1];
          this.startDate=startTime;
          this.endDate=endTime;
        }
      },
    },
    mounted:function () {
      this.endDate=this.dateFormatCustom(new Date());
      this.startDate=this.dateFormatCustom(new Date(new Date().getFullYear(), new Date().getMonth(), new Date().getDate()-7, 0, 0, 0));
      this.value7=[this.startDate,this.endDate];
      this.getProductList(1,20,null,null,this.value7[0],this.value7[1]);
    },
    data() {
      return {
        electData: [ ],
        tableData:[],
        electValue:null,
        cusName: '',
        mobile: '',
        itemName: '',
        value7: '',
        startDate: null,
        endDate: null,
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
        electDataEnabled: [
          {key:1,Id:'启用'},
          {key:0,Id:'不启用'},
        ],
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
