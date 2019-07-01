<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>任务列表</el-breadcrumb-item>
      <el-breadcrumb-item v-if="type == 0">通知类</el-breadcrumb-item>
      <el-breadcrumb-item v-if="type == 1">触发类</el-breadcrumb-item>
      <el-breadcrumb-item v-if="type == 2">营销类</el-breadcrumb-item>
      <el-breadcrumb-item v-if="type == 3">技术人员</el-breadcrumb-item>
      <el-breadcrumb-item>详情</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="bankBox" v-if="type == 0">
      <!--通知类-->
      <p><span>产品：</span>{{ruleForm.productName}}</p>
      <p><span>用户：</span>{{ruleForm.conditionName}}</p>
      <p><span>分类：</span>{{ruleForm.classifyName}}</p>
      <p><span>设定日期：</span>{{ruleForm.setTime | setTimeFalse}}</p>
      <p v-if="ruleForm.setTime == 1"><span>时间段：</span>{{ruleForm.timeQuantum}}</p>
      <p v-if="ruleForm.setTime == 2"><span>日期：</span>{{ruleForm.date}}</p>
      <p v-if="ruleForm.setTime == 3"><span>相隔天数：</span>{{ruleForm.days}}</p>
      <p><span>具体时间：</span>{{ruleForm.specificTime}}</p>
      <p><span>备注：</span>{{ruleForm.description}}</p>
      <p><span>内容：</span>
        <template>
          <el-table
            :data="taskList"
            border
            style="width: 70%;margin-left: 138px;margin-top: -30px"
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
              width="150">
            </el-table-column>
            <el-table-column
              prop="classifyName"
              label="消息分类"
              :formatter="classifyNameFormatter"
              width="150">
            </el-table-column>
            <el-table-column
              prop="content"
              label="内容"
              width="500">
            </el-table-column>
          </el-table>
        </template>
      </p>
    </div>
    <div class="bankBox" v-if="type == 1">
      <!--触发类-->
      <p><span>产品：</span>{{ruleForm.productName}}</p>
      <p><span>用户：</span>{{ruleForm.conditionName}}</p>
      <p><span>分类：</span>{{ruleForm.classifyName}}</p>
      <p><span>备注：</span>{{ruleForm.description}}</p>
      <p><span>内容：</span>
        <template>
          <el-table
            :data="taskList"
            border
            style="width: 70%;margin-left: 138px;margin-top: -30px"
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
              width="150">
            </el-table-column>
            <el-table-column
              prop="classifyName"
              label="消息分类"
              :formatter="classifyNameFormatter"
              width="150">
            </el-table-column>
            <el-table-column
              prop="content"
              label="内容"
              width="500">
            </el-table-column>
          </el-table>
        </template>
      </p>
    </div>
    <div class="bankBox" v-if="type == 2">
      <!--营销类-->
      <p><span>产品：</span>{{ruleForm.productName}}</p>
      <p><span>用户：</span>{{ruleForm.conditionName}}</p>
      <p><span>分类：</span>{{ruleForm.classifyName}}</p>
      <p><span>设定日期：</span>{{ruleForm.setTime | setTimeFalse}}</p>
      <p v-if="ruleForm.setTime == 1"><span>时间段：</span>{{ruleForm.timeQuantum}}</p>
      <p v-if="ruleForm.setTime == 2"><span>日期：</span>{{ruleForm.date}}</p>
      <p v-if="ruleForm.setTime == 3"><span>相隔天数：</span>{{ruleForm.days}}</p>
      <p><span>具体时间：</span>{{ruleForm.specificTime}}</p>
      <p><span>备注：</span>{{ruleForm.description}}</p>
      <p><span>内容：</span>
        <template>
          <el-table
            :data="taskList"
            border
            style="width: 70%;margin-left: 138px;margin-top: -30px"
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
              width="150">
            </el-table-column>
            <el-table-column
              prop="classifyName"
              label="消息分类"
              :formatter="classifyNameFormatter"
              width="150">
            </el-table-column>
            <el-table-column
              prop="content"
              label="内容"
              width="500">
            </el-table-column>
          </el-table>
        </template>
      </p>
    </div>
    <div class="bankBox" v-if="type == 3">
      <!--技术人员-->
      <p><span>产品：</span>{{ruleForm.productName}}</p>
      <p><span>名称：</span>{{ruleForm.name}}</p>
      <p><span>分类：</span>{{ruleForm.classifyName}}</p>
      <p><span>备注：</span>{{ruleForm.description}}</p>
      <p><span>内容：</span>{{ruleForm.content}}</p>
      <p><span>短信ID：</span>{{ruleForm.noteId}}</p>
    </div>
    <el-button-group class="btGrop">
      <el-button @click="resetForm()" class="guan">返回</el-button>
    </el-button-group>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        taskList:[],
        ruleForm: {},
        type:''
      };
    },
    methods: {
      //查询消息详情
      getTaskById(id) {
        axios({
          method:"POST",
          url:"http://"+this.baseUrl+"/message/admin/edit/task",
          headers:{
            'Content-Type':'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            taskId: id,
          }
        }).then((res)=>{
          if(res.data.msgCd=='ZYCASH-200'){
            this.ruleForm=res.data.body;
            this.taskList=res.data.body.list;
          }else if(res.data.msgCd=='ZYCASH-1009'){
            this.$message.error(res.data.msgInfo);
          }
          else {
            this.$message.error(res);
          }
        })
      },
      //取消按钮
      resetForm() {
        this.$router.go(-1);
      },
      //过滤消息形式字段
      modeIdFormatter(row){
        let modeId = row.modeId;
        if(modeId == 1){
          return '提醒消息'
        } else if (modeId == 2){
          return '短信消息'
        } else if (modeId == 3){
          return '弹窗消息'
        } else if (modeId == 4){
          return '推送消息'
        }
      },
      //过滤分类字段
      classifyNameFormatter(row){
        return this.ruleForm.classifyName;
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.type=this.$route.params.type;
      this.getTaskById(this.id);
    },
    filters:{
      setTimeFalse:function(arg1){
        if(arg1==0){
          var result = "每一天";
          return result;
        }else if(arg1==1){
          var result = "时间段";
          return result;
        }
        else if(arg1==2){
          var result = "某一天";
          return result;
        }
        else if(arg1==3){
          var result = "每隔多少天";
          return result;
        }else if(arg1==4){
          var result = "立即执行";
          return result;
        }
      },
    }
  }
</script>

<style scoped>
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }
  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .bankBox p{
    height: 35px;
    line-height: 35px;
  }
  .bankBox p span{
    display: inline-block;
    width: 140px;
  }
  .btGrop{
    display: block;
    width: 500px;
    margin: 0 auto;
    margin-top: 300px;
  }
  .content .guan{
    display: inline-block;
    font-size: 20px;
    padding: 15px 40px;
    margin-left: 40px;
  }
</style>
