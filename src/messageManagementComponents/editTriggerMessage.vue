<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/messageConfigurationList' }">任务列表</el-breadcrumb-item>
      <el-breadcrumb-item>编辑</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="160px" class="demo-ruleForm">
      <el-form-item style="margin-top: 20px;width: 480px" label="选择产品：" prop="productId">
        <el-select style="width: 320px" v-model="ruleForm.productId" value-key="id" placeholder="请选择" @change="selectChange1($event,productList)">
          <el-option
            v-for="item in productList"
            :key="item.productId"
            :label="item.productName"
            :value="item.productId">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="用户：" prop="conditionName" style="margin-top: 20px;width: 480px;position: relative;">
        <el-input v-model="ruleForm.conditionName" disabled></el-input>
        <el-button style="position: absolute;right:-115px;top:1px;" @click="addUser()" type="primary" plain>添加用户</el-button>
      </el-form-item>
      <el-form-item label="分类：" prop="classifyId" style="margin-top: 20px;width: 480px">
        <el-select style="width: 320px" v-model="ruleForm.classifyId" placeholder="请选择" @change="selectChange2($event,messageClassifyList)">
          <el-option
            v-for="item in messageClassifyList"
            :key="item.id"
            :label="item.classifyName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item style="text-align: left;margin-top: 20px;">
        <el-button type="primary" @click="toMessage()">短信</el-button>
        <el-button type="primary" @click="toPop()">弹窗</el-button>
        <el-button type="primary" @click="toPush()">推送</el-button>
        <el-button type="primary" @click="toTip()">提醒消息</el-button>
      </el-form-item>
      <el-form-item label="内容：" style="text-align: left;margin-top: 30px;;width: 1000px">
        <template>
          <el-table
            :data="taskList"
            border
            style="width: 100%"
            max-height="1000">
            <el-table-column
              prop="modeId"
              label="消息形式"
              :formatter="modeIdFormatter"
              width="120">
            </el-table-column>
            <el-table-column
              prop="messageName"
              label="名称"
              width="120">
            </el-table-column>
            <el-table-column
              prop="classifyName"
              label="分类"
              :formatter="classifyNameFormatter"
              width="120">
            </el-table-column>
            <el-table-column
              prop="content"
              label="内容"
              width="350">
            </el-table-column>
            <el-table-column
              label="操作"
              width="100">
              <template slot-scope="scope">
                <el-button @click.native.prevent="deleteRow(scope.$index, taskList)" type="text" size="small">移除</el-button>
                <el-button @click="detailProduct(scope.row)" type="text" size="small">详情</el-button>
              </template>
            </el-table-column>
          </el-table>
        </template>
      </el-form-item>
      <el-form-item label="备注：">
        <el-input type="textarea" :rows="4" placeholder="请输入备注" v-model="ruleForm.description"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">保存</el-button>
        <el-button type="" @click="resetForm('ruleForm')">取消</el-button>
      </el-form-item>
    </el-form>
    <!--触发条件表结构-->
    <el-dialog
      title="触发条件表"
      :visible.sync="centerDialogVisible"
      width="60%"
      center>
      <el-radio-group v-for="(item,index) in conditionList" :key="index" style="text-align: center" v-model="conditionId" size="medium">
        <el-radio style="margin-bottom: 20px;width: 160px;margin-right: 10px" :dataType="item.id"
                  border :label="item.id" @change="changeHandler(item.conditionName)">{{item.conditionName}}</el-radio>
      </el-radio-group>
    </el-dialog>
    <!--短信选择结构-->
    <el-dialog
      title="选择短信页面"
      :visible.sync="centerDialogVisible1"
      width="70%" height="80%"
      center>
      <template>
        <div>
          <div class="operationContent">
            <el-input style="width: 350px;" class="searchContent"
                      placeholder="输入名称或ID"
                      v-model="nameKey1"
                      clearable>
              <el-button id="searchBtn" @click="searchContent(nameKey1,2)" slot="append" icon="el-icon-search">查询</el-button>
            </el-input>
            <el-button type="info" id="cancelBtn" @click="centerDialogVisible1=false" slot="append">取消</el-button>
            <el-button type="primary" id="saveBtn" @click="saveContent1()" slot="append">保存</el-button>
          </div>
          <template>
            <el-table
              ref="multipleTable"
              :data="tableData"
              @selection-change="handleSelectionChange1"
              border
              style="width: 100%">
              <el-table-column
                type="selection"
                label="选择"
                align="center"
                width="40">
              </el-table-column>
              <el-table-column
                prop="productName"
                label="APP"
                align="center"
                width="150">
              </el-table-column>
              <el-table-column
                prop="name"
                label="短信名称"
                align="center"
                width="200">
              </el-table-column>
              <el-table-column
                prop="content"
                label="内容"
                align="center"
                width="600">
              </el-table-column>
            </el-table>
          </template>
          <div class="block">
            <el-pagination class="paginationBox"
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
      <div>
      </div>
    </el-dialog>
    <!--弹窗选择结构-->
    <el-dialog
      title="选择弹窗页面"
      :visible.sync="centerDialogVisible2"
      width="70%" height="80%"
      center>
      <template>
        <div>
          <div class="operationContent">
            <el-input style="width: 350px;" class="searchContent"
                      placeholder="输入名称或ID"
                      v-model="nameKey2"
                      clearable>
              <el-button @click="searchContent(nameKey2,3)" slot="append" icon="el-icon-search">查询</el-button>
            </el-input>
            <el-button type="info" @click="centerDialogVisible2=false" slot="append">取消</el-button>
            <el-button type="primary" @click="saveContent2()" slot="append">保存</el-button>
          </div>
          <template>
            <el-table
              ref="multipleTable"
              :data="tableData"
              @selection-change="handleSelectionChange2"
              border
              style="width: 100%">
              <el-table-column
                type="selection"
                label="选择"
                align="center"
                width="40">
              </el-table-column>
              <el-table-column
                prop="productName"
                label="APP"
                align="center"
                width="150">
              </el-table-column>
              <el-table-column
                prop="name"
                label="短信名称"
                align="center"
                width="200">
              </el-table-column>
              <el-table-column
                prop="content"
                label="内容"
                align="center"
                width="600">
              </el-table-column>
            </el-table>
          </template>
          <div class="block">
            <el-pagination class="paginationBox"
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
      <div>
      </div>
    </el-dialog>
    <!--推送选择结构-->
    <el-dialog
      title="选择推送页面"
      :visible.sync="centerDialogVisible3"
      width="70%" height="80%"
      center>
      <template>
        <div>
          <div class="operationContent">
            <el-input style="width: 350px;" class="searchContent"
                      placeholder="输入名称或ID"
                      v-model="nameKey3"
                      clearable>
              <el-button @click="searchContent(nameKey3,4)" slot="append" icon="el-icon-search">查询</el-button>
            </el-input>
            <el-button type="info" @click="centerDialogVisible3=false" slot="append">取消</el-button>
            <el-button type="primary" @click="saveContent3()" slot="append">保存</el-button>
          </div>
          <template>
            <el-table
              ref="multipleTable"
              :data="tableData"
              @selection-change="handleSelectionChange3"
              border
              style="width: 100%">
              <el-table-column
                type="selection"
                label="选择"
                align="center"
                width="40">
              </el-table-column>
              <el-table-column
                prop="productName"
                label="APP"
                align="center"
                width="150">
              </el-table-column>
              <el-table-column
                prop="name"
                label="短信名称"
                align="center"
                width="200">
              </el-table-column>
              <el-table-column
                prop="content"
                label="内容"
                align="center"
                width="600">
              </el-table-column>
            </el-table>
          </template>
          <div class="block">
            <el-pagination class="paginationBox"
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
      <div>
      </div>
    </el-dialog>
    <!--提醒选择结构-->
    <el-dialog
      title="选择提醒页面"
      :visible.sync="centerDialogVisible4"
      width="70%" height="80%"
      center>
      <template>
        <div>
          <div class="operationContent">
            <el-input style="width: 350px;" class="searchContent"
                      placeholder="输入名称或ID"
                      v-model="nameKey4"
                      clearable>
              <el-button @click="searchContent(nameKey4,1)" slot="append" icon="el-icon-search">查询</el-button>
            </el-input>
            <el-button type="info" @click="centerDialogVisible4=false" slot="append">取消</el-button>
            <el-button type="primary" @click="saveContent4()" slot="append">保存</el-button>
          </div>
          <template>
            <el-table
              ref="multipleTable"
              :data="tableData"
              @selection-change="handleSelectionChange4"
              border
              style="width: 100%">
              <el-table-column
                type="selection"
                label="选择"
                align="center"
                width="40">
              </el-table-column>
              <el-table-column
                prop="productName"
                label="APP"
                align="center"
                width="150">
              </el-table-column>
              <el-table-column
                prop="name"
                label="短信名称"
                align="center"
                width="200">
              </el-table-column>
              <el-table-column
                prop="content"
                label="内容"
                align="center"
                width="600">
              </el-table-column>
            </el-table>
          </template>
          <div class="block">
            <el-pagination class="paginationBox"
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
      <div>
      </div>
    </el-dialog>
    <!--短信详情结构-->
    <el-dialog
      title="短信详情"
      :visible.sync="centerDialogVisible5"
      width="50%"
      center>
      <template>
        <div class="bankBox">
          <p><span>名称：</span>{{msg.name}}</p>
          <p><span>分类：</span>{{msg.classifyName}}</p>
          <p><span>备注：</span>{{msg.description}}</p>
          <p><span>内容：</span>{{msg.content}}</p>
        </div>
        <el-button type="info" style="margin-left: 250px;margin-top: 10px" @click="centerDialogVisible5=false;">返回</el-button>
      </template>
      <div>
      </div>
    </el-dialog>
    <!--弹窗详情结构-->
    <el-dialog
      title="弹窗详情"
      :visible.sync="centerDialogVisible6"
      width="50%" height="100px"
      center>
      <template>
        <div class="bankBox" style="height: 280px">
          <p><span>名称：</span>{{msg.name}}</p>
          <p><span>分类：</span>{{msg.classifyName}}</p>
          <p><span>备注：</span>{{msg.description}}</p>
          <p><span>内容：</span>{{msg.content}}</p>
          <p><span>弹窗图：</span><img style="height: 80px;width: 170px"
                                   :src="'http://'+this.baseUrl + '/message/admin/message/down?file='+msg.picture+'&productCode='+msg.productCode"/></p>
          <p style="margin-top: 60px"><span>位置：</span>{{msg.positionName}}</p>
        </div>
        <el-button type="info" style="margin-left: 250px;margin-top: 10px" @click="centerDialogVisible6=false;">返回</el-button>
      </template>
      <div>
      </div>
    </el-dialog>
    <!--推送详情结构-->
    <el-dialog
      title="推送详情"
      :visible.sync="centerDialogVisible7"
      width="50%"
      center>
      <template>
        <div class="bankBox">
          <p><span>名称：</span>{{msg.name}}</p>
          <p><span>分类：</span>{{msg.classifyName}}</p>
          <p><span>标题：</span>{{msg.title}}</p>
          <p><span>备注：</span>{{msg.description}}</p>
          <p><span>内容：</span>{{msg.content}}</p>
          <p><span>LOGO：</span><img style="height: 80px;width: 170px"
                                    :src="'http://'+this.baseUrl + '/message/admin/message/down?file='+msg.picture+'&productCode='+msg.productCode"/></p>
        </div>
        <el-button type="info" style="margin-left: 250px;margin-top: 60px" @click="centerDialogVisible7=false;">返回</el-button>
      </template>
      <div>
      </div>
    </el-dialog>
    <!--提醒详情结构-->
    <el-dialog
      title="提醒详情"
      :visible.sync="centerDialogVisible8"
      width="50%"
      center>
      <template>
        <div class="bankBox">
          <p><span>名称：</span>{{msg.name}}</p>
          <p><span>分类：</span>{{msg.classifyName}}</p>
          <p><span>备注：</span>{{msg.description}}</p>
          <p><span>内容：</span>{{msg.content}}</p>
          <p><span>ICON图：</span><img style="height: 80px;width: 170px"
                                     :src="'http://'+this.baseUrl + '/message/admin/message/down?file='+msg.picture+'&productCode='+msg.productCode"/></p>
        </div>
        <el-button type="info" style="margin-left: 250px;margin-top: 60px" @click="centerDialogVisible8=false;">返回</el-button>
      </template>
      <div>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        modeCode:'short_message',
        productList:[],
        messageClassifyList:[],
        conditionList:[],
        conditionId:'',
        nameKey1:'',
        nameKey2:'',
        nameKey3:'',
        nameKey4:'',
        multipleSelection1:[],
        multipleSelection2:[],
        multipleSelection3:[],
        multipleSelection4:[],
        ruleForm: {
          id:'',
          productId:'',
          productName:'',
          productCode:'',
          conditionId:'',
          conditionName:'',
          classifyId:'',
          classifyName:'',
          messageName:'',
          description:'',
        },
        rules: {
          productId: [
            {required: true, message: '请选择产品', trigger: 'change'}
          ],
          conditionName: [
            {required: true, message: '请选择用户', trigger: 'change'}
          ],
          classifyId: [
            {required: true, message: '请选择分类', trigger: 'change'}
          ],
        },
        tableData:[],
        taskList:[],
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[20],
        nowPageSizes:20,
        centerDialogVisible:false,
        centerDialogVisible1:false,
        centerDialogVisible2:false,
        centerDialogVisible3:false,
        centerDialogVisible4:false,
        centerDialogVisible5:false,
        centerDialogVisible6:false,
        centerDialogVisible7:false,
        centerDialogVisible8:false,
        msg:'',
        id:'',
      };
    },
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
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //添加用户弹窗
      addUser(){
        this.centerDialogVisible=true;
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
        this.ruleForm.conditionId=this.conditionId;
        this.ruleForm.conditionName=key;
        this.centerDialogVisible = false;
      },
      //封装产品名称
      selectChange1(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.productId === vId;
        });
        this.ruleForm.productName=obj.productName;
        this.ruleForm.productCode=obj.productCode;
      },
      //封装分类名称
      selectChange2(vId,list){
        let obj = {};
        obj = list.find((item)=>{
          return item.id === vId;
        });
        this.ruleForm.classifyName=obj.classifyName;
      },
      /**
       * 短信列表接口
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 消息类型码 1提醒，2短信，3弹窗，4推送
       * @param data4 消息名称
       * @param data5 消息分类id
       */
      getShortMessageList(data1,data2,data3,data4,data5){
        axios({
          method: "POST",
          url: "http://"+this.baseUrl+"/message/admin/getMessageByType",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            pageNum: data1,
            pageSize: data2,
            modeCode: data3,
            name: data4,
            classifyId: data5,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.tableData=res.data.body.list;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          } else {
            this.$message.error(res);
          }
        })
      },
      //全选
      handleSelectionChange1(val) {
        this.multipleSelection1 = val;
      },
      //全选
      handleSelectionChange2(val) {
        this.multipleSelection2 = val;
      },
      //全选
      handleSelectionChange3(val) {
        this.multipleSelection3 = val;
      },
      //全选
      handleSelectionChange4(val) {
        this.multipleSelection4 = val;
      },
      //短信选择弹窗
      toMessage(){
        if (this.ruleForm.classifyId == '') {
          this.$message({
            message: '请先选择分类',
            type: 'warning'
          });
        } else {
          this.centerDialogVisible1=true;
          this.getShortMessageList(1,20,'short_message',null,this.ruleForm.classifyId);
        }
      },
      //选择短信后保存
      saveContent1(){
        if (this.multipleSelection1 == 0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条内容',
            type: 'warning'
          });
        } else {
          var _this=this;
          this.multipleSelection1.forEach(function (item,index){
            _this.taskList.push({
              messageId:item.id,modeId:2,messageName:item.name,classifyName:item.classifyName,content:item.content,taskId:_this.id
            });
          });
          this.centerDialogVisible1=false;
        }
      },
      //弹窗选择弹窗
      toPop(){
        if (this.ruleForm.classifyId == '') {
          this.$message({
            message: '请先选择分类',
            type: 'warning'
          });
        } else {
          this.centerDialogVisible2=true;
          this.getShortMessageList(1,20,'popup_message',null,this.ruleForm.classifyId);
        }
      },
      //选择弹窗后保存
      saveContent2(){
        if (this.multipleSelection2 == 0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条内容',
            type: 'warning'
          });
        } else {
          var _this=this;
          this.multipleSelection2.forEach(function (item,index){
            _this.taskList.push({
              messageId:item.id,modeId:3,messageName:item.name,classifyName:item.classifyName,content:item.content,taskId:_this.id
            });
          });
          this.centerDialogVisible2=false;
        }
      },
      //推送选择弹窗
      toPush(){
        if (this.ruleForm.classifyId == '') {
          this.$message({
            message: '请先选择分类',
            type: 'warning'
          });
        } else {
          this.centerDialogVisible3=true;
          this.getShortMessageList(1,20,'push_message',null,this.ruleForm.classifyId);
        }
      },
      //选择推送后保存
      saveContent3(){
        if (this.multipleSelection3 == 0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条内容',
            type: 'warning'
          });
        } else {
          var _this=this;
          this.multipleSelection3.forEach(function (item,index){
            _this.taskList.push({
              messageId:item.id,modeId:4,messageName:item.name,classifyName:item.classifyName,content:item.content,taskId:_this.id
            });
          });
          this.centerDialogVisible3=false;
        }
      },
      //提醒选择弹窗
      toTip(){
        if (this.ruleForm.classifyId == '') {
          this.$message({
            message: '请先选择分类',
            type: 'warning'
          });
        } else {
          this.centerDialogVisible4=true;
          this.getShortMessageList(1,20,'remind_message',null,this.ruleForm.classifyId);
        }
      },
      //选择提醒后保存
      saveContent4(){
        if (this.multipleSelection4 == 0) {
          this.$message({
            showClose: true,
            message: '请至少选择一条内容',
            type: 'warning'
          });
        } else {
          var _this=this;
          this.multipleSelection4.forEach(function (item,index){
            _this.taskList.push({
              messageId:item.id,modeId:1,messageName:item.name,classifyName:item.classifyName,content:item.content,taskId:_this.id
            });
          });
          this.centerDialogVisible4=false;
        }
      },
      //条件查询列表
      searchContent(nameKey,type){
        this.getShortMessageList(1,20,type,nameKey,this.ruleForm.classifyId);
      },
      //翻页
      handleCurrentChange(val) {
        this.getShortMessageList(val,20,2,this.nameKey,this.ruleForm.classifyId);
      },
      //过滤消息形式字段
      modeIdFormatter(row){
        let modeId = row.modeId;
        if(modeId == 1){
          return '提醒'
        } else if (modeId == 2){
          return '短信'
        } else if (modeId == 3){
          return '弹窗'
        } else if (modeId == 4){
          return '推送'
        }
      },
      //过滤分类字段
      classifyNameFormatter(row){
        return this.ruleForm.classifyName;
      },
      //移除
      deleteRow(index, rows) {
        rows.splice(index, 1);
      },
      //保存任务
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            if (this.taskList.length > 0) {
              this.ruleForm.messageName=this.taskList[0].messageName;
            } else {
              this.ruleForm.messageName=null;
            }
            var param = new FormData();
            param.append('id', this.ruleForm.id);
            param.append('productId', this.ruleForm.productId);
            param.append('productName', this.ruleForm.productName);
            param.append('productCode', this.ruleForm.productCode);
            param.append('conditionId', this.ruleForm.conditionId);
            param.append('conditionName', this.ruleForm.conditionName);
            param.append('classifyId', this.ruleForm.classifyId);
            param.append('classifyName', this.ruleForm.classifyName);
            param.append('taskList', JSON.stringify(this.taskList));
            param.append('messageName', this.ruleForm.messageName);
            param.append('description', this.ruleForm.description);
            param.append('type', 1);
            axios({
              method: "POST",
              url: "http://"+this.baseUrl+"/message/admin/save/task",
              headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'Authorization': localStorage.token
              },
              data:param,
            }).then((res) => {
              if (res.data.msgCd == 'ZYCASH-200') {
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
                this.$router.push('/messageConfigurationList');
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
      //取消按钮
      resetForm(formName) {
        this.$router.go(-1);
      },
      //详情
      detailProduct(row){
        if (row.modeId == 2) {
          this.centerDialogVisible5=true;
        } else if (row.modeId == 3) {
          this.centerDialogVisible6=true;
        } else if (row.modeId == 4) {
          this.centerDialogVisible7=true;
        } else if (row.modeId == 1) {
          this.centerDialogVisible8=true;
        }
        axios({
          method: "POST",
          url: "http://"+this.baseUrl+"/message/admin/message/get",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            id: row.messageId,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.msg=res.data.body;
          } else {
            this.$message.error(res);
          }
        })
      },
      //查询任务详情
      getTaskById(data){
        axios({
          method: "POST",
          url: "http://"+this.baseUrl+"/message/admin/edit/task",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params:{
            taskId: data,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.ruleForm=res.data.body;
            this.taskList=res.data.body.list;
          } else {
            this.$message.error(res);
          }
        })
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      console.log(this.id);
      this.getTaskById(this.id);
      this.getProduct();
      this.getMessageClassifyList();
    },
  }
</script>

<style scoped>
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }

  .demo-ruleForm {
    width: 800px;
    text-align: center;
    margin-top: 50px;

  }

  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .searchContent{
    width: 200px;
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
  .bankBox p{
    height: 35px;
    line-height: 25px;
  }
  .bankBox p span{
    display: inline-block;
    width: 100px;
    text-align: center;
  }
</style>
