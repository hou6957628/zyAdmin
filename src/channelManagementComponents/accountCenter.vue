<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>账户中心</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button type="primary" icon="el-icon-plus" @click="addProductTip">新增账户</el-button>
      <el-input style="width: 300px;margin-left: 20px" placeholder="输入渠道名称" v-model="channelName" clearable>
        <el-button slot="append" icon="el-icon-search" @click="searchContent"></el-button>
      </el-input>
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
          label="渠道编号"
          width="100">
        </el-table-column>
        <el-table-column
          prop="parentChannelName"
          label="渠道名称"
          width="150">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="对应产品"
          width="150">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="时间"
          width="180">
        </el-table-column>
        <el-table-column
          prop="consumptionCount"
          label="总消耗"
          width="120">
        </el-table-column>
        <el-table-column
          prop="balance"
          label="余额"
          width="120">
        </el-table-column>
        <el-table-column
          prop="description"
          label="备注"
          width="300">
        </el-table-column>
        <el-table-column
          label="操作"
          width="100">
          <template slot-scope="scope">
            <el-button @click="editAccountTip(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button @click="recharge(scope.row)" type="text" size="medium">充值</el-button>
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
    <!--新增账户结构-->
    <el-dialog title="新增账户" :visible.sync="dialogFormVisible" width="40%" center>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="名称：" prop="parentChannelName">
          <el-input v-model="ruleForm.parentChannelName" autocomplete="off" placeholder="请填写名称"></el-input>
        </el-form-item>
        <el-form-item label="账号：" prop="account">
          <el-input v-model="ruleForm.account" autocomplete="off" placeholder="请填写账号"></el-input>
        </el-form-item>
        <el-form-item label="密码：" prop="passwd">
          <el-input v-model="ruleForm.passwd" autocomplete="off" placeholder="请填写密码"></el-input>
        </el-form-item>
        <el-form-item label="选择应用：" prop="productId">
          <el-select v-model="ruleForm.productId" placeholder="请选择应用" @change="selectChange1($event,productList)">
            <el-option
              v-for="item in productList"
              :key="item.productId"
              :label="item.productName"
              :value="item.productId">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="备注：">
          <el-input v-model="ruleForm.description" autocomplete="off" placeholder="请填写备注"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addProduct('ruleForm')">确 定</el-button>
      </div>
    </el-dialog>
    <!--编辑账户结构-->
    <el-dialog title="编辑账户" :visible.sync="dialogFormVisible1" width="40%" center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="80px" class="demo-ruleForm">
        <el-form-item label="账号：" prop="account">
          <el-input v-model="ruleForm2.account" autocomplete="off" placeholder="请填写账号"></el-input>
        </el-form-item>
        <el-form-item label="密码：" prop="passwd">
          <el-input v-model="ruleForm2.passwd" autocomplete="off" placeholder="请填写密码"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible1 = false">取 消</el-button>
        <el-button type="primary" @click="editAccount('ruleForm2')">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    methods: {
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
      //条件查询
      searchContent(){
        this.getProductList(1,30,this.channelName);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.channelName);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.channelName);
      },
      //所属应用下拉选择
      selectChange1(vId,list){
        let obj1 = {};
        obj1 = list.find((item)=>{
          return item.productId ===vId;
        });
        this.ruleForm.productName=obj1.productName;
        this.ruleForm.productCode=obj1.productCode;
      },
      /**
       * 查询账户列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 渠道名称
       */
      getProductList(data1,data2,data3){
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/channel/admin/account/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            channelName: data3,
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
      //添加账户弹窗
      addProductTip(){
        this.dialogFormVisible=true;
        this.ruleForm.parentChannelName='',
        this.ruleForm.account='',
        this.ruleForm.passwd='',
        this.ruleForm.productId='',
        this.ruleForm.productName='',
        this.ruleForm.productCode='',
        this.ruleForm.description=''
      },
      //添加账户
      addProduct(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method: "post",
              url: "http://" + this.baseUrl + "/channel/admin/account/add",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params: {
                parentChannelName: this.ruleForm.parentChannelName,
                accountName: this.ruleForm.parentChannelName,
                account: this.ruleForm.account,
                passwd: this.ruleForm.passwd,
                productId: this.ruleForm.productId,
                productName: this.ruleForm.productName,
                productCode: this.ruleForm.productCode,
                description: this.ruleForm.description,
              }
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '添加成功',
                  type: 'success'
                });
                this.dialogFormVisible = false;
                this.getProductList(1,30,null);
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
      //编辑弹窗
      editAccountTip(row){
        this.copyId=row.id;
        this.dialogFormVisible1=true;
        axios({
          method:"GET",
          url:"http://"+this.baseUrl+"/channel/admin/account/getByChannelCode",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            channelCode: row.parentChannelCode,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm2=res.data.body;
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //编辑保存
      editAccount(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            axios({
              method: "post",
              url: "http://" + this.baseUrl + "/channel/admin/account/updatePassWd",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              params: {
                id: this.ruleForm2.id,
                account: this.ruleForm2.account,
                passwd: this.ruleForm2.passwd,
              }
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.dialogFormVisible1 = false;
                this.getProductList(1,30,null);
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
      //充值
      recharge(row){
        let channelName = row.parentChannelName;
        let id = row.id;
        this.$router.push({path: `/rechargeCenter/${channelName}/${id}`});
      }
    },
    mounted:function () {
      this.getFenList();
      this.getProductList(1,30,null);
    },
    data() {
      //账户名称不能重复
      var validateName1 = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入账户名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/channel/admin/account/checkRepetition",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              parentChannelName: value
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body.parentChannelName == 'false') {
                callback(new Error('名称重复'));
              } else {
                callback();
              }
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }
      };
      //账号名称不能重复
      var validateName2 = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入账号名称'));
        } else {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/channel/admin/account/checkRepetition",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params:{
              accout: value
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              if (res.data.body.accout == 'false') {
                callback(new Error('名称重复'));
              } else {
                callback();
              }
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        }
      };
      return {
        tableData: [],
        productList: [],
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20,30,50],
        nowPageSizes:30,
        channelName: '',
        remark: '',
        dialogFormVisible:false,
        dialogFormVisible1:false,
        ruleForm: {
          parentChannelName: '',
          account: '',
          passwd: '',
          productId: '',
          productName: '',
          productCode: '',
          description: '',
        },
        rules: {
          parentChannelName: [
            {required: true, validator: validateName1, trigger: 'blur'},
          ],
          account: [
            {required: true, validator: validateName2, trigger: 'blur'},
          ],
          passwd: [
            {required: true, message: '请输入账户密码', trigger: 'blur'},
          ],
          productId: [
            {required: true, message: '请选择应用', trigger: 'blur'},
          ],
        },
        ruleForm2: {
          id: '',
          account: '',
          passwd: '',
        },
        rules2: {
          account: [
            {required: true, message: '请输入账户账号', trigger: 'blur'},
          ],
          passwd: [
            {required: true, message: '请输入账户密码', trigger: 'blur'},
          ],
        },
        copyId:'',
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
