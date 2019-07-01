<template>
    <div class="leftSilder">
      <div style="width: 175px;height: 70px"></div>
      <el-row class="tac">
        <el-col :span="24">
          <el-menu
            :default-active="$route.path"
            class="el-menu-vertical-demo"
            @open="handleOpen"
            @close="handleClose"
            background-color="#545c64"
            text-color="#fff"
            active-text-color="#ffd04b" router>
            <el-submenu v-if="this.hasPermissionCustom('operate:borrowing:all')" index="1" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>金融产品配置</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('operate:productlist')" index="/financeProductList">金融产品列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('operate:merchant:all')" index="2" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>商户管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('operate:merchantlist')" index="/merchantProductList">商户列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('operate:product:all')" index="3" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>产品管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('operate:productManage:list')" index="/productProductList">产品列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('user:admin')" index="4" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>用户管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('user:customer:list')" index="/userProductList">用户列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('user:black:list')" index="/blackList">黑名单列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('credit:bank:getBankCardInfoByParams')" index="/bankCardManagement">银行卡管理列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('risk:admin')" index="5" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>风控管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('risk:field:list')" index="/fieldList">字段管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('risk:tag:list')" index="/tagList">用户标签管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('risk:rule:list')" index="/ruleList">规则管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('risk:rule_collection:list')" index="/collectionList">规则集管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('risk:collection_flow:list')" index="/flowList">子流程管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('risk:parent_flow:list')" index="/flowHeadList">总流程管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('risk:flow_log:list')" index="/flowLogList">风险命中列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('order:daihou')" index="6" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>贷后管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:overdue:orderList')" index="/afterLoanManagement">逾期订单管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:overdue:noRepayOrderList')" index="/afterLoanNoRepay">逾期未还订单</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:overdue:repayedOrderList')" index="/afterLoanRepayed">逾期已还订单</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:outsourcing:orderList')" index="/outsourcingOrderList">委外订单</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:outsourcing:repayedOrderList')" index="/outsourcingOrderRepayedList">委外已还</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:label:list')" index="/collectionTag">催收标签</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:callType:list')" index="/callTypeTag">通话类型</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('operate:collection:list')" index="/collectionPersonManagement">催收人员管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('operate:group:list')" index="/collectionGroupManagement">催收群组管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:distribution:find')" index="/collectionOrder">催收分单</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:borrowing:findAssignedOrder')" index="/collectionOrderList">催收已分订单</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:borrowing:findCollectionOrder')" index="/collectionTaskList">催收任务列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:borrowing:findCollectionOrder')" index="/collectionTaskFinishList">催回列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('operate:productVie')" index="7" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>权限管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('operate:role:list')" index="/roleList">角色列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('operate:account:list')" index="/accountManagement">账户管理列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('channel:admin')" index="8" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>渠道管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('channel:channel:list')" index="/channelconnectionManagement">渠道链接列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('channel:channel_statistics:list')" index="/channelCount">渠道统计列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('channel:account:list')" index="/accountCenter">账户中心</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('order:admin')" index="9" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>订单管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item index="/approvalCenter">审批中心</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:auditJS:list')" index="/pendingApprovalJS">待机审列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:audit:list')" index="/pendingApproval">待审批列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:pending:list')" index="/pendingLoan">待放款列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:payment:list')" index="/loanMade">已放款列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:borrowing:list')" index="/orderList">订单管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:repayment:list')" index="/paymentHistory">还款记录列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:paymentLog:list')" index="/loanRecord">放款记录列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('quota:admin')" index="10" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>额度管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('quota:amountField:list')" index="/amountFieldList">字段管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('quota:amountTag:list')" index="/amountTagList">标签管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('quota:amountFlow:list')" index="/amountFlowList">流程管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('quota:flowLog:list')" index="/amountFlowLogList">命中列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('order:flowField')" index="11" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>展期管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:flowField:getFieldList')" index="/orderFieldList">字段管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:flowField:list')" index="/orderFlowList">流程管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('order:ordertTag:hitList')" index="/orderFlowLogList">命中列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('message:admin')" index="12" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>消息管理</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('message:message_classify:find')" index="/messageClassify">分类列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('message:message:smsMessage')" index="/smsMessage">短信消息</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('message:message:reminderMessage')" index="/reminderMessage">提醒消息</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('message:message:popUpMessage')" index="/popUpMessage">弹窗消息</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('message:message:pushMessage')" index="/pushMessage">推送消息</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu v-if="this.hasPermissionCustom('message:message')" index="13" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>消息配置</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('message:task:list')" index="/messageConfigurationList">任务列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item v-if="this.hasPermissionCustom('message:messagelog:list')" index="/messageRecord">日志列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu  index="14" style="padding-left:0 ;">
              <template style="padding-left: 5px" slot="title">
                <i class="el-icon-menu"></i>
                <span>导出报表</span>
              </template>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item  index="/exportExcel">报表列表</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group style="padding-left: 5px">
                <el-menu-item  index="/addExcelSetting">新建导出</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
          </el-menu>
        </el-col>
      </el-row>
    </div>
</template>

<script>
  export default {
    methods: {
      handleOpen(key, keyPath) {
        console.log(key, keyPath);
      },
      handleClose(key, keyPath) {
        console.log(key, keyPath);
      },
    }
  }
</script>

<style>
  .leftSilder {
    position: fixed;
    width: 165px;
    height: 100%;
    min-height: 100%;
    overflow-y: auto;
    top: 0;
    z-index: 100;
    box-shadow: 1px 0 5px rgba(0,0,0,.05);
    background-color: rgb(84, 92, 100);
    font-size: 18px;
  }
  .el-menu {
    border-right: 0;
    list-style: none;
    position: relative;
    margin: 0;
    padding-left: 0;

  }
  .leftSilder .el-submenu__title , .leftSilder .el-menu-item{
    color: rgb(255, 255, 255);
    background-color: rgb(84, 92, 100);
    font-size: 16px;

  }
  .leftSilder .el-menu-item-group__title{
    display: none;
  }
  .el-submenu .el-menu-item{
     min-width: 160px;
  }
</style>
