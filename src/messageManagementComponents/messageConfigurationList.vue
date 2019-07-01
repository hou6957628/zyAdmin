<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>消息配置</el-breadcrumb-item>
      <el-breadcrumb-item>任务列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <div style="margin-bottom: 15px">
        <el-dropdown trigger="click" @command="handleCommand">
          <el-button type="primary">
            创建任务<i class="el-icon-arrow-down el-icon--right"></i>
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="a">触发类</el-dropdown-item>
            <el-dropdown-item command="b">通知类</el-dropdown-item>
            <el-dropdown-item command="c">营销类</el-dropdown-item>
            <el-dropdown-item command="d">技术人员</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>&nbsp;&nbsp;&nbsp;&nbsp;
      <el-button class="upLoadBtn" @click="toMessageClassify()" type="primary">日志列表&nbsp;<i class="el-icon-upload el-icon-circle-plus"></i></el-button>
      <el-button type="primary" id="cancelBtn" @click="batchDel()" slot="append">批量删除</el-button>
      <el-button type="primary" id="cancelBtn1" @click="batchStop()" slot="append">批量停用</el-button>
      <el-button type="primary" id="cancelBtn2" @click="batchExe()" slot="append">批量执行</el-button>
      </div>
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
        形式：
        <el-select v-model="modeId" placeholder="请选择">
          <el-option
            v-for="item in modeList"
            :key="item.id"
            :label="item.modeName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="12" style="height: 55px;">
        分类：
        <el-select v-model="classifyId" placeholder="请选择">
          <el-option
            v-for="item in messageClassifyList"
            :key="item.id"
            :label="item.classifyName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-col>
      <template>
        时间：
        <el-date-picker style="margin-left: 0px;margin-right: 15px;"
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
      <el-input style="width: 350px;" class="searchContent"
                placeholder="请输入消息名称，bean名称，ID"
                v-model="conditionName"
                clearable>
      </el-input>
      <el-button type="primary" id="searchBtn" @click="searchContent()" slot="append" icon="el-icon-search">查询</el-button>
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
          prop="id"
          label="ID"
          width="80">
        </el-table-column>
        <el-table-column
          prop="productName"
          label="所属产品"
          width="120">
        </el-table-column>
        <el-table-column
          prop="messageName"
          label="消息名称"
          width="150">
        </el-table-column>
        <el-table-column
          prop="list"
          label="消息形式"
          :formatter="listFormatter"
          width="150">
        </el-table-column>
        <el-table-column
          prop="classifyName"
          label="消息分类"
          width="150">
        </el-table-column>
        <el-table-column
          prop="description"
          label="备注"
          width="300">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="创建时间"
          width="180">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="更新时间"
          width="180">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="发送时间"
          :formatter="sendTimeFormatter"
          width="240">
        </el-table-column>
        <el-table-column
          prop="enabled"
          label="状态"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.enabled == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enabled == true ? '使用中' : '已停用'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="type"
          label="创建方式"
          :formatter="typeFormatter"
          width="120">
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="280">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button @click="deleteProduct(scope.row)" type="text" size="medium">删除</el-button>
            <el-button @click="copeProductTip(scope.row)" type="text" size="medium">复制</el-button>
            <el-button @click="stopProduct(scope.row)" v-if="scope.row.enabled" type="text" size="medium">停用</el-button>
            <el-button @click="startProduct(scope.row)" v-if="!scope.row.enabled" type="text" size="medium">启用</el-button>
            <el-button @click="exeProduct(scope.row)" type="text" size="medium">执行</el-button>
            <el-button @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
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
    <!--复制结构-->
    <el-dialog
      title="复制任务"
      :visible.sync="centerDialogVisible1"
      width="40%"
      center>
      <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-width="80px" class="demo-ruleForm">
        <el-form-item label="产品：" prop="productId">
          <el-select style="width: 320px" v-model="ruleForm2.productId" value-key="id" placeholder="请选择" @change="selectChange1($event,productList1)">
            <el-option
              v-for="item in productList1"
              :key="item.productId"
              :label="item.productName"
              :value="item.productId">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="用户：" prop="conditionName">
          <el-input v-model="ruleForm2.conditionName" disabled style="width: 320px"></el-input>
          <el-button style="position: absolute;right:35px;top:1px;" @click="addUser()" type="primary" plain>添加用户</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="copeProduct('ruleForm2')">保存<i class="el-icon-check el-icon--right"></i></el-button>
          <el-button type="info"  @click="centerDialogVisible1 = false">取消<i class="el-icon-close el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!--触发条件表结构-->
    <el-dialog
      title="触发条件表"
      :visible.sync="centerDialogVisible2"
      width="60%"
      center>
      <el-radio-group v-for="(item,index) in conditionList" :key="index" style="text-align: center" v-model="ruleForm2.conditionId" size="medium">
        <el-radio style="margin-bottom: 20px;width: 160px;margin-right: 10px" :dataType="item.id"
                  border :label="item.id" @change="changeHandler(item.conditionName)">{{item.conditionName}}</el-radio>
      </el-radio-group>
    </el-dialog>
  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    methods: {
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
      //查询所有产品
      getProduct1() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/order/admin/borrowing/getProductList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.productList1=res.data.body;
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
      //查询所有分类
      getMessageClassifyList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_classify/findList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.messageClassifyList=res.data.body;
            this.messageClassifyList.unshift({id: null, classifyName: '全部分类'});
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //查询所有形式
      getModeList() {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/message_mode/findList",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.modeList=res.data.body;
            this.modeList.unshift({id: null, modeName: '全部形式'});
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //条件查询列表
      searchContent(data){
        this.getProductList(this.pageNum,this.pageSize,this.productId,this.modeId,this.classifyId,this.startTime,this.endTime,this.conditionName);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.nowPageSizes=val;
        this.getProductList(this.pageNum,val,this.productId,this.modeId,this.classifyId,this.startTime,this.endTime,this.conditionName);
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.productId,this.modeId,this.classifyId,this.startTime,this.endTime,this.conditionName);
      },
      /**
       * 获取待放款订单列表
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 产品id
       * @param data4 消息形式id
       * @param data5 分类id
       * @param data6 开始时间
       * @param data7 结束时间
       * @param data8 消息名称、bean名称、ID
       */
      getProductList(data1,data2,data3,data4,data5,data6,data7,data8){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/task/list",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum:data1,
            pageSize:data2,
            productId: data3,
            modeId: data4,
            classifyId: data5,
            startTime: data6,
            endTime: data7,
            conditionName: data8,
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
      //创建任务
      handleCommand(command) {
        if (command == 'a') {
          this.$router.push({
            path: `/triggerMessage`,
          });
        } else if (command == 'b') {
          this.$router.push({
            path: `/noticeMessage`,
          });
        } else if (command == 'c') {
          this.$router.push({
            path: `/marketingMessage`,
          });
        } else if (command == 'd') {
          this.$router.push({
            path: `/techMessage`,
          });
        }
      },
      //日志列表
      toMessageClassify(){
        this.$router.push({
          path: `/messageRecord`,
        });
      },
      //编辑
      editProduct(row){
        let id = row.id;
        if (row.type ==0) {
          this.$router.push({path: `/editNoticeMessage/${id}`});
        } else if (row.type ==1) {
          this.$router.push({path: `/editTriggerMessage/${id}`});
        } else if (row.type ==2) {
          this.$router.push({path: `/editMarketingMessage/${id}`});
        } else if (row.type ==3) {
          this.$router.push({path: `/editTechMessage/${id}`});
        }
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
      //时间筛选
      logTimeChange(){
        if(this.value5==''||this.value5==null){
          this.startTime=null;
          this.endTime=null;
        }else {
          var startTime=this.value5[0];
          var endTime=this.value5[1];
          this.startTime=startTime;
          this.endTime=endTime;
        }
      },
      //批量删除任务接口
      batchDel(){
        if (this.ids.length == 0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条记录',
            type: 'warning'
          });
        } else {
          this.$confirm('是否确认删除所选任务?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            center: true
          }).then(() => {
            var _this=this;
            this.ids.forEach(function (item,index){
              _this.taskList.push({
                id:item,status:true
              });
            });
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/message/admin/batchDeleteOrStop",
              headers:{
                'Content-Type':'application/json',
                'Authorization': localStorage.token
              },
              data:JSON.stringify(this.taskList),
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.multipleSelection = null;
                this.getProductList(1,30,null,null,null,null,null,null);
              }else {
                this.$message.error(res.data.msgInfo);
              }
            })
          })
        }
      },
      //批量停用任务接口
      batchStop(){
        if (this.ids.length == 0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条记录',
            type: 'warning'
          });
        } else {
          this.$confirm('是否确认停用所选任务?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            center: true
          }).then(() => {
            var _this=this;
            this.ids.forEach(function (item,index){
              _this.taskList.push({
                id:item,enabled:false
              });
            });
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/message/admin/batchDeleteOrStop",
              headers:{
                'Content-Type':'application/json',
                'Authorization': localStorage.token
              },
              data:JSON.stringify(this.taskList),
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.multipleSelection = null;
                this.getProductList(1,30,null,null,null,null,null,null);
              }else {
                this.$message.error(res.data.msgInfo);
              }
            })
          })
        }
      },
      //单个删除任务接口
      deleteProduct(row){
        this.$confirm('是否确认删除所选任务?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }).then(() => {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/message/admin/delteOrStop/task",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params: {
              id: row.id,
              status: true,
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.getProductList(1,30,null,null,null,null,null,null);
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        })
      },
      //单个停用任务接口
      stopProduct(row){
        this.$confirm('是否确认停用所选任务?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }).then(() => {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/message/admin/delteOrStop/task",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params: {
              id: row.id,
              enabled: false,
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.getProductList(1,30,null,null,null,null,null,null);
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        })
      },
      //单个启用任务接口
      startProduct(row){
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/delteOrStop/task",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: row.id,
            enabled: true,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1,30,null,null,null,null,null,null);
          }else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //过滤创建方式字段
      typeFormatter(row){
        let type = row.type;
        if (type == 0) {
          return '通知类';
        } else if (type == 1) {
          return '触发类';
        } else if (type == 2) {
          return '营销类';
        } else if (type == 3) {
          return '技术人员';
        }
      },
      //过滤消息形式字段
      listFormatter(row){
        let list = row.list;
        let classFy = [];
        list.forEach(function(item,index){
          if (index == 0) {
            classFy.push(item.modeName);
          } else {
            if (classFy.indexOf(item.modeName) == -1) {
              classFy.push(item.modeName);
            }
          }
        })
        return classFy.join('，');
      },
      //过滤发送时间字段
      sendTimeFormatter(row){
        let type = row.type;
        if (type == 1) {
          return '触发类无发送时间';
        } else {
          let setTime = row.setTime;
          let specificTime = row.specificTime;
          let timeQuantum = row.timeQuantum;
          let days = row.days;
          let date = row.date;
          if (setTime == 0) {
            return '每一天：' + specificTime;
          } else if (setTime == 1) {
            return '时间段：' + timeQuantum + ' ' + specificTime;
          } else if (setTime == 2) {
            return '某天：' + date + ' ' + specificTime;
          } else if (setTime == 3) {
            return '每隔多少天：' + days + '天 '+ specificTime;
          } else if (setTime == 4) {
            return '立即执行';
          }
        }
      },
      //批量执行任务接口
      batchExe(){
        if (this.ids.length == 0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条记录',
            type: 'warning'
          });
        } else {
          this.$confirm('是否确认执行所选任务?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            center: true
          }).then(() => {
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/message/admin/messageBatchDeal",
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
                this.multipleSelection = null;
                this.getProductList(1,30,null,null,null,null,null,null);
              }else {
                this.$message.error(res.data.msgInfo);
              }
            })
          })
        }
      },
      //单个执行任务接口
      exeProduct(row){
        this.$confirm('是否确认执行所选任务?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }).then(() => {
          axios({
            method:"POST",
            url:"http://"+this.baseUrl+"/message/admin/messageDeal",
            headers:{
              'Content-Type':'application/x-www-form-urlencoded',
              'Authorization': localStorage.token
            },
            params: {
              taskId: row.id,
            }
          }).then((res)=>{
            if(res.data.msgCd=='ZYCASH-200'){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.multipleSelection = null;
              this.getProductList(1,30,null,null,null,null,null,null);
            }else {
              this.$message.error(res.data.msgInfo);
            }
          })
        })
      },
      //复制弹窗
      copeProductTip(row){
        this.copyId=row.id;
        this.centerDialogVisible1=true;
        this.getProduct1();
      },
      //复制任务
      copeProduct(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            var param = new FormData();  // 创建form对象
            param.append('id', this.copyId);
            param.append('productId', this.ruleForm2.productId);
            param.append('productName', this.ruleForm2.productName);
            param.append('productCode', this.ruleForm2.productCode);
            param.append('conditionId', this.ruleForm2.conditionId);
            param.append('conditionName', this.ruleForm2.conditionName);
            axios({
              method:"POST",
              url:"http://"+this.baseUrl+"/message/admin/copy/task",
              headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res)=>{
              if(res.data.msgCd=='ZYCASH-200'){
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.centerDialogVisible1=false;
                this.getProductList(1,30,null,null,null,null,null,null);
              } else {
                this.$message.error(res);
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      //封装产品名称
      selectChange1(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.productId === vId;
        });
        this.ruleForm2.productName=obj.productName;
        this.ruleForm2.productCode=obj.productCode;
      },
      //添加用户弹窗
      addUser(){
        this.centerDialogVisible2=true;
        //后台查询条件
        axios({
          method: "POST",
          url: "http://"+this.baseUrl+"/message/admin/message_condition/list",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.conditionList=res.data.body;
          } else {
            this.$message.error(res);
          }
        })
      },
      //选择用户条件
      changeHandler(key) {
        this.ruleForm2.conditionName=key;
        this.centerDialogVisible2 = false;
      },
      //编辑
      detailProduct(row){
        let id = row.id;
        let type = row.type;
        this.$router.push({path: `/taskDetail/${id}/${type}`});
      },
    },
    mounted:function () {
      this.getProduct();
      this.getModeList();
      this.getMessageClassifyList();
      this.getProductList(1,30,null,null,null,null,null,null);
    },
    data() {
      return {
        productList:[],
        productList1:[],
        messageClassifyList:[],
        modeList:[],
        tableData:[],
        productId:null,
        modeId:null,
        classifyId:null,
        value5:'',
        startTime:null,
        endTime:null,
        conditionName:null,
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
        nowPageSizes:20,
        multipleSelection:'',
        ids:'',
        taskList:[],
        conditionList:[],
        centerDialogVisible1:false,
        centerDialogVisible2:false,
        copyId:'',
        ruleForm2: {
          productId: '',
          productName: '',
          productCode: '',
          conditionId: '',
          conditionName: '',
        },
        rules2: {
          productId: [
            { required: true, message: '请选择产品', trigger: 'change' }
          ],
          conditionName: [
            {required: true, message: '请选择用户', trigger: 'change'}
          ],
        },
      }
    }
  }
</script>

<style scoped>
  .searchContent{
    width: 200px;
  }
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
    margin-right: 15px;
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
