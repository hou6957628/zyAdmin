<template>
  <div>
    <div class="listContent">
      <h4>基本信息</h4>
      <p>用户姓名：{{this.cusCustomer.realName}}</p>
      <p>手机号：{{this.cusCustomer.mobile}}</p>
      <p>身份证号：{{this.cusCustomer.cardNumber}}</p>
    </div>
    <div class="listContent">
      <h4>联系人</h4>
      <table >
        <template v-for="(item,index) in this.linkManList">
          <tr>
            <td>第一联系人</td><td>联系人借款关系：{{item.relation | myCurrency}}</td>
            <td>姓名：{{item.name}}</td><td>手机号：{{item.phoneNum}}</td>
          </tr>
        </template>
      </table>
    </div>
    <div class="listContentBox">
      <el-row>
        <el-col :span="24">
          <div class="">
            <h4>通话呼入分析</h4>
            <template>
              <el-table
                :data="tableDataFrom"
                border
                highlight-current-row
                style="width: 100%;margin-top: 20px;">
                <el-table-column
                  fixed
                  prop="name"
                  label="姓名"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="mobile"
                  label="号码"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="netIdenty"
                  label="互联网标识"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="typeLabel"
                  label="类别标签"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="location"
                  label="归属地"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="calledTime"
                  label="联系时间(分)"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="calledNumber"
                  label="被叫次数"
                  width="150">
                </el-table-column>
              </el-table>
            </template>
            <div class="block">
              <el-pagination class="paginationBox"
                             @size-change="handleSizeChangeFrom"
                             @current-change="handleCurrentChangeFrom"
                             :unique-opened="true"
                             :current-page="pageNum"
                             :page-sizes="pageSizes"
                             :page-size="pageSize"
                             layout="total, sizes, prev, pager, next, jumper"
                             :total="proTotal">
              </el-pagination>
            </div>
          </div>
        </el-col>
        <el-col :span="24">
          <div class="">
            <h4>通话呼出分析</h4>
            <template>
              <el-table
                :data="tableDataTo"
                border
                highlight-current-row
                style="width: 100%;margin-top: 20px;">
                <el-table-column
                  fixed
                  prop="name"
                  label="姓名"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="mobile"
                  label="号码"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="netIdenty"
                  label="互联网标识"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="typeLabel"
                  label="类别标签"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="location"
                  label="归属地"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="callTime"
                  label="联系时间(分)"
                  width="150">
                </el-table-column>
                <el-table-column
                  prop="callNumber"
                  label="主叫次数"
                  width="150">
                </el-table-column>
              </el-table>
            </template>
            <div class="block">
              <el-pagination class="paginationBox"
                             @size-change="handleSizeChangeTo"
                             @current-change="handleCurrentChangeTo"
                             :unique-opened="true"
                             :current-page="pageNum1"
                             :page-sizes="pageSizes1"
                             :page-size="pageSize1"
                             layout="total, sizes, prev, pager, next, jumper"
                             :total="proTotal1">
              </el-pagination>
            </div>
          </div>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      return {
        tableDataFrom: [],
        tableDataTo: [],
        finProduct: '',
        pageNum: 1,
        proTotal:null,
        pageSize:null,
        pageSizes:[30,50,80],
        nowPageSizes:30,
        pageNum1: 1,
        proTotal1:null,
        pageSize1:null,
        pageSizes1:[30,50,80],
        nowPageSizes1:30,
        value7:'',
        linkManList:[],
        cusCustomer:{},
      }
    },
    methods: {
      //每页显示多少条
      handleSizeChangeFrom(val) {
        this.nowPageSizes=val;
        this.getAddressFrom(this.pageNum,val,this.id,0);
      },
      //翻页
      handleCurrentChangeFrom(val) {
        this.getAddressFrom(val,this.nowPageSizes,this.id,0);
      },
      //每页显示多少条
      handleSizeChangeTo(val) {
        this.nowPageSizes1=val;
        this.getAddressTo(this.pageNum1,val,this.id,1);
      },
      //翻页
      handleCurrentChangeTo(val) {
        this.getAddressTo(val,this.nowPageSizes1,this.id,1);
      },
      //运营商通讯录比对---呼入
      getAddressFrom(data1,data2,data3,data4){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/user_center/admin/address/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            id:data3,
            type:data4
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableDataFrom=res.data.body.callInRecord.list;
            this.proTotal=res.data.body.callInRecord.total;
            this.pageSize=res.data.body.callInRecord.pageSize;
            this.pageNum=res.data.body.callInRecord.pageNum;
            this.linkManList=res.data.body.linkMan;
            this.cusCustomer=res.data.body.cusCustomer;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //运营商通讯录比对--呼出
      getAddressTo(data1,data2,data3,data4){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/user_center/admin/address/get",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            id:data3,
            type:data4
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.tableDataTo=res.data.body.calloutRecord.list;
            this.proTotal=res.data.body.calloutRecord.total;
            this.pageSize=res.data.body.calloutRecord.pageSize;
            this.pageNum=res.data.body.calloutRecord.pageNum;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getAddressFrom(1,30,this.id,0);
      this.getAddressTo(1,30,this.id,1);
    },
    //过滤器的本质 就是一个有参数有返回值的方法
    filters:{
      myCurrency:function(arg1){
        //根据业务需要，对传来的数据进行处理
        // 并返回处理后的结果
        if(arg1==0){
          var result = "父母";
          return result;
        }else if(arg1==1){
          var result = "配偶";
          return result;
        }
        else if(arg1==2){
          var result = "兄弟姐妹";
          return result;
        }
        else if(arg1==3){
          var result = "同事";
          return result;
        }
        else if(arg1==4){
          var result = "同学";
          return result;
        }
        else if(arg1==5){
          var result = "朋友";
          return result;
        }
      }
    }
  }
</script>

<style scoped>
  .listContent{
    clear: both;
    height: 140px;
    width: 90%;
    border: 1px solid #ccc;
    margin-top: 15px;
    padding: 10px;
  }
  .listContentBox{
    clear: both;
    min-height: 400px;
    width: 90%;
    border: 1px solid #ccc;
    margin-top: 15px;
    padding: 10px;
    margin-bottom: 30px;
  }
  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .content p{
    margin-top: 15px;
  }
  table,table tr th, table tr td { border:1px solid #838383; }
  table { width: 100%; min-height: 40px; line-height: 40px; text-align: left;text-indent: 5px; border-collapse: collapse;margin-top: 20px}

  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
