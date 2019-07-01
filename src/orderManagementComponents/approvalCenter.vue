<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/approvalCenter' }">审批中心</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="listContent">
      <div class="listBox" v-for="item in productList" @click="fen(item.key)">{{item.Id}}</div>
    </div>
    <el-row style="width: 90%;text-align: center;font-size: 20px;margin-bottom: 20px">
      <el-col :span="4">
        <el-card shadow="always">
          <p>昨日到期订单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日放款金额</p>
          <p class="nnd">2000.00元</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
    </el-row>
    <div class="line">今日进件情况</div>
    <el-row style="width: 90%;text-align: center;font-size: 20px;margin-top: 20px;margin-bottom: 20px">
      <el-col :span="4">
        <el-card shadow="always">
          <p>昨日到期订单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日放款金额</p>
          <p class="nnd">2000.00元</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" style="margin-top: 20px">
        <el-card shadow="always">
          <p>昨日到期订单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1" style="margin-top: 20px">
        <el-card shadow="always">
          <p>今日放款金额</p>
          <p class="nnd">2000.00元</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1" style="margin-top: 20px">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1" style="margin-top: 20px">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1" style="margin-top: 20px">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
    </el-row>
    <div class="line"></div>
    <el-row style="width: 90%;text-align: center;font-size: 20px;margin-top: 20px">
      <el-col :span="4">
        <el-card shadow="always">
          <p>昨日到期订单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日放款金额</p>
          <p class="nnd">2000.00元</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card shadow="always">
          <p>今日到期单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
      <el-col :span="4" style="margin-top: 20px">
        <el-card shadow="always">
          <p>昨日到期订单数</p>
          <p class="nnd">2000</p>
        </el-card>
      </el-col>
    </el-row>
    </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
      //查询金融产品
      searchContent(data){
        if(data==""){
          this.getProductList(1,20,null,null);
          // this.$message.error('搜索内容不可以为空');
        }else {
          this.getProductList(1,20,data,this.finProduct);
          console.log(data);
        }
      },
      //创建金融产品
      toAddProduct(){
        this.$router.push({
          path: `/editFinanceProduct`,
        });
      },
      /**
       * 获取金融产品列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 产品名称
       * @param data4 产品编号
       */
      getProductList(data1,data2,data3,data4){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/operate/admin/Product/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            name: data3,
            id: data4,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-SUPERMARKET-200'){
            this.tableData=res.data.body.list;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //过滤类型字段
      typeFormatter(row){
        let status = row.type;
        if(status === 0){
          return '信贷产品'
        } else {
          return '分期产品'
        }
      },
    },
    mounted:function () {
      // this.finProduct=this.$route.params.name;
      // this.getProductList(1,20,null,null);
    },
    data() {
      return {
        productList:[
          {key:0,Id:'天使分单1'},
          {key:1,Id:'天使分单2'},
          {key:2,Id:'天使分单3'},
          {key:3,Id:'天使分单4'},
          {key:4,Id:'天使分单5'}
        ],
        tableData: [],
        finProduct: '',
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:20,
      }
    }
  }
</script>

<style scoped>
  .nnd{
    margin-top: 25px;
    margin-bottom: 15px;
    text-align: center;
    font-size: 20px;
    color: red;
    letter-spacing: 1px;
  }
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
  .fs-16{
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }

  .operationContent .upLoadBtn{

  }
  .operationContent .searchContent{
    margin-left: 20px;
    width: 300px;
    margin-right: 30px;
  }
  .listContent{
    clear: both;
    height: 80px;
  }
  .listContent li{
    display: inline-block;
    padding: 10px 15px;
    margin-right: 10px;
    margin-left: 5px;
    font-size: 16px;
    color: #ffffff;
    background-color: #118efe;
    border: 1px solid #0f91ff;
    border-radius: 5px;
    cursor: pointer;
  }
  .listContent li:hover{
    color: #ffffff;
    background-color: #0abcfe;
    border: 1px solid #0fbeff;
  }
  .line{
    background-color: #ccc;
    width: 90%;
    height: 1px;
  }
</style>
